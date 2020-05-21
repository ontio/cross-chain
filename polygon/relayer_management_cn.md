# Polygon的权限系统

中继链没有手续费设置，但是与中继链交互需要有相应的权限，中继链的权限是基于账户的，每个账户使用独立且唯一的公私钥对，发起交易时使用其私钥进行签名，接收方可通过公钥验签知道交易具体是由哪个账户发出，实现交易的可控及后续监管的追溯。能够与中继链交互的账户称为relayer。

## Relayer管理合约

### 申请注册relayer

registerRelayer

参数：

```go
type RelayerListParam struct {
	AddressList []common.Address
	Address     common.Address
}
```

`AddressList`为申请注册成为relayer的地址列表， `Address`为调用者地址。

合约会为每个申请生成一个申请ID，并将ID notify给用户。

### 同意注册Relayer请求

approveRegisterRelayer

参数：

```go
type ApproveRelayerParam struct {
	ID      uint64
	Address common.Address
}
```

`ID`为申请ID，`Address`为发交易的共识节点的地址，超过三分之二的共识节点都同意注册请求注册才会生效。

### 申请移除relayer

removeRelayer

```go
type RelayerListParam struct {
	AddressList []common.Address
	Address     common.Address
}
```

`AddressList`为申请注册成为relayer的地址列表， `Address`为调用者地址。

合约会为每个申请生成一个申请ID，并将ID notify给用户。

### 同意移除Relayer请求

approveRemoveRelayer

参数：

```go
type ApproveRelayerParam struct {
	ID      uint64
	Address common.Address
}
```

`ID`为申请ID，`Address`为发交易的共识节点的地址，超过三分之二的共识节点都同意移除请求移除才会生效。

## Relayer权限校验

中继链在交易进入交易池之前会做relayer的权限校验，默认接受当前共识节点发送的交易，其他地址发送的交易需要有relayer权限才能够进入交易池。