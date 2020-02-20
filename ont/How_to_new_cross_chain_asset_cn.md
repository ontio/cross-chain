## 本体跨链合约开发

本体多链生态支持跨链合约，所谓跨链合约就是合约的逻辑贯穿两条甚至多条链，例如：

发布一本OEP4，该OEP4合约资产可以在ontology和ethereum之间转移。

一本dApp合约，该合约部分逻辑在ontology完成，部分逻辑在ethereum完成。

### 跨链合约概述

本体跨链合约实际上是多本合约，例如dApp开发者需要在链A和链B上实现跨链业务，此时开发者需要分别在链A和链B上部署智能合约（暂时称之为智能合约A和智能合约B）。

本体跨链合约的开发总的来说可以分为两部分，业务部分和跨链部分：

业务部分是指在某条链中运行的逻辑代码，按照标准的智能合约开发方式开发，完成合约在该链中的业务。

当需要跨链时，如链A的逻辑执行完，接下来需要执行链B的逻辑，那么就需要用到跨链接口。

### 跨链接口

合约的跨链对开发者来说只需要关注一个跨链接口，也就是跨链管理合约的“createCrossChainTx”接口，该接口将链A上已经执行的业务存入merkle tree，会有矿工生成该跨链交易的merkle证明，并将其提交到链B的跨链管理合约中，跨链管理合约会验证merkle证明，并按照参数调用智能合约B中对应的方法。

### 跨链合约开发示例

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

可以看到该接口先执行转账操作来冻结用户的链A资产，然后调用了跨链管理合约中的createCrossChainTx方法，该方法接受六个参数：

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