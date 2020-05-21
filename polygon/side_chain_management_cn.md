# 中继链的治理机制

## 侧链治理

### 新侧链加入

新侧链加入跨链也需要现有共识节点的批准，新侧链先在中继链上提出申请，现有共识节点对该申请进行投票签名，超过三分之二的共识节点同意，则新侧链加入网络。

方法名： "registerSideChain"

用于新区块链申请接入中继链跨链

参数：

```go
type RegisterSideChainParam struct {
	Address      string    //注册该链所用的本体钱包地址，只有该地址有修改注册信息的权限
	ChainId      uint64    //该链的链ID
	Router       uint64    //该链的路由协议，目前已实现的协议有1:btc, 2:eth, 3:ont，同构链可以采用已有协议，异构链则需要提交信息新增路由协议
	Name         string    //链的名称
	BlocksToWait uint64    //终局所需要的确认区块数
	CCMCAddress  []byte    //该链的跨链管理合约地址
}
```

方法名： "approveRegisterSideChain"

用于当前共识节点对新加入的区块链进行投票签名，超过三分之二的当前共识节点同意则该区跨链加入到中继链跨链中去。

参数：

```go
type ChainidParam struct {
	Chainid uint64          //新区块链的链ID
	Address common.Address  //共识节点的节点地址（节点公钥对应的地址）
}
```

### 侧链信息的升级

侧链的注册信息需要升级时，需要注册侧链的账户提出升级申请，现有共识节点对该申请进行投票签名，超过三分之二的共识节点同意，则更新侧链的注册信息。

方法名： "updateSideChain"

用于某条区块链更新其基本信息。

参数：

```go
type RegisterSideChainParam struct {
	Address      string    //注册该链所用的本体钱包地址，只有该地址有修改注册信息的权限
	ChainId      uint64    //该链的链ID
	Router       uint64    //该链的路由协议，目前已实现的协议有1:btc, 2:eth, 3:ont，同构链可以采用已有协议，异构链则需要提交信息新增路由协议
	Name         string    //链的名称
	BlocksToWait uint64    //终局所需要的确认区块数
	CCMCAddress  []byte    //该链的跨链管理合约地址
}
```

方法名： "approveUpdateSideChain"

用于当前共识节点对新加入的区块链的更新请求进行投票签名，超过三分之二的当前共识节点同意其更新。

参数：

```go
type ChainidParam struct {
	Chainid uint64          //新区块链的链ID
	Address common.Address  //共识节点的节点地址（节点公钥对应的地址）
}
```

### 侧链的退出

侧链需要退出时，需要注册侧链的账户提出退出申请，现有共识节点对该申请进行投票签名，超过三分之二的共识节点同意，则侧链退出跨链。需要注意的是，共识节点在同意侧链退出之前必须验证侧链上的跨链资产已经全部转回源链。

方法名： "quitSideChain"

用于某条区块链申请退出中继链跨链。

参数：

```go
type ChainidParam struct {
	Chainid uint64          //区块链的链ID
	Address common.Address  //注册该链所用的本体钱包地址，只有该地址有修改注册信息的权限
}
```

方法名： "approveQuitSideChain"

用于当前共识节点对区块链的退出请求进行投票签名，超过三分之二的当前共识节点同意其退出。

参数：

```go
type ChainidParam struct {
	Chainid uint64          //新区块链的链ID
	Address common.Address  //共识节点的节点地址（节点公钥对应的地址）
}
```

### BTC信托地址的绑定

由于btc不支持智能合约，因此不同于其他链锁定资产，释放资产的资产转移方式，btc采用信托多签地址来进行资产的锁定和释放，用户在使用信托地址前，需要信任该地址。目前跨链协议支持任何一个机构成为信托地址，用户可以选择信任其中任何一个信托地址。

信托地址和其在其他链上发行的btcx资产地址是绑定的，一个信托地址对应于每条链上单一的btcx资产地址。因此用户选择不同的信托地址进行锁定会得到其他链上不同的btcx资产。信托地址和btcx资产地址的绑定关系需要验证信托地址的签名。

方法名： "registerRedeem"

用于注册信托地址并绑定其在每条链上的资产合约地址。

参数：

```go
type RegisterRedeemParam struct {
	RedeemChainID   uint64           //信托地址所在链的链ID
	ContractChainID uint64			//资产合约所在链的链ID
	Redeem          []byte			//信托地址的Redeem脚本
	CVersion        uint64			//版本信息
	ContractAddress []byte			//资产合约的合约地址
	Signs           [][]byte		//信托地址的多签
}
```

### BTC交易参数设置

由于btc采用信托地址的形式进行资产锁定和释放，释放时的手续费率，最小找零等参数需要有信托地址进行设置。

方法名： "setBtcTxParam"

用于信托地址修改其交易参数配置。

参数：

```go
type BtcTxParam struct {
	Redeem        []byte				//信托地址的Redeem脚本
	RedeemChainId uint64				//信托地址所在链的链ID
	Sigs          [][]byte				//信托地址的多签
	Detial        *BtcTxParamDetial		 //参数配置信息
}
type BtcTxParamDetial struct {
	PVersion  uint64					//版本号
	FeeRate   uint64					//交易费率
	MinChange uint64					//最小找零
}
```
