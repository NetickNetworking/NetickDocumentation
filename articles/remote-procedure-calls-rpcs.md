# Remote Procedure Calls (RPCs)

RPCs are method calls on Network Behaviors that are replicated across the network. They can be used for events or to transfer data.

An important use of RPCs is to set up the game and send configuration messages. Use reliable RPCs for things like that.

> [!Note]
> While other solutions are heavily dependent on RPCs, Netick is designed to make usage of RPCs very minimal (less than 3 RPCs in the entire game). RPCs teach bad practices and produce [spaghetti code](https://en.wikipedia.org/wiki/Spaghetti_code). Read the article on [RPCs vs Properties](rpcs-vs-properties.md) for more.

An RPC example:

```csharp
[Rpc(source: RpcPeers.Everyone, target: RpcPeers.InputSource, isReliable: true, localInvoke: false)]
private void MyRpc(int arg1)
{
    // Code to be executed
}
```

You use the [<xref:Netick.Rpc>] attribute to mark a method as an RPC.

## Static RPCs

Static RPCs are RPCs not attached to a specific instance of a network behavior. Static RPCs must have their first parameter as a `NetickEngine` type. Which can be used to access the `NetworkSandbox` instance if needed. 

```csharp
[Rpc]
public static void MyStaticRpc(NetickEngine engine, int someRpcPara)
{
  var sandbox = engine.UserObject as NetworkSandbox;
}

// this is how you would call the rpc:

MyStaticRpc(Sandbox.Engine, 56);
```

> [!WARNING]
> RPCs are not called on resimulated ticks.
> [!WARNING]
> By default all RPCs are unreliable.

### [Rpc] method constraints

- Must have the return type of void.
- Must not have reference types as parameters.
- Must not have class-based network collections as parameters. Only `NetworkArrayStruct` varaints are allowed as array RPC parameters. 
- Must not have `string` as a parameter type. Instead, you can use NetworkString varaints.

### [Rpc] attribute parameters

- Source: the peer/peers the RPC should be sent from
- Target: the peer/peers the RPC will be executed on
- isReliable: whether the RPC is sent reliably or unreliably
- localInvoke: whether to invoke the RPC locally or not

Source and target can be any of the following:

- Owner (the server)
- Input Source: the client which is providing inputs for this Network Object
- Proxies: everyone except the Owner and the Input Source
- Everyone: the server and every connected client

### Source Connection of RPCs

If you need to know which connection (a client, or the server) the current RPC is being executed from, you can use `Sandbox.CurrentRpcSource`

```csharp
[Rpc(source: RpcPeers.Everyone, target: RpcPeers.InputSource, isReliable: true, localInvoke: false)]
private void MyRpc(int arg1)
{
    var rpcSource = Sandbox.CurrentRpcSource;
}
```
