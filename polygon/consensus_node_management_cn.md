# Polygon的治理机制

## 节点治理

### 新节点加入

新节点想参与中继链的网络共识需要现有共识节点的批准，新节点先在中继链上提出申请，现有共识节点对该申请进行投票签名，超过三分之二的共识节点同意，则新节点加入网络，先成为候选节点。

方法名：registerCandidate

用于节点提出申请成为中继链的共识节点。

参数：

```go
type RegisterPeerParam struct {
	PeerPubkey string              //节点公钥
	Address    common.Address      //发起请求的账户地址
}
```

方法名：unRegisterCandidate

用于节点取消成为中继链的共识节点的申请。

参数：

```go
type RegisterPeerParam struct {
	PeerPubkey string              //节点公钥
	Address    common.Address      //发起请求的账户地址，需要与提出申请时的账户地址一致
}
```

方法名：approveCandidate

用于当前共识节点投票签名，超过三分之二的当前共识节点同意则新节点加入中继链共识网络。

参数：

```go
type PeerParam struct {
	PeerPubkey string              //节点公钥
	Address    common.Address      //共识节点的节点地址（节点公钥对应的地址）
}
```

### 共识切换

中继链采用vbft共识算法，每隔6w（可配置）个区块，进行一次共识节点切换，目前中继链的策略是所有的节点（包括共识节点和新申请的候选节点）都在周期切换后成为共识节点。因此新节点加入后将在下个周期成为共识节点。

### 节点退出

节点可以选择退出中继链网络，分两种情况：1.若是在候选节点状态下退出，则可以立即关机退出，下个周期不会被选为共识节点。2.若是在共识节点状态下退出，因为共识周期内共识节点保持不变，因此该节点不能立即关机退出，必须等到共识周期切换后，新共识节点选出，该节点才可以关机退出。否则将按照共识节点宕机处理。

方法名：quitNode

用于节点退出中继链共识网络。

参数：

```go
type PeerParam struct {
	PeerPubkey string              //节点公钥
	Address    common.Address      //发起请求的账户地址，需要与提出申请时的账户地址一致
}
```

### 节点作恶

若在网络运行过程中发现节点作恶，则当前共识节点可以发起投票，将某个作恶节点拉黑并踢出网络，这里又分为两种情况：1.若作恶节点是候选节点，则直接踢出，下次共识周期切换后其不会被选择成为共识节点。2.若作恶节点是共识节点，则踢出后自动切换共识周期，选出新的共识节点。

方法名：blackNode

用于节点退出中继链共识网络。

参数：

```go
type PeerParam struct {
	PeerPubkey string              //节点公钥
	Address    common.Address      //共识节点的节点地址（节点公钥对应的地址）
}
```
