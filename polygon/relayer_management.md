# Administrator Privilege System of Polygon Chain

The relay chain does not have a transaction fee setting, but there are certain permissions required to communicate with the relay chain. Permissions are granted to individual accounts and each account has a unique public-private key pairs. Each transaction is signed using the account's private key, and on the receiver's end the the signature is verified using the public key to determine the sender of the transaction. This puts in place a transaction system that can be managed conveniently and monitored. The account that can interact with the relay chain is referred to as the **relayer**.

## Relayer Management Contract

### Relayer Registration

The API method used to register a relayer is `registerRelayer`. The method definition is as follows:

```go
type RelayerListParam struct {
	AddressList []common.Address
	Address     common.Address
}
```

|  Parameter  | Description                                           |
| :---------: | ----------------------------------------------------- |
| AddressList | List of addresses to be registered as relayers        |
|   Address   | Address of the party invoking the registration method |

The contract generates an ID for each application, and each ID is sent to the respective account using `notify`. 

### Approve Relayer Application

The `approveRegisterRelayer` API method is used to approve a relayer application. The method definition is as follows:

```go
type ApproveRelayerParam struct {
	ID      uint64
	Address common.Address
}
```

| Parameter | Description                                                                                                                  |
| :-------: | ---------------------------------------------------------------------------------------------------------------------------- |
|    ID     | Application ID                                                                                                               |
|  Address  | Address of the consensus node that sends the transaction, two thirds of the conesensus nodes need to approve the application |

### Removing a Relayer

The `removeRelayer` API method is used to remove an account from the relayer list. The method definition is as follows:

```go
type RelayerListParam struct {
	AddressList []common.Address
	Address     common.Address
}
```

|  Parameter  | Description                                |
| :---------: | ------------------------------------------ |
| AddressList | List of addresses to be removed as relayer |
|   Address   | Address of the party invoking the method   |

The contract generates an ID for each removal request, and each ID is sent to the respective account using `notify`. 

### Approving a Removal Request

The `approveRemoveRelayer` API method is used to aprove a removal request for a relayer. The method definition is as follows:

```go
type ApproveRelayerParam struct {
	ID      uint64
	Address common.Address
}
```

| Parameter | Description                                                                                                                  |
| :-------: | ---------------------------------------------------------------------------------------------------------------------------- |
|    ID     | Application ID                                                                                                               |
|  Address  | Address of the consensus node that sends the transaction, two thirds of the conesensus nodes need to approve the application |


## Relayer Privilege Verification

The relay chain performs privilege verification for the relayer before the transaction enters the transaction pool. By default, only the transactions that are sent by the consensus nodes can enter the transaction pool, but transactions sent by other addresses need to have the relayer privilege for the transactions to enter the transaction pool. 