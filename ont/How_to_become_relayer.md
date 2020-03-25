<h1 align=center> ONT Relayer </h1>

English | [中文](./How_to_become_relayer_CN.md)

The **relayer** plays a very important role in the Ontology cross chain ecosystem. It monitors the relay chain for cross chain transactions to Ontology. When a transaction is picked up by the relayer, it fetches the respective block header using the chain ID and block height, and then transfers it to the target chain. It acts the means of communication between the relay chain and the outside world.

## Framework

The ONT relayer monitors the Ontology chain for cross chain transactions as well. When a cross chain transaction occurs, the relayer sends the block header to the relay chain, while also collecting small incentives. It also monitors the relay chain  and broadcasts the transactions created by the relay chain, thereby facilitating communication between Ontology and other chains.

The relayer consistently scans the Ontology network for cross chain transactions, and then fetches the respective block header and the merkle proof, which is then transferred to the relay chain. The relay chain creates the response transaction to Ontology, and the relayer broadcasts this transaction to the Ontology network. The entire cross chain ecosystem can continue to function normally even with just one active relayer.

## Setup

1. Create a relay chain wallet 

Create a wallet to be used on the relay chain. If you have an existing Ontology wallet it can be used on the relay chain. The two wallets are mutually interchangeable.

2. Register the relayer
   

Next, register your address as the **relayer** on the relay chain.

## Enabling the Relayer

**Step 1:** Download the executable file for the relayer and extract it to a specific directory.

**Step 2:** Modify the configuration settings. Here is a sample configuration.

```json
	{
	  "AliaJsonRpcAddress":"http://172.168.3.73:40336", // Relay chain RPC address
	  "SideJsonRpcAddress":"http://172.168.3.73:20336", // Ontologyrpc地址
	  "AliaChainID": 4,	// 联盟链编码
	  "SideChainID": 2,	// Ontology链编码
	  "AliaWalletFile": "./wallet1.dat",	// 联盟链钱包
	  "SideWalletFile": "./wallet2.dat",	// Ontology钱包
	  "DBPath": "boltdb",			// 存储位置
	  "ScanInterval": 1,		
	  "RetryInterval": 1,
	  "GasPrice":0, // GasPrice
	  "GasLimit":200000 // GasLimit
	}
```

**Step 3:** Use the following command with the configuration file path to enable the relayer.

```shell
./ont_relayer --loglevel 0 --cliconfig “path”
```
