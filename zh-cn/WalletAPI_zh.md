# NEO3 SDK - 钱包相关接口

```WalletAPI```提供了对钱包概念相关接口的封装，包括余额查询，GAS Claim，转账等功能。

## 添加引用
在需要使用该模块的项目文件的头部添加如下代码：

```c#
using Neo.Network.RPC;
```

## 初始化
使用钱包相关接口需要依赖`RpcClient`：

```c#
// choose a neo node with rpc opened
RpcClient client = new RpcClient("http://seed1t.neo.org:20332");
WalletAPI walletAPI = new WalletAPI(client);
```

## 查询余额

NEP5资产余额查询示例：

```c#
// get the neo balance of account
string tokenHash = "0x43cf98eddbe047e198a3e5d57006311442a0ca15";
string address = "AJoQgnkK1i7YSAvFbPiPhwtgdccbaQ7rgq";
walletAPI.GetTokenBalance(tokenHash, address);
```

在NEO3中NEO和GAS都是NEP5资产，且脚本哈希固定，所以这里提供了更简单的接口：

```c#
// get the neo balance
uint neoBalance = walletAPI.GetNeoBalance(address);

// get the neo balance
decimal gasBalance = walletAPI.GetGasBalance(address);
```

## Claim GAS

在Claim GAS之前可以查询当前账户可以Claim的GAS数量：

```c#
// get the claimable GAS of one address
string address = "AJoQgnkK1i7YSAvFbPiPhwtgdccbaQ7rgq";
decimal gasAmount = walletAPI.GetUnclaimedGas(address);
```

在NEO3中Claim GAS的过程是在NEO转账时自动进行的，所以Claim GAS的操作需要构建一笔账户给自己转账的交易：

```c#
// claiming gas needs the KeyPair of account, you can also use wif or private key hex string
string wif = "L1rFMTamZj85ENnqNLwmhXKAprHuqr1MxMHmCWCGiXGsAdQ2dnhb";
Transaction transaction = walletAPI.ClaimGas(wif);
```

## 资产转账

`WalletAPI`中封装了NEP5转账方法:

```c#
string tokenHash = "0x43cf98eddbe047e198a3e5d57006311442a0ca15";
string wif = "L1rFMTamZj85ENnqNLwmhXKAprHuqr1MxMHmCWCGiXGsAdQ2dnhb";
string address = "AJoQgnkK1i7YSAvFbPiPhwtgdccbaQ7rgq";

// transfer 10 neo from wif to address
walletAPI.Transfer(tokenHash, wif, address, 10);

// print a message after the transaction is on chain
WalletAPI neoAPI = new WalletAPI(client);
neoAPI.WaitTransaction(transaction)
    .ContinueWith(async (p) => Console.WriteLine($"Transaction is on block {(await p).BlockHash}"));
```