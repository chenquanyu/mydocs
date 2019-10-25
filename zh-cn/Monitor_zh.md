# NEO3 SDK - 获取区块链数据和状态信息

注：本文档中使用的 NEO 版本为 3.0 及以上。

通过RpcClient模块可以获取区块，交易和合约的信息，通过特定合约方法的调用可以获取Policy和NEP5 Token的信息。

在需要使用该模块的项目文件的头部添加如下代码：

```c#
using Neo.Network.RPC;
```

## 通过RPC接口直接获取区块数据

获取最新区块高度或哈希

```c#
// choose a neo node with rpc opened
RpcClient client = new RpcClient("http://seed1t.neo.org:20332");

// get the highest block hash
string hash = client.GetBestBlockHash();

// get the highest block height
uint height = client.GetBlockCount() - 1;
```

获取某个区块内具体数据，包括交易列表等

```c#
// get block data
RpcBlock block = client.GetBlock("166396");

// get block data with block hash
RpcBlock block = client.GetBlock("0x953f6efa29c740b68c87e0a060942056382a6912a0ddeddc2f6641acb92d9700");
```

获取某个交易内具体数据

```c#
// get transaction
RpcTransaction transaction = client.GetRawTransaction("0x48ec3d235c6b386eee324a77a10b0f9e8e37d3c1ebb99626f3d1dd70db26d788");
```

更多信息请参见[RPC模块文档](RPC_zh.md)。

## 获取Policy相关信息

在NEO3中通过调用原生合约PolicyContract中的方法获取Policy相关信息：

```c#
// choose a neo node with rpc opened
PolicyAPI policyAPI = new PolicyAPI(new RpcClient("http://seed1t.neo.org:20332"));

// get the accounts blocked by policy
UInt160[] blockedAccounts = policyAPI.GetBlockedAccounts(); // [], no account is blocked by now

// get the system fee per byte
long feePerByte = policyAPI.GetFeePerByte(); // 1000, 0.00001000 GAS per byte

// get the max size of one block
uint maxBlockSize = policyAPI.GetMaxBlockSize(); // 262144, (1024 * 256) bytes one block

// get the max transaction count per block
uint maxTransactionsPerBlock = policyAPI.GetMaxTransactionsPerBlock(); // 512, max 512 transactions one block
```


## 获取合约信息

通过RpcClient模块可以获取合约脚本，哈希与manifest的所有信息：

```c#
// choose a neo node with rpc opened
RpcClient client = new RpcClient("http://seed1t.neo.org:20332");

// get neo contract state
ContractState contractState = client.GetContractState(NativeContract.NEO.Hash.ToString());
```

对于NEP5合约可以通过Nep5API获取名称，标记，小数位和总量等信息：

```c#
// get nep5 token info
Nep5API nep5API = new Nep5API(client);
RpcNep5TokenInfo tokenInfo = nep5API.GetTokenInfo(NativeContract.NEO.Hash);
```
