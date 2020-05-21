<h1 align="center">多签签名工具</h1>
<h4 align="center">Version 1.0 </h4>

[English](./redeem_tool_guide.md) | 中文

## 引言

​	Alice、Bob和Carl准备好了自己的比特币多签钱包和中继链钱包，正准备开展比特币跨链业务，可是发现他们缺少和中继链交互的工具，需要对中继链返回的交易进行签名，这时候他们就可以用到这个工具啦！

​	多签签名工具实现了对中继链的监听，会获取中继链返回的需要Alice他们签名的交易，他们每人都启动这样一个工具，监听到对应交易之后就使用自己的私钥自动化签名，这样比特币的跨链流程就完整了。

## 使用方法

### 1. 准备

​	首先，[下载]()对应的可执行文件，然后准备好注册为中继链Relayer的钱包，用于发送交易。

### 2. 配置

​	配置信息和对应解释如下，Alice只需要准备好带“*”的配置即可。

```json
{
  "AllianceJsonRpcAddress": "http://ip:40336", //中继链地址*
  "WalletFile": "/path/to/wallet.dat", //中继链钱包文件*
  "WalletPwd": "pwd", //中继链钱包密码*
  "BtcWalletPwd": "pwd", //比特币钱包密码*
  "AlliaObLoopWaitTime": 2, //监听中继链的间隔时间
  "BtcPrivkFile": "/path/to/btcprivk", //比特币钱包路径*
  "WatchingKeyToSign": "makeBtcTx", //监听关键字
  "ConfigBitcoinNet": "test", //比特币网络类型*
  "ConfigDBPath": "./db", //数据库路径
  "RestPort": 50071, //REST端口
  "SleepTime": 10, //网络阻塞等问题的重试时间
  "AlliaNet": "testnet", //中继链网络类型
  "CircleToSaveHeight": 300, //每N个中继链高度本地记录一次
  "MaxReadSize": 5000000, //每次从数据库读取的最大字节数
  "Redeem": "your redeem in hex", //多签Redeem脚本的十六进制字符串*
  "SignerAddr": "", //监听签名分离模式，签名的REST地址
  "ObServerAddr": "" //监听签名分离模式，监听的REST地址
}
```

​	比特币钱包和中继链钱包采取了同样的格式，可以通过该[工具](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md)将私钥转换为钱包，或者通过Polygon转换，priv文件填写你的WIF格式私钥，生成完毕后记得删除：

```
./polygon account import -w btcprivk --source ./priv --wif
```

### 3. 启动

​	直接运行脚本`bin/start.sh`，然后查看日志`log/spv.log`以查看运行情况

