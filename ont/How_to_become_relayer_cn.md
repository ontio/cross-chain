# ont-relayer

[English](./How_to_become_relayer.md) | 中文

Relayer在本体团队提出的跨链协议中起到了至关重要的作用，其负责监听这些跨链事件，当监听到一笔跨链交易时，Relayer根据链ID和区块高度去源链上获取交易和区块头，然后将交易和区块头提交到目标链，从而实现了中继链与外界完成信息的交互。

## 架构
Ont-relayer实现了Ontology网络的监听，能识别并转发跨链交易，向中继链提交区块头，并获取收益，同时监听中继链，广播中继链构造的跨链交易，从而实现Ontology与其他公有链信息的交互。

Relayer会持续扫描每个Ontology网络区块，找到跨链区块并获取交易的梅克尔证明，一同提交给中继链。中继链会构造Ontology的返回交易，Relayer会把交易广播到Ontology网络中。只要存在一个Relayer在工作，那么整个Ontology跨链生态就可以持续运作。

## 准备
1. 申请联盟链钱包
申请一个中继链的钱包，如果有本体钱包直接使用即可，二者钱包是通用的。

2. 注册Relayer
然后向联盟链注册自己的地址为Relayer。

## 启动Relayer
第一步，下载Relayer的可执行文件，解压至特定位置；
第二步，更改配置信息，如下：

```json
	{
	  "AliaJsonRpcAddress":"http://172.168.3.73:40336", // 联盟链rpc地址
	  "SideJsonRpcAddress":"http://172.168.3.73:20336", // Ontologyrpc地址
	  "AliaChainID": 0,	// 联盟链编码
	  "SideChainID": 3,	// Ontology链编码
	  "AliaWalletFile": "./wallet1.dat",	// 联盟链钱包
	  "SideWalletFile": "./wallet2.dat",	// Ontology钱包
	  "DBPath": "boltdb",			// 存储位置
	  "ScanInterval": 1,		
	  "RetryInterval": 1,
	  "GasPrice":0, // GasPrice
	  "GasLimit":200000 // GasLimit
	}
```
第三部，输入./ont_relayer --loglevel 0 --cliconfig “path”执行Relayer可执行文件以运行ont
-relayer。
