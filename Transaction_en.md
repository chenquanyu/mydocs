# NEO SDK - RPC module

注：本文档中使用的 NEO 版本为 3.0 及以上。
PS: 

The standard methods provided in this module can be used to send RPC requests in code by passing required parameters, instead of using third party tools (like Postman, Fiddler, etc.) to construct a JSON request body.

## Add references
Add the following code at the beginning of the project file where this module is needed:

```c#
using Neo.Network.RPC;
```

## Initialization
Sending RPC requests is realized by using the `RpcClient` class in the module. There are two ways to initialize this class and both need the RPC port of a NEO node. The testnet seed node is used here:

```c#
RpcClient client = new RpcClient("http://seed1.neo.org:10331");
```

Or

```c#
RpcClient client2 = new RpcClient(new HttpClient()
{
    BaseAddress = new Uri("http://seed1.neo.org:10331")
});
```

## Get current block height
Index = Height = Count - 1

```c#
uint blockHeight = client.GetBlockCount() - 1;
```

## Get block hash
Get the hash of the tallest block in the main chain:

```c#
string hexString = client.GetBestBlockHash();
byte[] hashBytes = hexString.HexToBytes();
UInt256 hash256 = UInt256.Parse(hexString);
```

Or get the hash of a specific block by its index:

```c#
string hexString = client.GetBlockHash(10000);
byte[] hashBytes = hexString.HexToBytes();
UInt256 hash256 = UInt256.Parse(hexString);
```

## Get block information
Get the details of a specific block by its hash or index:

```c#
Block block = client.GetBlock("773dd2dae4a9c9275290f89b56e67d7363ea4826dfd4fc13cc01cf73a44b0d0e").Block;
```

Or

```c#
Block block = client.GetBlock("10000").Block;
```

Or get the serialized information of a specific block by its hash or index:

```c#
string serializedBlock = client.GetBlockHex("773dd2dae4a9c9275290f89b56e67d7363ea4826dfd4fc13cc01cf73a44b0d0e");
```

## Get block header information
Get the details of a specific block header by its hash or index:

```c#
Header header = client.GetBlockHeader("a5508c9b6ed0fc09a531a62bc0b3efcb6b8a9250abaf72ab8e9591294c1f6957").Header;
```

Or

```c#
Header header = client.GetBlockHeader("10000").Header;
```

Or get the serialized information of a specific block header by its block hash or index:

```c#
string serializedBlockHeader = client.GetBlockHeaderHex("a5508c9b6ed0fc09a531a62bc0b3efcb6b8a9250abaf72ab8e9591294c1f6957");
```

## Get system fee
Get the system fee of all the blocks until the specific block index:

```c#
BigInteger sysFee = BigInteger.Parse(client.GetBlockSysFee(10000));
```

## Get node count
Get the current number of nodes connected to the node which RPC requests are sent to:

```c#
int connectionCount = client.GetConnectionCount();
```

## Get contract information
Get the detailed information of a contract by its script hash:

```c#
ContractState contractState = client.GetContractState("dc675afc61a7c0f7b3d2682bf6e1d8ed865a0e5f");
```

## Get connected/unconnected nodes
Get a list of nodes connected to/disconnected from the node which RPC requests are sent to:

```c#
RpcPeers rpcPeers = client.GetPeers();
RpcPeer[] connected = rpcPeers.Connected;
RpcPeer[] unconnected = rpcPeers.Unconnected;
if (connected.Length > 0)
{
    RpcPeer peer = connected[1];
    string address = peer.Address;
    int port = peer.Port;
}
```

## Get transactions in memory
Get a list of confirmed transaction hashes in memory:

```c#
string[] verifiedTransactions = client.GetRawMempool();
```

Or get both confirmed and unconfirmed transaction hashes in memory:

```c#
RpcRawMemPool memPool = client.GetRawMempoolBoth();
string[] verifiedTransactions = memPool.Verified;
string[] unverifiedTransactions = memPool.UnVerified;
```

## 获取交易信息Get transaction information
Get the details of a specific transaction by its ID:

```c#
RpcTransaction rpcTransaction = client.GetRawTransaction("f4250dab094c38d8265acc15c366dc508d2e14bf5699e12d9df26577ed74d657");
Transaction transaction = rpcTransaction.Transaction;
```

Or get the serialized information of a specific transaction by its ID:

```c#
string serializedTransaction = client.GetRawTransactionHex("f4250dab094c38d8265acc15c366dc508d2e14bf5699e12d9df26577ed74d657");
```

## Get a value from contract storage
Get the stored value by the corresponding contract script hash and the storage key:

```c#
string value = client.GetStorage("03febccf81ac85e3d795bc5cbd4e84e907812aa3", "5065746572");
```

## Get transaction height
Get the height of the block where the specific transaction ID is found:

```c#
uint height = client.GetTransactionHeight("f4250dab094c38d8265acc15c366dc508d2e14bf5699e12d9df26577ed74d657");
```

## Get validators information
Get the information and voting status of the current NEO consensus nodes:

```c#
RpcValidator[] rpcValidators = client.GetValidators();
foreach (var validator in rpcValidators)
{
    string publicKey = validator.PublicKey;
    BigInteger voteCount = validator.Votes;
    bool isActive = validator.Active;
}
```

## Get node version
Get the version information of the node which RPC requests are sent to:

```c#
RpcVersion rpcVersion = client.GetVersion();
string version = rpcVersion.UserAgent;
```

## Invoke contract method
Return the result after invoking a specific method of a specific contract with specific parameters in the virtual machine:

```c#
RpcStack rpcStack = new RpcStack()
{
    Type = "Hash160",
    Value = "91b83e96f2a7c4fdf0c1688441ec61986c7cae26"
};
RpcInvokeResult rpcInvokeResult = client.InvokeFunction("af7c7328eee5a275a3bcaee2bf0cf662b5e739be", "balanceOf", new RpcStack[] { rpcStack });
string script = rpcInvokeResult.Script;
string engineState = rpcInvokeResult.State;
long gasConsumed = long.Parse(rpcInvokeResult.GasConsumed);
ContractParameter[] resultStacks = rpcInvokeResult.Stack;
foreach (var item in resultStacks)
{
    ContractParameterType contractParameterType = item.Type;
    object value = item.Value;
}
string transaction = rpcInvokeResult.Tx;
```

## Invoke scripts
Returns the result after invoking specific scripts in the virtual machine:

```c#
byte[] script = "00046e616d656724058e5e1b6008847cd662728549088a9ee82191".HexToBytes();
RpcInvokeResult rpcInvokeResult = client.InvokeScript(script);
```

## Get plugin information
Get a list of plugins loaded by the node which RPC requests are sent to:

```c#
RpcPlugin[] rpcPlugins = client.ListPlugins();
foreach (var item in rpcPlugins)
{
    string name = item.Name;
    string version = item.Version;
}
```

## Send transactions
Send and broadcast a serialized transaction to the NEO network:

```c#
bool result = client.SendRawTransaction("80000001195876cb34364dc38b730077156c6bc3a7fc570044a66fbfeeea56f71327e8ab0000029b7cffdaa674beae0f930ebe6085af9093e5fe56b34a5c220ccdcf6efc336fc500c65eaf440000000f9a23e06f74cf86b8827a9108ec2e0f89ad956c9b7cffdaa674beae0f930ebe6085af9093e5fe56b34a5c220ccdcf6efc336fc50092e14b5e00000030aab52ad93f6ce17ca07fa88fc191828c58cb71014140915467ecd359684b2dc358024ca750609591aa731a0b309c7fb3cab5cd0836ad3992aa0a24da431f43b68883ea5651d548feb6bd3c8e16376e6e426f91f84c58232103322f35c7819267e721335948d385fae5be66e7ba8c748ac15467dcca0693692dac");
```

When `result` is `true`, it means the transaction is successfully broadcasted;
when `result` is `false`, it means the transaction is NOT successfully broadcasted and the reason may be double spending, incomplete signature and etc.

## Send blocks
Send and broadcast a serialized block to the NEO network:

```c#
bool result = client.SubmitBlock("000000000000000000000000000000000000000000000000000000000000000000000000845c34e7c1aed302b1718e914da0c42bf47c476ac4d89671f278d8ab6d27aa3d65fc8857000000001dac2b7c00000000be48d3a3f5d10013ab9ffee489706078714f1ea2010001510400001dac2b7c00000000400000455b7b226c616e67223a227a682d434e222c226e616d65223a22e5b08fe89a81e882a1227d2c7b226c616e67223a22656e222c226e616d65223a22416e745368617265227d5d0000c16ff28623000000da1745e9b549bd0bfa1a569971c77eba30cd5a4b00000000400001445b7b226c616e67223a227a682d434e222c226e616d65223a22e5b08fe89a81e5b881227d2c7b226c616e67223a22656e222c226e616d65223a22416e74436f696e227d5d0000c16ff286230008009f7fd096d37ed2c0e3f7f0cfc924beef4ffceb680000000001000000019b7cffdaa674beae0f930ebe6085af9093e5fe56b34a5c220ccdcf6efc336fc50000c16ff2862300be48d3a3f5d10013ab9ffee489706078714f1ea201000151");
```

When `result` is `true`, it means the block is successfully broadcasted;
when `result` is `false`, it means the block is NOT successfully broadcasted and an exception occurs.

## Validate addresses
Validate if an address is a valid NEO address:

```c#
RpcValidateAddressResult result = client.ValidateAddress("AQVh2pG732YvtNaxEGkQUei3YA4cvo7d2i");
bool isValid = result.IsValid;
```
