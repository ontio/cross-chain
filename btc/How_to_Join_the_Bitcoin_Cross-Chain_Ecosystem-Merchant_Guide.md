<h1 align="center">如何加入比特币跨链生态：商家篇</h1>
<h4 align="center">Version 1.0 </h4>

[English](https://github.com/ontio/cross-chain/blob/master/btc/How_to_Join_the_Bitcoin_Cross-Chain_Ecosystem-Merchant_Guide.md) | [中文](https://github.com/ontio/cross-chain/blob/master/btc/How_to_Join_the_Bitcoin_Cross-Chain_Ecosystem-Merchant_Guide_CN.md)

## 引言

​	通过联盟链提供的基础设施，可以实现比特币到其他链的转移，下面会以比特币与以太坊为例子，介绍商家如何开展自己的比特币跨链业务。

​	Alice、Bob和Carl是合作伙伴，他们热爱比特币与去中心化技术，希望在比特币上开展更加复杂的业务，但是碍于比特币链的诸多不便，难以实现。于是他们决定利用联盟链的跨链生态，将比特币转移到以太坊上，然后通过以太坊的合约开发自己的业务。

​	跨链的BTC会锁定在商家的多签地址，这意味着需要用户信任商家，商家需要证明自己可信，同时意识到内部资产安全的重要性。

## 跨链的准备工作

1. Alice、Bob和Carl分别准备了自己的**比特币私钥**，并用这些私钥构成了一个P2WSH或者P2SH形式的2/3**多签地址**，多签的作用主要是锁住比特币。

2. 然后，他们需要按照模板开发一个特殊的ERC20代币**合约**（以下称为EBTC），部署在以太坊上，用来接收跨链的比特币。每个人都是用自己的私钥对合约和Redeem脚本进行签名，然后向中继链进行注册，这可以使用[工具](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md)实现。

3. 接下来，下载多签的[**签名工具**](https://github.com/ontio/cross-chain/blob/master/btc/redeem_tool_guide.md)，它可以监听联盟链，当有比特币从以太坊合约返回的时候，各自使用自己的私钥对解锁交易签名，这样比特币就回到了客户自己的手里。

4. 最后，每人都创建一个联盟链跨链生态的**钱包**，用来向联盟链提交自己的签名。

## 开启跨链业务

### 商家的工作

​	用户将比特币发送到商家的多签，称之为“锁定”，在锁定之后，跨链生态会在目标链上商家的合约中为用户发放BTC，经过这个过程，BTC相当于从比特币链转移到了目标链。

​	当BTC从其他链回到比特币链的时候，中继链会帮助商家构造解锁交易，将商家多签中的BTC退回给用户，这中间产生的手续费由用户承担，商家的多签不会有财产的损失。构造完的交易将会被签名工具监听到，然后多签中的每个人会分开对该交易签名，然后通过调用中继链合约的形式，将签名提交到中继链上，最终中继链会将交易和签名合并，完整的交易会被广播到比特币网络，所以商家不需要维护多签的UTXO，中继链将会代替商家维护这些信息。

### BTC到以太坊

#### 第一步，部署以太坊合约

​	将准备好的EBTC合约部署到以太坊上，准备接收比特币，合约模板请见[此]()，修改合约内的Redeem脚本为商家自己的多签脚本即可。

#### 第二步，启动签名工具

​	按照[说明](https://github.com/zouxyan/cross-chain/blob/master/btc/redeem_tool_guide.md)，把自己的私钥加密后导入工具，启动工具监听联盟链并提交自己的签名。

#### 第三步，发送跨链交易

​	按照特定的格式[构造交易](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md)，发送比特币到多签地址，活跃在跨链生态中的Relayer就会把这些跨链交易转发到联盟链，最终比特币会准确地发送到EBTC指定的账户中了。

​	Alice在交易中写入了EBTC合约的地址、自己以太坊的地址和以太坊的chain id，并发送了1BTC到多签地址，在这笔交易拥有6个确认之后，Alice发现自己多了1个EBTC，而发送到多签地址的1BTC则被锁住。

​	Alice想把自己的1EBTC转回比特币链，于是直接调用EBTC合约的接口，填入要转回的金额、比特币地址、比特币chain id等信息，一段时间后，发现签名工具对一笔交易进行了签名，这笔交易就是多签用来释放Alice比特币的，同时另外两人的签名工具也会签名，最终这笔交易会被Relayer成功广播到比特币网络中，这时候Alice会看到这笔交易，但是她发现转回来的比特币不足1BTC，因为这笔多签交易的手续费需要Alice支付，这是跨链生态所要求的，不过手续费并不多，Alice表示完全能接受。

​	经过测试，三人开始基于EBTC开发自己的比特币业务合约，他们终于能在以太坊智能合约上实现自己天马行空的想法了，这多亏了跨链生态！

### BTC到本体

​	同样地，按照协议开发一本合约，它将作为BTC在本体上的映射，与比特币保持一比一的关系，也可以通过修改[模板]()的Redeem脚本后使用，这里称作OBTC。将准备好的OBTC合约部署到本体网络上，可以通过[smartx](https://smartx.ont.io/)其余步骤和以太坊类似。