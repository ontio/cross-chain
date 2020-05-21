<h1 align=center>Polygon Chain Governance Mechanism </h1>

## Node Governance

### Node Joins the Network

A new node that wants to join the consensus mechanism of the relay chain must first be approved by the existing consensus nodes. The node first submits an application to the relay chain and the existing consensus nodes verify the application and vote for it. An application that is verified by at least two-thirds of the nodes is approved and the respective node joins the consensus network, and becomes a candidate node.

The `registerCandidate` method is used to submit an application for allowing a node to join the consensus network of the relay chain. The parameters are as follows:


```go
type RegisterPeerParam struct {
	PeerPubkey string              
	Address    common.Address      
}
```

| Parameter  | Description                            |
| :--------: | -------------------------------------- |
| PeerPubkey | Node public key                        |
|  Address   | Wallet address used to submit requests |


The `unRegisterCandidate` method is used to submit an application to remove a node from consensus network of the relay chain. The parameters are as follows:

```go
type RegisterPeerParam struct {
	PeerPubkey string              
	Address    common.Address      
}
```

| Parameter  | Description                                                                                         |
| :--------: | --------------------------------------------------------------------------------------------------- |
| PeerPubkey | Node public key                                                                                     |
|  Address   | Wallet address used to submit requests, must be the same as the address that submit the application |

The `approveCandidate` method is used to by consensus nodes to vote for consensus node applications. An application that receives at least two-thirds of the votes from existing consensus nodes is approved to join the network. The parameters are as follows:

```go
type PeerParam struct {
	PeerPubkey string              
	Address    common.Address      
}
```

| Parameter  | Description                                            |
| :--------: | ------------------------------------------------------ |
| PeerPubkey | Node public key                                        |
|  Address   | Node address (address corresponding to the public key) |

### Consensus Switching

The relay chain employs the VBFT consensus algorithm. A consensus network switch takes place every 60k blocks generated. Considering the current policy employed by the relay chain, newly added candidate nodes become a part of the network when the consensus cycle is switched.

### Node Withdrawal

A node may withdraw from the relay chain network. There are two possible scenarios-
1. If the node in question withdraws when it is a part of the candidate node pool, it can leave the network immediately. In the next consensus cycle this node will not be selected as the candidate node.
2. If the node withdraws when it is in the consensus node status in the current cycle, it cannot withdraw immediately. Once the current consensus node cycle completes, a new node takes it place and this node is removed from the network.

The `quitNode` method can be used by a consensus node to withdraw from the relay chain's consensus network. The parameters are as follows:

```go
type PeerParam struct {
	PeerPubkey string              
	Address    common.Address      
}
```

| Parameter  | Description                                                                                         |
| :--------: | --------------------------------------------------------------------------------------------------- |
| PeerPubkey | Node public key                                                                                     |
|  Address   | Wallet address used to submit requests, must be the same as the address that submit the application |

### Malicious Nodes

If a node in the relay chain network is found to be malicious in nature, the current consensus nodes can vote on removing the malicious node from the consensus network. There are two possible scenarios that may occur-
1. If the malicious node is in the candidate node pool, it will be kicked out immediately and will not be selected as a consensus node in the following cycle.
2. If the malicious node is in the consensus node status, the current consensus will be stopped, the node will kicked out and replaced with a new consensus node to carry out a new consensus cycle.

The `blackNode` method can be used to remove a node from the relay chain network. The parameters are as follows:

```go
type PeerParam struct {
	PeerPubkey string              
	Address    common.Address      
}
```
| Parameter  | Description                                            |
| :--------: | ------------------------------------------------------ |
| PeerPubkey | Node public key                                        |
|  Address   | Node address (address corresponding to the public key) |

