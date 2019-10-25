# NEO3 SDK - 合约部署与调用

我们提供了```ContractClient```类来实现合约部署和只读调用的接口，此外对于符合NEP5规范的合约我们还提供了```Nep5API```类来实现相关的方法。

## 合约部署

合约部署之前需要对合约进行编译以获取到合约脚本与manifest，合约部署时同样需要发送账户支付系统费和网络费，下面的示例通过```DeployContract```方法构造交易，交易广播上链成功后才可以调用合约中的方法：

```c#
using Neo.Network.P2P.Payloads;
using Neo.Network.RPC;
using Neo.SmartContract;
using Neo.SmartContract.Manifest;
using Neo.Wallets;
using System;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            // choose a neo node with rpc opened
            RpcClient client = new RpcClient("http://seed1t.neo.org:20332");
            ContractClient contractClient = new ContractClient(client);

            // contract script, it should be from compiled file, we use empty byte[] in this example
            byte[] script = new byte[1];

            // we use default ContractManifest in this example
            ContractManifest manifest = ContractManifest.CreateDefault(script.ToScriptHash());

            // deploy contract needs sender to pay the system fee
            KeyPair senderKey = "L1rFMTamZj85ENnqNLwmhXKAprHuqr1MxMHmCWCGiXGsAdQ2dnhb".ToKeyPair();

            // create the deploy transaction
            Transaction transaction = contractClient.DeployContract(script, manifest, senderKey);

            // Broadcasts the transaction over the NEO network
            client.SendRawTransaction(transaction);
            Console.WriteLine($"Transaction {transaction.Hash.ToString()} is broadcasted!");

            // print a message after the transaction is on chain
            WalletAPI neoAPI = new WalletAPI(client);
            neoAPI.WaitTransaction(transaction)
               .ContinueWith(async (p) => Console.WriteLine($"Transaction is on block height {await p}"));

            Console.ReadKey();
        }
    }
}
```

## 合约模拟调用

```ContractClient```提供了```TestInvoke```方法来对合约进行模拟调用，执行后不会影响链上数据。可以直接调用读取信息的合约方法，比如下面的例子调用了NEO原生合约中的name方法

```c#
// choose a neo node with rpc opened
RpcClient client = new RpcClient("http://seed1t.neo.org:20332");
ContractClient contractClient = new ContractClient(client);

// get the contract hash
UInt160 scriptHash = NativeContract.NEO.Hash;

// test invoking the method provided by the contract 
string name = contractClient.TestInvoke(scriptHash, "name")
    .Stack.Single().ToStackItem().GetString();
```

## 合约调用（上链交易）

上链的合约调用要经过构建交易脚本，构建交易和广播上链等过程，可参见[交易构造模块文档](Transaction_zh)：

```c#
using Neo;
using Neo.Network.P2P.Payloads;
using Neo.Network.RPC;
using Neo.SmartContract.Native;
using Neo.VM;
using Neo.Wallets;
using System;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            // choose a neo node with rpc opened
            RpcClient client = new RpcClient("http://seed1t.neo.org:20332");

            // get the KeyPair of your account, this account will pay the system and network fee
            KeyPair sendKey = "L1rFMTamZj85ENnqNLwmhXKAprHuqr1MxMHmCWCGiXGsAdQ2dnhb".ToKeyPair();
            UInt160 sender = sendKey.ToScriptHash();

            // add Cosigners, this is a collection of scripthashs which need to be signed
            Cosigner[] cosigners = new[] { new Cosigner { Scopes = WitnessScope.CalledByEntry, Account = sender } };

            // get the scripthash of the account you want to transfer to
            UInt160 receiver = "AKviBGFhWeS8xrAH3hqDQufZXE9QM5pCeP".ToUInt160();

            // construct the script, in this example, we will transfer 1 NEO to receiver
            UInt160 scriptHash = NativeContract.NEO.Hash;
            byte[] script = scriptHash.MakeScript("transfer", sender, receiver, 1);

            // initialize the TransactionManager with rpc client and sender scripthash
            Transaction tx = new TransactionManager(client, sender)
                // fill the script, attribute, cosigner and network fee
                .MakeTransaction(script, null, cosigners, 0)
                // add signature for the transaction with sendKey
                .AddSignature(sendKey)
                // sign transaction with the added signature
                .Sign()
                .Tx;

            // broadcasts transaction over the NEO network.
            client.SendRawTransaction(tx);
            Console.WriteLine($"Transaction {tx.Hash.ToString()} is broadcasted!");

            // print a message after the transaction is on chain
            WalletAPI neoAPI = new WalletAPI(client);
            neoAPI.WaitTransaction(tx)
               .ContinueWith(async (p) => Console.WriteLine($"Transaction is on block height {await p}"));

            Console.ReadKey();
        }
    }
}   
```

## NEP5合约

```Nep5API```封装了转账交易的生成方法，以上过程中的交易构造过程可简化为：

```c#
byte[] script = scriptHash.MakeScript("transfer", sender, receiver, 1);
Transaction tx = new TransactionManager(client, sender)
    .MakeTransaction(script, null, cosigners, 0)
    .AddSignature(sendKey)
    .Sign()
    .Tx;

// equals to
Nep5API nep5API = new Nep5API(client);
Transaction tx = nep5API.CreateTransferTx(scriptHash, sendKey, receiver, 1);
```

此外```Nep5API```还提供了简单的读取方法：

```c#
// get nep5 name
string name = nep5API.Name(scriptHash);

// get nep5 symbol
string symbol = nep5API.Symbol(scriptHash);

// get nep5 token decimals
uint decimals = nep5API.Decimals(scriptHash);

// get nep5 token total supply
BigInteger totalSupply = nep5API.TotalSupply(scriptHash);
```
