## 本体跨链合约开发

[English](How_to_new_cross_chain_asset.md) | 中文

本体多链生态支持跨链合约，所谓跨链合约就是合约的逻辑贯穿两条甚至多条链，例如：

发布一本OEP4，该OEP4合约资产可以在ontology和ethereum之间转移。

### 跨链合约概述

跨链合约实际上是多本合约，例如dApp开发者需要在链A和链B上实现跨链业务，此时开发者需要分别在链A和链B上部署智能合约（暂时称之为智能合约A和智能合约B）。

本体跨链合约的开发总的来说可以分为两部分，业务部分和跨链部分：

业务部分是指在某条链中运行的逻辑代码，按照标准的智能合约开发方式开发，完成合约在该链中的业务。

当需要跨链时，如链A的逻辑执行完，接下来需要执行链B的逻辑，那么就需要用到跨链接口。

### 跨链接口

合约的跨链对开发者来说只需要关注一个跨链接口，也就是跨链管理合约的“createCrossChainTx”接口，该接口将链A上已经执行的业务存入merkle tree，会有矿工生成该跨链交易的merkle证明，并将其提交到链B的跨链管理合约中，跨链管理合约会验证merkle证明，并按照参数调用智能合约B中对应的方法。

## 跨链合约开发示例（新发行的OEP4资产）

某开发者想在链A和链B上发行资产，但是希望链A和链B上的资产互通，也就是说其需要发行一种资产，这种资产能够在链A和链B上同时使用，并且可以在链A和链B上自由转移。

为了使资产可以在链A和链B之间互相转移，在OEP4标准接口的基础上，需要增加lock和unlock接口，用户在链A中调用lock接口将资产锁定在智能合约A，该接口调用跨链管理合约实现跨链调用智能合约B中的unlock接口，unlock接口在链B中将智能合约B中的资产释放给该用户。反之亦然。

lock接口实现：

```python
def lock(to_chainId, from_address, to_address, amount):
    """
    lock some amount of tokens of this contract, call cross chain method to release to_amount of tokens of another chain's contract
    :param to_chain_id: chain id of destination chain
    :param address: address of caller
    :param to_amount: amount to lock
    :return:
    """
    assert (amount >= 0)
    assert (CheckWitness(from_address))
    assert (not isPaused())
    # eth address format:0x673dfa9caf9145fdbef98e9d9874f36e63d8a5b4,length is 42
    assert (len(to_address) != 0)

    Put(ctx, concat(BALANCE_KEY, from_address), Sub(balanceOf(from_address), amount))
    Put(ctx, TOTAL_SUPPLY_KEY, Sub(totalSupply(), amount))

    btcRedeemScriptBytes = getBtcRedeemScript()
    argsList = [to_address, amount, btcRedeemScriptBytes]

    input_bytes = _serialzieArgs(argsList)
    to_contract = getContractAddrWithChainId(to_chainId)
    param = state(to_chainId, to_contract, "unlock", input_bytes)
    assert (Invoke(0, CROSS_CHAIN_CONTRACT_ADDRESS, "createCrossChainTx", param))

    LockEvent(to_chainId, from_address, to_address, amount)
    return True
```

参数为：

```
to_chain_id: 目标链的链ID
destination_contract：目标合约的合约地址的反序
address: 调用发起者地址，矿工费从该地址扣除
amount：转移资产的数量
```

可以看到该接口先执行转账操作来冻结用户的链A资产，然后调用了跨链管理合约中的createCrossChainTx方法，该方法接受四个参数：

```
to_chain_id: 目标链的链ID
destination_contract：目标合约的合约地址的反序
function: 要跨链调用的目标合约方法名
input_bytes：序列化之后的输入参数
```

该方法会跨链调用目标合约地址的方法名，输入参数为input_bytes。

unlock接口实现：

```python
def unlock(params, fromContractAddr, fromChainId):
    """
    lock some amount of tokens of this contract, call cross chain method to release to_amount of tokens of another chain's contract
    :param fee: miner fee of this cross chain tx
    :param to_chain_id: chain id of destination chain
    :param address: address of caller
    :param to_amount: amount to lock
    :return:
    """
    res = _deserialzieArgs(params)
    toAddress = res[0]
    value = res[1]
    assert(fromContractAddr == getContractAddrWithChainId(fromChainId))
    assert (value >= 0)
    assert (isAddress(toAddress))
    assert (CheckWitness(CROSS_CHAIN_CONTRACT_ADDRESS))

    Put(ctx, concat(BALANCE_KEY, toAddress), Add(balanceOf(toAddress), value))
    Put(ctx, TOTAL_SUPPLY_KEY, Add(totalSupply(), value))
    UnlockEvent(toAddress, value)
    return True
```

该接口校验原链上的合约地址，接受input_bytes作为输入参数，反序列化出原始的input_map，然后执行下面的逻辑，解锁链B资产给用户。

需要注意的是该方法作为跨链的目标方法，只能接受跨链管理合约的调用。

## 跨链合约开发示例（已存在的OEP4资产）

对于已经存在的OEP4资产来说，其合约代码是不能修改并添加lock、unlock方法的。以需要从ontology跨链到以太坊为例，需要额外实现一个代理合约，相当于原先OEP4合约的补充功能，实现了跨链协议中的主要接口，也可以使用现有的代理合约，实现跨链，避免重复开发。

如下为代理合约必须实现的接口：

```python
'''
This function binds the target chain proxy contract. Only the binded proxy contract request is accept by this contract.
@param toChainId         		The target chain id

@param targetProxyHash   		The binded proxy contract address
'''
def bindProxyHash(toChainId, targetProxyHash):
    assert (CheckWitness(Operator))
    Put(GetContext(), concat(PROXY_HASH, toChainId), targetProxyHash)
    Notify(["bindProxyHash", toChainId, targetProxyHash])
    return True

def bindAssetHash(fromAssetHash, toChainId, toAssetHash, assetLimit, isTargetChainAsset):
    assert (CheckWitness(Operator))
    Put(GetContext(), concat(ASSET_HASH, concat(fromAssetHash, toChainId)), toAssetHash)
    if isTargetChainAsset:
        currentLimit = getCrossedLimit(fromAssetHash, toChainId)
        assert (assetLimit >= currentLimit)
        increment = assetLimit - currentLimit
        Put(GetContext(), concat(TCC_ASSET_SUPPLY, concat(fromAssetHash, toChainId)),
            getCrossedAmount(fromAssetHash, toChainId) + increment)
    Put(GetContext(), concat(TCC_ASSET_LIMIT, concat(fromAssetHash, toChainId)), assetLimit)
    Notify(["bindAssetHash", fromAssetHash, toChainId, toAssetHash, assetLimit])
    return True

'''
This function is meant to be invoked by the user, a certin amount teokens will be locked in the proxy contract immediately. Then the same amount of tokens will be unloked from target chain proxy contract at the target chain with chainId later.
@param fromAssetHash   				The asset hash in current chain
@param fromAddress 					The invoker address
@param toChainId         			The target chain id
                         
@param toAddress         			The address in bytes format to receive same amount of tokens in 									target chain 
@param amount            			The amount of tokens to be crossed from ethereum to the chain 										with chainId
'''
def lock(fromAssetHash, fromAddress, toChainId, toAddress, amount)

'''
This function is meant to be invoked by the crosschain management contract, then mint a certin amount of tokens to the designated address since a certain amount was burnt from the source chain invoker.
@param params            The argument bytes recevied by the lock proxy contract, need to be 								deserialized.
						based on the way of serialization in the source chain proxy contract.
@param fromContractAddr  The source chain contract address
@param fromChainId       The source chain id
'''
def unlock(params, fromContractAddr, fromChainId)
```

- `bindProxyHash`绑定目标链上的代理合约hash，或者实现了跨链接口的合约hash，在OEP4朝其他链跨链的时候会将该目标链代理合约作为中转站，进而将OEP4转到目标链的映射合约，当然在跨链开始前应该预先部署好映射合约；
- `bindAssetHash`绑定OEP4代币合约地址和目标链映射合约，比如USDT地址，并设置金额限制；
- `lock`接口会把OEP4代币锁到代理合约中，然后向管理合约发出跨链请求；
- `unlock`接口会释放原先锁定的OEP4代币，即从其他链返回的OEP4代币，管理合约会调用该接口，为用户释放代币；

所以代理合约主要实现的是OEP4的注册、锁定与解锁，锁定就是用户将OEP4转到合约中，解锁只有跨链管理合约可以调用。代理合约可以自己部署，也可以使用现有的合约，多个OEP4可以使用一本代理合约。

### 实例：已有的OEP4跨链

#### Step1 部署跨链代理合约以及配置

以将USDT从ontology跨链到以太坊为例：

<div align=center><img width="360" height="200" src="resources/usdt_ex.png"/></div>

- 在ontology上部署一个OEP4的USDT映射合约。
- 需要在以太上和ontology上部署跨链代理合约，在两边都调用`bindProxyHash`绑定代理之间的对应关系。
- 在两边的代理合约绑定资产，即在代理合约上绑定ERC20和OEP4跨链关系，调用`bindAssetHash`在代理合约中绑定USDT到ontology的映射合约，以及金额限制。
- 如果使用其他代理服务提供商的代理合约，需要和向其提出请求，由其完成绑定。

#### Step2 跨链转账

用户调用`lock`接口将USDT锁定到代理合约，然后本体的跨链生态就会将USDT搬运到以太坊，用户可以在以太坊的USDT上看到余额了；同样地，在以太坊端调用代理合约的`lock`接口，把USDT转回以本体。

## 文档示例

实现跨链接口的OEP4合约代码示例[To be added]()

OEP4代理合约代码示例[To be added]()