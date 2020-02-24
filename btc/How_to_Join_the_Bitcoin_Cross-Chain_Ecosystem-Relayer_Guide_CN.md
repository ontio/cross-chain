<h1 align="center">如何加入比特币跨链生态：Relayer篇</h1>
<h4 align="center">Version 1.0 </h4>

[English](https://github.com/ontio/cross-chain/blob/master/btc/How_to_Join_the_Bitcoin_Cross-Chain_Ecosystem-Relayer_Guide.md) | 中文

## 引言

​	在跨链生态中，Relayer起到了举足轻重的作用，它会转发跨链交易和区块头，实现中继链与外界的信息交互，可以说是生态的对外沟通的媒介。

​	下面将介绍如何启动自己的Relayer

## 架构

<img src="/Users/zou/go/src/github.com/ontio/cross-chain/btc/pic/relayer.png" style="zoom:50%;" />

​	比特币Relayer实现了比特币网络的监听，能识别并转发跨链交易，向中继链提交区块头，并获取收益，同时监听中继链，广播中继链构造的跨链交易，是实现比特币跨链的关键部分。

​	Relayer会持续扫描每个比特币区块，找到跨链交易并获取交易的梅克尔证明，一同提交给中继链，对还没有足够确认数的交易，Relayer会存储下来，等到有足够的确认后，再发送给中继链。Relayer负责提交比特币区块头到中继链，能处理常见的分叉情况，稳定并正确地向中继链提交区块头。中继链会构造比特币的返回交易，Relayer会把交易广播到比特币网络中。只要存在一个Relayer在工作，那么整个比特币跨链生态就可以持续运作。

## 准备

### 1. 申请联盟链钱包

​	申请一个中继链的钱包，如果有本体钱包直接使用即可，二者钱包是通用的。

### 2. 注册Relayer

​	然后向联盟链注册自己的地址为Relayer 。

### 3. 启动一个比特币全节点

​	要启动Relayer，必须要拥有可用的比特币RPC功能。

## 启动Relayer

​	第一步，下载Relayer的可执行文件，解压至特定位置；

​	第二步，更改配置信息，如下：

```json
{
  "btc_ob_conf": {
    "net_type": "test", //比特币的网络类型，比如regtest、test和main
    "btc_ob_loop_wait_time": 3, //监听比特币网络的间隔时间
    "btc_json_rpc_address": "http://localhost:18443", //你的比特币全节点的rpc地址*
    "user": "", //比特币rpc用户名*
    "pwd": "", //比特币rpc密码*
    "start_height": 0 //监控比特币起始高度
  },
  "allia_ob_conf": {
    "alliance_json_rpc_address": "http://ip:40336", //要连接的中继链地址*
    "allia_ob_loop_wait_time": 3, //监听中继链的时间间隔
    "watching_key": "btcTxToRelay", //监听中继链的事件关键词
    "wallet_file": "relayer_btc/wallet.dat", //你中继链钱包的路径*
    "wallet_pwd": "", //钱包的密码，也可以在运行时以flag的形式传入*
    "net_type": "testnet", //网络类型
    "waiting_cycle": 300 //每多少区块就记录一下高度，下次启动从该高度开始
  },
  "retry_duration": 1, //重试广播交易的时间
  "retry_times": 0, //重试次数，0代表无穷次
  "retry_db_path": "relayer_btc/db", //DB路径*
  "log_level": 0, //日志level, 0代表TRACE、1代表DEBUG、2代表INFO、3代表WARN、4代表ERROR
  "sleep_time": 10, //当网络出现问题的重试间隔
  "max_read_size": 5000000, //最多一次从DB中读取的字节数
  "retry_cci_dura": 5, //重试确认数不足的交易的时间间隔
  "send_headers_dura": 5 //发送区块头的时间间隔
}
```

​	用户需要更改的仅仅是后面追加“*”的配置。

​	第三步，启动bin目录下的start-btcrelayer.sh。也可以直接通过可执行文件bin/run_btc_relayer直接运行比特币Relayer，比如以下命令：

```shell
bin/run_btc_relayer -wallet-pwd="pwd" -conf-file=conf.json
```

