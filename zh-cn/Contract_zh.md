# NEO SDK - NEP5接口调用

注：本文档中使用的 NEO 版本为 3.0 及以上。

通过该模块可以使用特定的参数和方法构造NEO3中的交易，完成个性化的功能，该部分操作需要对NEO3交易体的结构有一定了解，具体可参考[交易](https://web3j.io)。

## 添加引用

在需要使用该模块的项目文件的头部添加如下代码：

```c#
using Neo.Network.RPC;
```

## 初始化
发送RPC请求是通过模块中的`TransactionManager`类来实现的，初始化该类有两个参数，第一个是RpcClient对象，第二个是交易发起者的账户Hash (UInt160)：

```c#
RpcClient client = new RpcClient("http://seed1.neo.org:10331");
UInt160 sender = "AdmyedL3jdw2TLvBzoUD2yU443NeKrP5t5".ToUInt160();
TransactionManager txManager = new TransactionManager(client, sender);
```


## 构造一笔NEP5转账交易

区块索引 = 区块高度 = 区块数量 - 1
Index = Height = Count - 1

```c#
uint blockHeight = client.GetBlockCount() - 1;
```

## 构造一笔多签转账交易

获取主链中高度最大的区块的散列：

```c#
string hexString = client.GetBestBlockHash();
byte[] hashBytes = hexString.HexToBytes();
UInt256 hash256 = UInt256.Parse(hexString);
```

也可以根据区块索引获取其他区块的散列：

```c#
string hexString = client.GetBlockHash(10000);
byte[] hashBytes = hexString.HexToBytes();
UInt256 hash256 = UInt256.Parse(hexString);
```
