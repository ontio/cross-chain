<h1 align="center">如何加入比特币跨链生态：用户篇</h1>

<h4 align="center">Version 1.0 </h4>

[English](./How_to_Join_the_Bitcoin_Cross-Chain_Ecosystem-Users_Guide.md) | 中文

## 引言

跨链用户指的是有需求将BTC从比特币网络转移到其他链的人或组织，他们需要找到提供跨链服务的供应商，获取供应商的多签等信息，用户需要信任供应商。  	

本文档将以跨链用户的角度，讲述如何实现自己BTC的跨链转移。比如你需要将自己的1BTC转到某条链上，你需要做的准备工作以及具体的操作步骤。

## 准备工作

首先，某些组织可能开展了你感兴趣的跨链业务，你需要挑选其中一个，并获取其比特币多签地址和目标链的BTC代币合约地址；然后，获取跨链交易构造工具，或者按照协议自己构造交易；最后，生成自己在目标链的地址，用于接收跨链来的比特币。

## 操作实例

在这里我们以测试网BTC到以太坊测试网为例。

在工具上填入需要的信息，并通过工具构造跨链交易，工具的使用请[移步](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md)。将工具显示的交易通过任何工具广播出去，比如该[网站](https://tbtc.bitaps.com/broadcast)。剩下的就是等待交易的六个确认，然后不久就能看到目标链的账户中多了对应数额的BTC代币。

我选择了信任的测试供应商，他们的信息如下表。

| 信息            | 地址                                                         |
| --------------- | ------------------------------------------------------------ |
| 多签地址        | 2NB3HZzhqntJz5cHJcViiR7rkXUUdqhLYkD 或者 tb1qy94qnjuwu5w6r2g74z2z25khjdkgs6ssk5rjnyqrvcvpds8f7x9shrfspn |
| 以太BTC合约地址 | 0xa389c761307bde552e0329ae0915ee998da61095                   |
| 本体BTC合约地址 | 待定                                                         |

### 实例1：BTC到以太坊

#### 1. BTC到以太

##### 1.1. 跨链准备

  1. **获取BTC**：准备好一个比特币地址，在进行之前先在[BTC测试网水龙头](https://testnet-faucet.mempool.co/)中获取测试网比特币；
  2. **获取ETH**：准备好一个以太坊地址，要获取一定的ETH，方便支付调用合约的Gas，这里使用的是[Ropsten](https://teth.bitaps.com/)；

##### 1.2. 发送交易

通过[工具](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md)发送即可，教程见工具文档。发送完毕后可以通过[以太坊浏览器](https://ropsten.etherscan.io/)查看账户在合约里的BTC代币数目。

#### 2. BTC返回比特币

  详情请见[这里](https://github.com/ontio/cross-chain/blob/master/ethereum/how_to_cross_on_ethereum_CN.md)



### 实例2：BTC到本体

#### 1. BTC到本体

##### 1.1. 准备工作

  1. **获取BTC**：在进行之前先在[BTC测试网水龙头](https://coinfaucet.eu/en/btc-testnet)中获取测试网比特币；

  2. **获得本体地址**：需要一个[本体](https://ont.io/)的地址，用来在目标链上接收跨链后的比特币，跨链到ONT的BTC会被一比一映射到一种[OEP4](https://github.com/ontio/OEPs/blob/master/OEPS/OEP-4.mediawiki)代币；

  3. **获得合约地址**：这里使用上面表格中的合约[OBTC](https://github.com/zouxyan/btc_crosschain_demo)(hash:)，

  4. **准备工具**：为了确定BTC跨链成功，建议使用Chrome插件[Cyano Wallet](https://chrome.google.com/webstore/detail/cyano-wallet/dkdedlpgdmmkkfjabffeganieamfklkm)，导入用户的ONT账户。为了连接到ONT的测试网络，请进行如下配置：

     - 首先，导入或者创建钱包，钱包的使用方法请[移步](https://dev-docs.ont.io/#/docs-cn/cyano/01-chrome-wallet)或者观看[使用教程中文](https://www.youtube.com/watch?v=u_MtHccKaNQ)、[教程英文](https://www.youtube.com/watch?v=S2qk-Gkrs9s)；

       <div align=center><img width="300" height="400" src="./pic/cyano-login.png"/></div>

     - 点击右上角的设置按钮(小齿轮)，按图片中红框内信息配置本体测试网（），点击最下方的Save按钮，即可连接到本体测试网络；

       <div align=center><img width="300" height="400" src="./pic/cyano-confignet.png"/></div>

       

     - 配置网络完成后，需要添加BTC映射的合约地址，还是点击设置按钮，然后选择图中红框的**OEP-4 TOKENS**，然后单击ADD按钮，将合约地址输入到script hash中，点击CONFIRM，即可看到OBTC的合约已经被添加了，点击BACK，然后点击SAVE；

       <div align=center><img width="300" height="400" src="./pic/cyano-configoep4.png"/></div>

##### 1.2. 发送交易

使用[工具](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md)构造并发送跨链交易。交易发送完毕，比特币的六个区块确认之后，即可在Cyano上看到OBTC（BTC映射）的余额增加了，现在用户可以在ONT链上交易自己的比特币了（先得有ONG），本体的交易处理速度更快，更适合高频的BTC交易，或者为BTC编写智能合约，当然用户可以把自己的BTC从ONT链上转回BTC链上。

<div align=center><img width="300" height="400" src="./pic/cyano-btcx.png"/></div>

#### 2. BTC返回比特币

要转回BTC，用户需要调用OBTC合约中的lock方法，填写对应的参数。

以本体链为例，用户可以通过[smartx](https://smartx.ont.io/)调用本体的BTC映射合约，将BTC转回比特币网络，将上文中的[OBTC](https://github.com/zouxyan/btc_crosschain_demo)合约代码复制到smartx的编辑器中，编译并调用即可，详细可见smartx的[使用教程](https://ontio.github.io/documentation/)。

除此之外，用户还可以使用本体链的[SDK](https://github.com/ontio/ontology-go-sdk)调用智能合约，示例如下。

```go
  package main
  
  import (
  	"fmt"
  	sdk "github.com/ontio/ontology-go-sdk"
  	"github.com/ontio/ontology-go-sdk/utils"
  )
  
  var (
  	btcxCode      = "your avm code"
  	yourBtcAddr   = "your btc address"
  	yourBtcAmount = 8000
  	ontRpcAddr    = "http://ip:20336"
  	walletPath    = "path to your wallet.dat"
  	walletPwd     = "password of your wallet"
  )
  
  func main() {
  	ontSdk := sdk.NewOntologySdk()
  	ontSdk.NewRpcClient().SetAddress(ontRpcAddr)
  	wallet, err := ontSdk.OpenWallet(walletPath)
  	if err != nil {
  		fmt.Printf("failed to open wallet: %v", err)
  		return
  	}
  
  	acc, err := wallet.GetDefaultAccount([]byte(walletPwd))
  	if err != nil {
  		fmt.Printf("failed to get account: %v", err)
  		return
  	}
  
  	contractAddr, err := utils.GetContractAddress(btcxCode)
  	if err != nil {
  		fmt.Printf("failed to parse address from bytes: %v", err)
  		return
  	}
  
  	_, err = ontSdk.NeoVM.InvokeNeoVMContract(0, 21600000, acc, contractAddr,
  		[]interface{}{"lock", []interface{}{1, acc.Address[:], yourBtcAddr, yourBtcAmount}})
  	if err != nil {
  		fmt.Printf("failed to invoke contract: %v", err)
  		return
  	}
  	fmt.Printf("successful to transfer %d btc back to bitcoin-net", yourBtcAmount)
  }
```

上述demo中，有一些信息需要使用者自己填入：

  - 智能合约的avm code，需要编译合约，比如smartx编译之后在右边即可看到avm字节码，或者使用[Neptune](https://github.com/ontio/ontology-python-compiler)编译；

  - 用户用来接收BTC的地址；

  - ONT节点的Rpc地址，如果使用本体测试网，IP可填138.91.6.125；

  - ONT钱包文件的路径以及钱包的密码。

    在合约调用成功之后，用户只需等待一段时间，即可看到BTC在比特币链上到账。