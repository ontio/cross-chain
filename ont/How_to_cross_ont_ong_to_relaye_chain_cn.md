## 如何将Ont、Ong资产跨到中继链上

中继链采用ong作为手续费，同时采用ont作为质押代币。因此需要将ontology网络中的ont和ong资产跨到中继链上去，目前中继链只支持ont和ong资产。

ont、ong在ontology网络上是以代理合约的方式跨到中继链的，代理合约是native合约，合约地址为：

```go
OntLockContractAddress, _    = common.AddressParseFromBytes([]byte{0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x0a})
```

这本代理合约实现了跨链的lock和unlock方法，因此用户可以通过调用lock方法来实现ont、ong资产跨到中继链。ontology go sdk目前支持lock操作：

```go
func (this *OntLock) Lock(gasPrice, gasLimit uint64, payer *Account, sourceAssetHash common.Address, from *Account, toChainID uint64, toAddress []byte, amount uint64) (common.Uint256, error) {}
```

参数为：

`payer`: 付交易手续费的账户，不填为默认为from账户。

`sourceAssetHash`: ont或者ong的资产合约地址。

`from`: 跨链交易的出金地址。

`toChainID`: 目标链的链ID，0表示跨链到中继链。

`toAddress`: 目标链上的入金地址，注意ontology和中继链采用不同的加密方式，因此地址不能复用。

`amount`: 资产跨链的数量。

## 如何将中继链上的Ont、Ong跨回

ont、ong在中继链网络上也是以代理合约的方式跨到中继链的，代理合约是native合约，合约地址为：

```go
OntLockProxyContractAddress, _      = common.AddressParseFromBytes([]byte{0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x07})
```

这本代理合约实现了跨链的lock和unlock方法，因此用户可以通过调用lock方法来实现ont、ong资产跨到中继链。中继链go sdk目前支持lock操作：

```go
func (this *OntLock) Lock(payer *Account, sourceAssetHash common.Address, from *Account, toChainID uint64, toAddress []byte, amount uint64) (common.Uint256, error) {}
```

参数为：

`payer`: 付交易手续费的账户，不填为默认为from账户。

`sourceAssetHash`: ont或者ong的资产合约地址。

`from`: 跨链交易的出金地址。

`toChainID`: 目标链的链ID，3表示跨链到ontology网络。

`toAddress`: 目标链上的入金地址，注意ontology和中继链采用不同的加密方式，因此地址不能复用。

`amount`: 资产跨链的数量。