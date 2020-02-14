<h1 align="center">侧链接入跨链指南</h1>

## 概述

本体跨链允许任何区块链加入，想要加入本体跨链的区块链需要实现一系列的接口，这些接口既可以以智能合约的方式来实现，也可以在底层直接实现。该链还需要提交其区块头格式，序列化和反序列话方式和签名验证方式，以便中继链可以解析并验证其区块头。同时提交其账本或交易的merkle tree构造和验证方式，使中继链可以验证其链上发生的真实交易。

## 需实现的接口

### 区块头同步合约：

SyncGenesisHeader：

同步中继链的创世区块头（或某个周期切换的区块头），在合约初始化是被调用，只能被调用一次。存储并解析创世区块，取得中继链此时的共识节点列表。相关代码可以参考https://github.com/siovanus/ontology/tree/master/smartcontract/service/native/cross_chain/header_sync

SyncBlockHeader

持续同步中继链的区块头，主要包括周期切换的关键区块头和跨链交易发生的区块头。relayer通过该接口同步中继链区块头。存储并解析区块头，如果发现共识节点变更，则存储中继链此时的共识节点列表。相关代码可以参考：https://github.com/siovanus/ontology/tree/master/smartcontract/service/native/cross_chain/header_sync

### 跨链管理合约：

CreateCrossChainTx

创建跨链交易，该接口主要用于业务智能合约在需要跨链功能时调用。构造一笔跨链交易，此交易具有唯一的自增ID，并将交易参数写入merkle tree。相关代码可以参考：https://github.com/siovanus/ontology/tree/master/smartcontract/service/native/cross_chain/cross_chain_manager

### 模块和接口之间的关系：

![module](/resources/module.png)

ProcessCrossChainTx

处理跨链交易，该接口用于该链接受来自其他链的跨链交易，由relayer同步跨链交易的merkle proof时调用。接口按照高度找到该跨链交易的merkle root（存在于区块头中），验证该跨链交易的真实性，验证通过则解析跨链参数，调用目标链上的业务合约。相关代码可以参考：https://github.com/siovanus/ontology/tree/master/smartcontract/service/native/cross_chain/cross_chain_manager

## 需要提交到跨链理事会的内容

### 本链的创世区块头（或者中间某个关键区块头）

提交本链的创世区块头或者中间某个关键区块头，使中继链能够验证后来提交的其他区块头。

### 本链区块头的结构、验证和解析方式

提交本链区块头的结构、验证和解析方式，使得中继链能够验证和解析其区块头。

### 本链跨链交易merkle tree的构造和验证方式

提交本链跨链交易merkle tree的构造和验证方式，使得中继链能够验证其跨链交易。