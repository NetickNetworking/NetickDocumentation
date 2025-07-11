# Remote Procedure Calls (RPCs)

---

Remote Procedure Calls (RPCs) are methods defined on `NetworkBehavior` scripts that can be invoked remotely across the network. They are typically used to synchronize discrete events or transmit small amounts of data between clients and the server.

A common use case for RPCs is initializing gameplay logic or sending configuration data at the start of a session. Such tasks should use **reliable RPCs** to ensure delivery.

> [!Note]
> While other solutions are heavily dependent on RPCs, Netick is designed to make usage of RPCs very minimal (less than 3 RPCs in the entire game). RPCs teach bad practices and produce [spaghetti code](https://en.wikipedia.org/wiki/Spaghetti_code). Read the article on [RPCs vs Properties](rpcs-vs-properties.md) for more.

> [!Warning]
> RPCs are not suitable for sending large amounts of data (e.g., over 500 bytes) or transferring files. For those use cases, refer to this [article](sending-large-amounts-of-data.md).

## Basic Example

Here's a simple example of an RPC:

```csharp
[Rpc(source: RpcPeers.Everyone, target: RpcPeers.InputSource, isReliable: true, localInvoke: false)]
private void MyRpc(int arg1)
{
    // Code to be executed remotely
}
```

To declare a method as an RPC, decorate it with the \[`Rpc`] attribute.

## Static RPCs

**Static RPCs** are RPCs not tied to a specific instance of a `NetworkBehavior`. The first parameter of a static RPC must be of type `NetickEngine`, which allows access to the current `NetworkSandbox`.

```csharp
[Rpc]
public static void MyStaticRpc(NetickEngine engine, int someRpcPara)
{
    var sandbox = engine.UserObject as NetworkSandbox;
}

// Invoking the RPC:
MyStaticRpc(Sandbox.Engine, 56);
```

> [!WARNING]
> RPCs are not executed on resimulated ticks.
> [!WARNING]
> Additionally, all RPCs are **unreliable by default** unless explicitly marked otherwise.

## `[Rpc]` Method Requirements

RPC methods must adhere to the following constraints:

* Must have a return type of `void`.
* Reference types are not allowed as parameters.
* Class-based network collections are not allowed. Use `NetworkArrayStruct` for array parameters.
* `string` parameters are not allowed. Use one of the `NetworkString` variants instead.

## `[Rpc]` Attribute Parameters

The `[Rpc]` attribute accepts the following options:

| Parameter     | Description                                     |
| ------------- | ----------------------------------------------- |
| `source`      | Specifies which peer(s) the RPC originates from |
| `target`      | Specifies which peer(s) should execute the RPC  |
| `isReliable`  | If `true`, the RPC will be sent reliably        |
| `localInvoke` | If `true`, the RPC will also be invoked locally |

### Peer Options

The `source` and `target` can be any of the following:

* `Owner` — The server.
* `Input Source` — The player providing input for the object.
* `Proxies` — All peers except the Owner and Input Source.
* `Everyone` — All connected peers, including the server.

## Targeted RPCs

To send an RPC to a specific peer (e.g., a single player), include a parameter of type `NetworkPlayerId` decorated with the `[RpcTarget]` attribute. For instance:

```csharp
[Rpc(source: RpcPeers.Everyone, target: RpcPeers.Everyone, isReliable: true, localInvoke: false)]
private void MyRpc([RpcTarget] NetworkPlayerId target, int arg1)
{
    // ...
}

// Invocation example:
MyRpc(Sandbox.Players[2], someValue);
```

In the case of static RPCs, the `NetworkPlayerId` must be the **second** parameter, following the `NetickEngine` parameter.

## Identifying the RPC Source

To determine which player originally called an RPC, add a final parameter of type `RpcContext`. This allows you to retrieve the `Source` of the RPC at runtime:

```csharp
[Rpc(source: RpcPeers.Everyone, target: RpcPeers.Everyone, isReliable: true, localInvoke: false)]
private void MyRpc(int arg1, RpcContext ctx = default)
{
    var rpcSource = ctx.Source; // Identifies the caller/sender
}
```



