<h1 align="center">Ontology Cross Chain - ethereum relayer manual</h1>
<h4 align="center">Version 1.0 </h4>

[English](how_to_deploy_crosschain_on_ethereum.md) | 中文

本文档介绍如何部署从以太到其他链的跨链。包括如何部署跨链代理合约，已有ERC20的跨链，部署新ERC20跨链的两种方式：通过跨链代理跨链和直接跨链。

## 部署跨链代理合约以及配置

以需要从以太到ontology跨链为例，需要在以太上和ontology上部署遵循跨链协议的跨链代理合约，如果跨链代理合约已经部署，只需要进行以下的配置。

在以太的跨链代理合约上配置绑定到ontology的跨链代理合约是通过调用以太跨链代理合约的bindProxyHash接口来完成：

```
/* @notice                  This function binds the target chain proxy contract.
*                           Only the binded proxy contract request is accept by this contract.
*  @param toChainId         The target chain id
*
*  @param targetProxyHash   The binded proxy contract address
*
*/
function bindProxyHash(uint64 toChainId, bytes memory targetProxyHash)
```

在ontology的跨链代理合约上配置绑定到以太的跨链代理合约，参考ontology的手册中关于部署跨链代理合约和配置部分。


## 部署已有ERC20的跨链

如果已经发行了ERC20，可以基于以下手册来部署该ERC20到其他链的跨链，以下以跨链到ontology示例。

### 在目标链部署资产合约并配置

以跨链到ontology为例，在ontology部署该ERC20对应的OEP4合约。

部署OEP4合约后，需要在ontology的跨链代理合约上配置该OEP4和ERC20的跨链关系，参考ontology的手册中关于配置OEP4的跨链部分。


### 配置ERC20

在以太的跨链代理合约上绑定ERC20和OEP4跨链关系。
调用以太跨链代理合约的bindAssetHash接口：
```
/* @notice                  This function binds the erc20 asset and eht other asset on other chain.
*                           Only the binded asset can cross chain.
*  @param sourceAssetHash         The erc20 asset hash
*
*  @param toChainId               The target chain id
*
*  @param targetAssetHash         The target asset hash on target chain
*
*  @param assetLimit              The limit of target asset used to cross chain
*
*  @param isTargetChainAsset      The indicate of sourceAssetHash, new erc20 or existing erc20
*
*/
function bindAssetHash(address sourceAssetHash, uint64 toChainId, bytes memory targetAssetHash, uint256 assetLimit, bool isTargetChainAsset)
```

## 通过代理合约跨链的新ERC20部署

如果新ERC20通过代理合约跨链的话，只需要部署遵循ERC20标准的合约，再按照已有ERC20的跨链进行部署和配置。

## 直接跨链的ERC20部署

### 开发合约

如果不通过代理合约跨链，那么需要遵循跨链协议和ERC20协议来开发ERC20合约。可以参考其模板。
主要有4个主要的接口需要实现，配置erc20的minter接口setminter，绑定目标链的资产合约接口bindContractAddrWithChainId，发送到其他链时调用的lock接口，以及从其他链发送回以太时由管理合约调用的unlock

setminter接口只能被该erc20的管理员调用，用来设置unlock只能被跨链管理合约调用，接口如下：
```
/* @notice                              Set the ETH cross chain contract as the minter such that the ETH cross chain contract
*                                       will be able to mint tokens to the designated account after a certain amount of tokens
*                                       are locked in the source chain
*  @param ethCrossChainContractAddr     The ETH cross chain management contract address
*/
function setMinter(address ethCrossChainContractAddr) onlyManager
```

代码示例：
```
function setMinter(address ethCrossChainContractAddr) onlyManager public {
    minter = ethCrossChainContractAddr;
    emit SetMinterEvent(ethCrossChainContractAddr);
}
```

绑定目标链的资产合约接口bindContractAddrWithChainId接口也只能被该erc20的管理员调用，用来设置该erc20合约绑定的目标链上的合约，只有目标链上与该erc20绑定的合约的请求才会被该erc20接受。
接口如下
```
/* @notice              Bind the target chain with the target chain id
*  @param chainId       The target chain id
*  @param contractAddr  The specific contract address in bytes format in the target chain
*/
function bindContractAddrWithChainId(uint64 chainId, bytes memory contractAddr) onlyManager
```

实现示例：
```
function bindContractAddrWithChainId(uint64 chainId, bytes memory contractAddr) onlyManager public {
	require(chainId >= 0, "chainId illegal!");
	contractAddrBindChainId[chainId] = contractAddr;
	emit BindContractAddrWithChainIdEvent(chainId, contractAddr);
}
```

lock接口如下：

```
/* @notice                  This function is meant to be invoked by the user,
*                           a certin amount teokens will be burnt from the invoker/msg.sender immediately.
*                           Then the same amount of tokens will be mint at the target chain with chainId later.
*  @param toChainId         The target chain id
*
*  @param toUserAddr        The address in bytes format to receive same amount of tokens in target chain
*  @param amount            The amount of tokens to be crossed from ethereum to the chain with chainId
*/
function lock(uint64 toChainId, bytes memory toUserAddr, uint64 amount)
```

以下是一个具体的示例以及注释解释
```
function lock(uint64 toChainId, bytes memory toUserAddr, uint256 amount) public returns (bool) {
    /*
	1. 构造跨链交易的参数，这个参数最终会发送给该资产合约绑定的目标链的资产合约的unlock
	*/
	TxArgs memory txArgs = TxArgs({
		toAddress: toUserAddr,
		amount: amount
	});
	bytes memory txData = _serializeTxArgs(txArgs);
    /*
	2. 将用户的erc20token锁定在本合约
	*/
	require(transfer(amount), "Burn msg.sender ERC20 tokens failed");
	/*
	3. 调用跨链管理合约，跨链管理合约会生成可以安全发送到目标链的消息
	*/
	bytes memory method = bytes("unlock");
	EthCrossChain ecc = EthCrossChain(minter);
	require(ecc.crossChain(toChainId, contractAddrBindChainId[toChainId], method, txData), "EthCrossChain crossChain executed error!");
	/*
	4. 生成lock的event
	*/
	emit LockEvent(address(this), toChainId, contractAddrBindChainId[toChainId], txData);
	return true;
}
```
unlock接口由跨链管理合约调用，仅仅可以被跨链管理合约调用，接口如下：
```
/* @notice                  This function is meant to be invoked by the ETH crosschain management contract,
*                           then mint a certin amount of tokens to the designated address since a certain amount
*                           was burnt from the source chain invoker.
*  @param argsBs            The argument bytes recevied by the ethereum business contract, need to be deserialized.
*                           based on the way of serialization in the source chain contract.
*  @param fromContractAddr  The source chain contract address
*  @param fromChainId       The source chain id
*/
function unlock(bytes memory argsBs, bytes memory fromContractAddr, uint64 fromChainId) onlyMinter
```

以下是一个具体的示例，其中有注释解释：
```
function unlock(bytes memory argsBs, bytes memory fromContractAddr, uint64 fromChainId) onlyMinter public returns (bool) {
    /*
	1. 跨链交易请求方构造的跨链交易参数
	*/
	TxArgs memory args = _deserializTxArgs(argsBs);
    /*
	2. 检查请求是否由该资产合约绑定的目标链的资产合约发起
	*/
	require(Utils.equalStorage(contractAddrBindChainId[fromChainId], fromContractAddr), "From contract address error!");
	/*
	3. 将锁定在本合约的erc20释放到指定的账户
	*/
	require(transfer(Utils.bytesToAddress(args.toAddress), args.amount), "mint ERCT in unlock method failed!");
	/*
	4. 生成unlock的event
	*/
	emit UnlockEvent(fromContractAddr, fromChainId, args.toAddress, args.amount);
	return true;
}
```

同样需要在跨链的目标链上如ontology上开发部署该erc20合约对应oep4资产合约。请参考ontology开发手册中关于oep4合约跨链开发指导部分。

### 配置合约

在部署好erc20合约后，需要部署合约者，也就是erc20合约的管理员首先调用合约的setMinter和bindContractAddrWithChainId来配置合约。


## 许可证

Ontology遵守GNU Lesser General Public License, 版本3.0。 详细信息请查看项目根目录下的LICENSE文件。
