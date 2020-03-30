<h1 align=center> Ontology Cross Chain Contract Development </h1>

English | [中文](How_to_new_cross_chain_asset_cn.md)

The Ontology multi-chain ecosystem supports cross chain contracts. A cross chain contract implements functionality that operates by interacting with two or more chains. For e.g. an OEP4 asset contract that can be transferred between Ontology and Ethereum, or a dApp contract wherein a part of the business logic takes place on Ontology and another part is carried out on Ethereum.

### Cross Chain Contract Overview

A cross chain system in fact consists of multiple smart contracts. For a dApp that functions in a cross chain environment, the developer needs to deploy smart contracts separately on the respective chains.

Cross chain contract development is composed of two elements, the business logic of the contract that is implemented on the respective chains, and cross chain logic that actually carries out cross chain transfers and interaction.

The business logic is developed conventionally based on the application, while the cross chain logic involves a sequence in which execution takes place first on the local chain, and then on the respective target chain. 

### Cross Chain Interface

Developers need to concern themselves with only one interface when working with cross chain contracts, that is the `createCrossChainTx` method of the management contract. Let's consider a cross chain scenario in which there are two chains, the local chain A and a target chain B. This interface can be used to store the execution result on chain A in the merkle tree. A miner generates the merkle proof for this transaction and sends it to the cross chain management contract deployed on chain B. The cross chain management contract verifies the merkle proof, and then invokes the respective methods in the smart contract on chain B based on the parameters.

### Sample Cross Chain Contract

A developer intends to deploy a new asset on two chains, chain A and chain B, and wants to ensure that the assets on chain A and chain B are mutually interoperable. That is, the asset they deploy on chain A and chain B can be used and transferred between chains A and B conveniently.

The actual implementation of this interoperability of the asset between chain A and chain B involves adding the `lock` and `unlock` methods to the standard **OEP-4** contract. The user invokes the `lock` method of the contract on chain A to lock the transfer amount in this contract. This method then invokes the cross chain management contract that in turn invokes the `unlock` method of the contract on chain B to release assets and transfer them to the user's account on chain B.

The `lock` method is implemented as follows:

```python
def lock(to_chainId, from_address, to_address, amount):
    """
    lock some amount of tokens of this contract, call cross chain method to release to_amount of tokens of another chain's contract
    :param to_chain_id: chain id of destination chain
    :param address: address of caller
    :param to_amount: amount to lock
    :return:
    """
    assert (amount >= 0)
    assert (CheckWitness(from_address))
    assert (not isPaused())
    # eth address format:0x673dfa9caf9145fdbef98e9d9874f36e63d8a5b4,length is 42
    assert (len(to_address) != 0)

    Put(ctx, concat(BALANCE_KEY, from_address), Sub(balanceOf(from_address), amount))
    Put(ctx, TOTAL_SUPPLY_KEY, Sub(totalSupply(), amount))

    btcRedeemScriptBytes = getBtcRedeemScript()
    argsList = [to_address, amount, btcRedeemScriptBytes]

    input_bytes = _serialzieArgs(argsList)
    to_contract = getContractAddrWithChainId(to_chainId)
    param = state(to_chainId, to_contract, "unlock", input_bytes)
    assert (Invoke(0, CROSS_CHAIN_CONTRACT_ADDRESS, "createCrossChainTx", param))

    LockEvent(to_chainId, from_address, to_address, amount)
    return True
```

`LockEvent` parameters:

| Parameter            | Description                                               |
| -------------------- | --------------------------------------------------------- |
| to_chain_id          | Target chain ID                                           |
| destination_contract | Reversed contract address on the target chain             |
| address              | Invocation address, mining fee deducted from this address |
| amount               | Transfer amount                                           |


This way this method carries out the transfer is by first locking the user's assets on chain A and then invoking the `createCrossChainTx` method of the cross chain management contract. This method takes six parameters:

| Parameter            | Description                                          |
| -------------------- | ---------------------------------------------------- |
| to_chain_id          | Target chain ID                                      |
| destination_contract | Reversed contract address on the target chain        |
| function             | Method to be invoked in the contract on target chain |
| input_bytes          | Serialized parameters                                |


This method invokes the contract on the target chain whose address is passed as a parameter. The inpur parameters are serialized into `input_bytes`.

The `unlock` method is implemented as follows:

```python
def unlock(params, fromContractAddr, fromChainId):
    """
    lock some amount of tokens of this contract, call cross chain method to release to_amount of tokens of another chain's contract
    :param fee: miner fee of this cross chain tx
    :param to_chain_id: chain id of destination chain
    :param address: address of caller
    :param to_amount: amount to lock
    :return:
    """
    res = _deserialzieArgs(params)
    toAddress = res[0]
    value = res[1]
    assert(fromContractAddr == getContractAddrWithChainId(fromChainId))
    assert (value >= 0)
    assert (isAddress(toAddress))
    assert (CheckWitness(CROSS_CHAIN_CONTRACT_ADDRESS))

    Put(ctx, concat(BALANCE_KEY, toAddress), Add(balanceOf(toAddress), value))
    Put(ctx, TOTAL_SUPPLY_KEY, Add(totalSupply(), value))
    UnlockEvent(toAddress, value)
    return True
```

The method first verifies the contract address of chain A, takes the input parameters `input_bytes`, obtains the `input_map` by deserializing it, and then proceeds with the consequent logic to release assets on chain B and transfer them to the user's address.

> Note: Since this method is the destination method of the cross chain transfer, it can only be invoked by the management contract on the relay chain.