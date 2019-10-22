# NEO3 SDK - 获取区块链数据和状态信息

注：本文档中使用的 NEO 版本为 3.0 及以上。

通过该模块可以使用特定的参数和方法构造NEO3中的交易，完成个性化的功能，该部分操作需要对NEO3交易体的结构有一定了解，具体可参考[交易](https://web3j.io)。

## 获取当前区块高度

在需要使用该模块的项目文件的头部添加如下代码：

```c#
using Neo.Network.RPC;
```

## 获取交易体

在需要使用该模块的项目文件的头部添加如下代码：

```c#
using Neo.Network.RPC;
```

## 获取区块数据

在需要使用该模块的项目文件的头部添加如下代码：

```c#
using Neo.Network.RPC;
```

## 获取Policy相关信息

发送RPC请求是通过模块中的`TransactionManager`类来实现的，初始化该类有两个参数，第一个是RpcClient对象，第二个是交易发起者的账户Hash (UInt160)：

```c#
RpcClient client = new RpcClient("http://seed1.neo.org:10331");
UInt160 sender = "AdmyedL3jdw2TLvBzoUD2yU443NeKrP5t5".ToUInt160();
TransactionManager txManager = new TransactionManager(client, sender);
```


## 获取合约信息

区块索引 = 区块高度 = 区块数量 - 1
Index = Height = Count - 1

```c#
uint blockHeight = client.GetBlockCount() - 1;
```
