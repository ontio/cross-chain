<h1 align=center> Polygon Chain Governance Mechanism </h1>

## Side Chain Management

### Register New Side Chain

The term **side chain** refers to a chain that is interested in joining the cross chain ecosystem. Every side chain that applies to join the cross chain ecosystem on the relay chain requires a two-thirds approval by the consensus nodes.


The `registerSideChain` method can be used to send a registration application to the relay chain for registering a side chain. The parameters are as follows:


```go
type RegisterSideChainParam struct {
	Address      string    
	ChainId      uint64    
	Router       uint64    
	Name         string    
	BlocksToWait uint64    
	CCMCAddress  []byte    
}
```

|  Parameter   | Description                                                                                                                                                                                     |
| :----------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   Address    | The Ontology wallet address used to register the chain, address has the privileges to modify registered chain information                                                                       |
|   ChainId    | Chain ID of this chain in the cross chain system                                                                                                                                                |
|    Router    | The routing protocol of the chain, existing routing protocols - **1:BTC**, **2:ETH**, **3:ONT**, isomorphic chains can select existing protocol, rest must first define the respective protocol |
|     Name     | Name of the blockchain network                                                                                                                                                                  |
| BlocksToWait | No. of blocks to wait for in order to confirm finality                                                                                                                                          |
| CCMCAddress  | Cross chain management contract address                                                                                                                                                         |


The `approveRegisterSideChain` method is used to sign and submit an approval vote for a side chain registration application. An application needs to be approved by at least two-thirds of the current consensus nodes to become a part of the cross chain system and be registered on the relay chain. The parameters are as follows:


```go
type ChainidParam struct {
	Chainid uint64          
	Address common.Address  
}
```

| Parameter | Description                                                             |
| :-------: | ----------------------------------------------------------------------- |
|  Chainid  | Chain ID of this chain in the cross chain system                        |
|  Address  | Address of the consensus node (address corresponding to the public key) |

### Update Side Chain Information

The registered account needs to send an update application to the relay chain when updating side chain information. An application needs to be approved by at least two-thirds of the current consensus nodes in order to successfully carry out the update action.

The `updateSideChain` method is used to send an application to update the registered information of a particular blockchain. The parameters are as follows:


```go
type RegisterSideChainParam struct {
	Address      string    
	ChainId      uint64    
	Router       uint64    
	Name         string    
	BlocksToWait uint64    
	CCMCAddress  []byte    
}
```

|  Parameter   | Description                                                                                                                                                                                     |
| :----------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|   Address    | The Ontology wallet address used to register the chain, address has the privileges to modify registered chain information                                                                       |
|   ChainId    | Chain ID of this chain in the cross chain system                                                                                                                                                |
|    Router    | The routing protocol of the chain, existing routing protocols - **1:BTC**, **2:ETH**, **3:ONT**, isomorphic chains can select existing protocol, rest must first define the respective protocol |
|     Name     | Name of the blockchain network                                                                                                                                                                  |
| BlocksToWait | No. of blocks to wait for in order to confirm finality                                                                                                                                          |
| CCMCAddress  | Cross chain management contract address                                                                                                                                                         |

The `approveUpdateSideChain` method is used by the consensus nodes to vote for an update application.  An application needs to be approved by at least two-thirds of the current consensus nodes in order to successfully carry out the update action. The parameters are as follows:

```go
type ChainidParam struct {
	Chainid uint64          
	Address common.Address  
}
```
| Parameter | Description                                                             |
| :-------: | ----------------------------------------------------------------------- |
|  Chainid  | Chain ID of this chain in the cross chain system                        |
|  Address  | Address of the consensus node (address corresponding to the public key) |


### Remove Side Chain

When a side chain needs to withdraw from the cross chain ecosystem an application needs to be submitted to the relay chain. The application needs to be approved by at least two-thirds of the current consensus nodes in order to successfully carry out the removal. However, before a side chain is removed, the relay chain first verifies whether the cross chain assets from the chain in question have been returned to the chain.

The `quitSideChain` method can be used to submit an application to the relay chain for the removal of a side chain. The parameters are as follows:

```go
type ChainidParam struct {
	Chainid uint64          
	Address common.Address  
}
```

| Parameter | Description                                        |
| :-------: | -------------------------------------------------- |
|  Chainid  | Chain ID of this chain in the cross chain system   |
|  Address  | Ontology wallet address used to register the chain |


The `approveQuitSideChain` method can be used by the consensus nodes to vote and sign for a removal application. The application needs to be approved by at least two-thirds of the current consensus nodes in order to successfully carry out the removal. The parameters are as follows:

```go
type ChainidParam struct {
	Chainid uint64          
	Address common.Address  
}
```

| Parameter | Description                                                             |
| :-------: | ----------------------------------------------------------------------- |
|  Chainid  | Chain ID of this chain in the cross chain system                        |
|  Address  | Address of the consensus node (address corresponding to the public key) |

### Linking BTC Multi-Chain Address

Since smart contracts cannot be used on the Bitcoin network, a multi-signature address mechanism is used to lock the cross chain assets. Once a user ensures that a cross chain vendor is reliable, they transfer their assets to the vendor's multi-chain address. Currently, any institution can become a vendor in the cross chain ecosystem and register a multi-signature address. It is up to the user to choose which vendor's cross chain service they want to use.

Each multi-signature address on the Bitcoin network is exclusively linked to one complement token address on the target chain. Hence, every different multi-signature address is linked to a different contract, and a cross chain operation using different multi-signature addresses will transfer the tokens to the target chain in a different form of asset on the target chain. Linking the multi-signature address to a complement token asset contract on the target chain requires signature verification of the multi-signature address.

The `registerRedeem` method can be used to register a multi-chain address and link it to an asset contract on a particular target chain. The parameters are as follows:

```go
type RegisterRedeemParam struct {
	RedeemChainID   uint64          
	ContractChainID uint64			
	Redeem          []byte			
	CVersion        uint64			
	ContractAddress []byte			
	Signs           [][]byte		
}
```

|    Parameter    | Description                                               |
| :-------------: | --------------------------------------------------------- |
|  RedeemChainID  | Chain ID of the host chain of the multi-signature address |
| ContractChainID | Chain ID of the host chain of the asset contract          |
|     Redeem      | Redeem script of the multi-signature address              |
|    CVersion     | Version number                                            |
| ContractAddress | Contract address of the asset contract                    |
|      Signs      | Signatures for the multi-signature address                |


### Setting the BTC Transaction Parameters 

Since assets are locked in a multi-signature contract on the Bitcoin network and need to be released when assets are transferred back to the Bitcoin network, there are certain parameters such as the fee charged for releasing assets, minimum amount of change, etc. that need to be set for the multi-signature address.

The `setBtcTxParam` method can be used to set the necessary parameters for a multi-chain address. The parameters are as follows:

```go
type BtcTxParam struct {
	Redeem        []byte				
	RedeemChainId uint64				
	Signs          [][]byte				
	Detail        *BtcTxParamDetial		
}
type BtcTxParamDetial struct {
	PVersion  uint64					
	FeeRate   uint64					
	MinChange uint64					
}
```

|   Parameter   | Description                                               |
| :-----------: | --------------------------------------------------------- |
|    Redeem     | Redeem script of the multi-signature address              |
| RedeemChainId | Chain ID of the host chain of the multi-signature address |
|     Signs     | Signatures for the multi-signature address                |
|    Detail     | Configuration details for the parameters                  |
|   PVersion    | Version no.                                               |
|    FeeRate    | Transaction fee rate                                      |
|   MinChange   | Min. amount of change                                     |



