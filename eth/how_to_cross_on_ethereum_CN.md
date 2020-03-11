<h1 align="center">How to Cross on Ethereum</h1>
<h4 align="center">Version 1.0 </h4>

[English](how_to_cross_on_ethereum.md) | 中文

这里介绍怎么使用以太的跨链平台，包括三个部分，如何将跨链到以太的btc发送回BTC链，如何将eth发送到ontology，如何发送erc20到ontology。

## 将跨链到以太的btc发送回BTC链

### 准备

位于以太上的btc跨链业务合约处理以太和BTC之间的资产跨链，确保以太上部署有btc跨链业务合约。同时如果要将btc发送回BTC链，确保发送账户有从BTC链发送到以太的btc。

### 发送

发送btc到BTC链是通过调用btc跨链业务合约的lock来完成。lock接口如下：

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
```
toChainId           - BTC为1
toUserAddr          - BTC上账户地址，base58格式的btc地址
amount              - BTC金额，以聪为单位
```

下面是go语言版本的调用示例：
```
func InvokeCross2Btc(cfg *config.ServiceConfig) bool {
	ethclient, err := ethclient.Dial(cfg.RestURL)
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - cannot dial sync node, err: %s", err)
		return false
	}
	contractabi, err := abi.JSON(strings.NewReader(contractbtc.BTCTokensABI))
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - err:" + err.Error())
		return false
	}
	txData, err := contractabi.Pack("lock", uint64(cfg.BindBtcChainId), []byte(cfg.Cross2BtcAddress), uint64(cfg.Cross2BtcAmount))
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - err:" + err.Error())
		return false
	}
	contractAddr := common.HexToAddress(cfg.BtcContractAddress)
	nonce := nonceManager.GetAddressNonce(common.HexToAddress(cfg.Signer))
	tx := types.NewTransaction(nonce, contractAddr, big.NewInt(0), uint64(5000000), big.NewInt(30000000000), txData)
	bf := new(bytes.Buffer)
	rlp.Encode(bf, tx)
	ethsigner := tools.NewETHSigner(cfg)
	rawtx := hexutil.Encode(bf.Bytes())
	signedtx, err := ethsigner.SignRawTx(rawtx)
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - sign raw tx error: %s", err.Error())
		return false
	}
	err = ethclient.SendTransaction(context.Background(), signedtx)
	if err != nil {
		fmt.Errorf("InvokeCross2Btc - send transaction error:%s\n", err.Error())
		return false
	}
	return true
}
```
用户也可以在remix上调用lock来发送btc到BTC链，前提是准备好remix环境，下面是remix上调用lock合约的示例：

 ![](pic/eth_cross_2_btc.png)

## 将eth发送到ontology

### 准备

以太上的跨链代理合约负责处理从以太跨链到其他链的请求，确保以太的跨链代理合约已经部署到以太。
确保目标链对于的id，在这里ontology为例，id为3。

### 发送

发送eth到其他链，如ontology，是通过调用代理合约的lock方法来完成。代理合约的lock接口如下：

```
/* @notice                  This function is meant to be invoked by the user,
*                           a certin amount teokens will be locked in the proxy contract the invoker/msg.sender immediately.
*                           Then the same amount of tokens will be unloked from target chain proxy contract at the target chain with chainId later.
*  @param sourceAssetHash   The asset hash in current chain
*  @param toChainId         The target chain id
*
*  @param toAddress         The address in bytes format to receive same amount of tokens in target chain
*  @param amount            The amount of tokens to be crossed from ethereum to the chain with chainId
*/
function lock(address sourceAssetHash, uint64 toChainId, bytes memory toAddress, uint256 amount)
```
```
sourceAssetHash              -    eth的资产hash指定为0000000000000000000000000000000000000000
toChainId                    -    目标链的id，跨链到ontology为例，id为3
toAddress                    -    目标链账户的地址
amount                       -    发送的金额
```

下面是go语言版本的调用示例：
```
func sendEthCrossOnt(index int) bool {
	gasPrice, err := ethclient.SuggestGasPrice(context.Background())
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - get suggest sas price failed error: %s", err.Error())
		return false
	}
	contractabi, err := abi.JSON(strings.NewReader(lock_proxy_abi.LockProxyABI))
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - err:" + err.Error())
		return false
	}
	cross2OntAddress := "ARiwjLzjzLKZy8V43vm6yUcRG9b56DnZtY"
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - err:" + err.Error())
		return false
	}
	assetaddress := common.HexToAddress("0000000000000000000000000000000000000000")
	txData, err := contractabi.Pack("lock", assetaddress, uint64(config.ONT_CHAIN_ID), cross2OntAddress[:], big.NewInt(int64(cfg.Cross2OntAmount)))
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - err:" + err.Error())
		return false
	}
	contractAddr := common.HexToAddress(cfg.ProxyContract)
	signerAccount := ethSigner.GetAccount(index)
	callMsg := ethereum.CallMsg{
		From: signerAccount.Address, To: &contractAddr, Gas: 0, GasPrice: gasPrice,
		Value: big.NewInt(int64(cfg.Cross2OntAmount)), Data: txData,
	}
	gasLimit, err := ethclient.EstimateGas(context.Background(), callMsg)
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - estimate gas limit error: %s", err.Error())
		return false
	}
	nonce := nonceManager.GetAddressNonce(signerAccount.Address)
	tx := types.NewTransaction(nonce, contractAddr, big.NewInt(int64(cfg.Cross2OntAmount)), gasLimit, gasPrice, txData)
	bf := new(bytes.Buffer)
	rlp.Encode(bf, tx)
	rawtx := hexutil.Encode(bf.Bytes())
	signedtx, err := ethSigner.SignRawTx(rawtx, index)
	if err != nil {
		fmt.Errorf("sendEthCrossOnt - sign raw tx error: %s", err.Error())
		return false
	}
	err = ethclient.SendTransaction(context.Background(), signedtx)
	if err != nil {
		ftm.Errorf("sendEthCrossOnt - send transaction error:%s", err.Error())
		return false
	}
	return true
}
```
用户也可以通过remix调用lock来完成交易发送，和上面类似，不在示例。

## 如何发送erc20到ontology

### 准备

以太上的跨链代理合约负责处理从以太跨链到其他链的请求，erc20的跨链也是通过跨链代理合约完成的，确保以太的跨链代理合约已经部署到以太。
确保erc20资产合约遵循跨链协议且已经部署到以太并完成配置。
确保目标链对于的id，在这里ontology为例，id为3。

### 发送

发送erc20到其他链分两个步骤，1. approve erc20到代理合约  2. 调用代理合约lock完成发送

1. approve erc20到代理合约

approve到代理合约的erc20可以被代理合约跨链发送到其他链，遵循跨链协议的erc20合约的approve接口如下：

```
/**
 * @dev See {IERC20-approve}.
 *
 * Requirements:
 *
 * - `spender` cannot be the zero address.
 */
function approve(address spender, uint256 amount)
```
```
address                      -    代理合约地址
amount                       -    金额
```

调用示例和上面类似，不再示例。

2. 调用代理合约lock完成发送

发送erc20到其他链，如ontology，是通过调用代理合约的lock方法来完成。代理合约的lock接口如下：

```
/* @notice                  This function is meant to be invoked by the user,
*                           a certin amount teokens will be locked in the proxy contract the invoker/msg.sender immediately.
*                           Then the same amount of tokens will be unloked from target chain proxy contract at the target chain with chainId later.
*  @param sourceAssetHash   The asset hash in current chain
*  @param toChainId         The target chain id
*
*  @param toAddress         The address in bytes format to receive same amount of tokens in target chain
*  @param amount            The amount of tokens to be crossed from ethereum to the chain with chainId
*/
function lock(address sourceAssetHash, uint64 toChainId, bytes memory toAddress, uint256 amount)
```
```
sourceAssetHash              -    erc20的资产hash
toChainId                    -    目标链的id，跨链到ontology为例，id为3
toAddress                    -    目标链账户的地址
amount                       -    发送的金额
```
调用示例和上面类似，不再示例。

## 许可证

Ontology遵守GNU Lesser General Public License, 版本3.0。 详细信息请查看项目根目录下的LICENSE文件。
