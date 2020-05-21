<h1 align="center">Multi-Signature Signing Tool: Redeem Tool Guide</h1>
<h4 align="center">Version 1.0 </h4>

English | [中文](./redeem_tool_guide_CN.md)

## Introduction

Say Alice, Bob, and Carl have already created their BTC and relay chain wallets and they are ready to start developing and testing their cross chain service. But they realize that they don't really have a means to communicate with the relay chain, say for actions like receiving the cross chain transactions sent by the relay chain or signing them. This is where they can start using this tool.

The multi-signature signing tool monitors the transactions on the relay chain and fetches the ones that need to be signed by the collaborators, each of whom run this tool individually. The collaborators use their respective private keys to sign the transactions fetched from the relay chain. This completes a cross chain BTC transaction.

## Usage Guide

### 1. Setup

Download the executable file using [this]() link, and create a wallet that is to be registered as the relayer on the relay chain. It will be used to create and send transactions.

### 2. Configuration

A sample configuration could look something like the following sample.

```json
{
  "AllianceJsonRpcAddress": "http://ip:40336", //Relay chain address*
  "WalletFile": "/path/to/wallet.dat", //Relay chain wallet file*
  "WalletPwd": "pwd", //Relay chain wallet password*
  "BtcWalletPwd": "pwd", //BTC wallet password*
  "AlliaObLoopWaitTime": 2, //Time interval for monitoring the relay chain
  "BtcPrivkFile": "/path/to/btcprivk", //BTC wallet path*
  "WatchingKeyToSign": "makeBtcTx", //Keyword to be monitored
  "ConfigBitcoinNet": "test", //BTC network type*
  "ConfigDBPath": "./db", //DB path
  "RestPort": 50071, //REST port
  "SleepTime": 10, //Connection attempt time interval in case of network anomaly
  "AlliaNet": "testnet", //Relay chain network type
  "CircleToSaveHeight": 300, //Block interval to record block height, start monitoring from this height the next time relayer is enabled
  "MaxReadSize": 5000000, //Max. bytes to be fetched from DB in a single read operation
  "Redeem": "your redeem in hex", //Redeem script hex value*
  "SignerAddr": "", //REST address of the signer, detached mode signature monitoring 
  "ObServerAddr": "" //REST address of the observer, detached mode signature monitoring
}
```

> The default configuration settings can be used by only modifying the fields marked with an asterisk*


The BTC and relay chain wallets use the same format. Private keys can be converted to wallets using this [tool](https://github.com/ontio/cross-chain/blob/master/btc/cross-chain_transaction_construction_tool_user_manual.md). The same conversion can be performed using the Polygon (relay chain). You can enter your WIF private key in the `priv` file which can then be used to generate a wallet. Please ensure that you delete the contents of the file after generating the wallet.

```shell
./polygon account import -w btcprivk --source ./priv --wif
```

### 3. Enabling the tool

Run the `bin/start.sh` shell script to enable the redeem tool. You can refer to the `log/spv.log` file to refer to the execution logs to monitor the operation. 

