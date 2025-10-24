# Managing Netick

---

## Starting and Shutting Down Netick

When you start Netick, you need to specify the mode you want to start it in. Like this:

### Single Peer

#### Start as Client

```csharp
var sandbox = Netick.Unity.Network.StartAsClient(Transport, Port);
```

#### Start as Host (a server with a local player)

```csharp
var sandbox = Netick.Unity.Network.StartAsHost(Transport, Port);
```

#### Start as Server

```csharp
var sandbox = Netick.Unity.Network.StartAsServer(Transport, Port);
```

#### Start as Single-Player (disables low level networking)

```csharp
var sandbox = Netick.Unity.Network.StartAsSinglePlayer();
```

### Multiple Peers (Sandboxing)

[Learn More About Sandboxing](sandboxing.md)

You can start both a client and a server together:

```csharp
   var sandboxes       = Netick.Unity.Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Port              = Port,
     TransportProvider = Transport,
     NumberOfServers   = 1,
     NumberOfClients   = 1
   });
```

Starting multiple servers:

```csharp
   int   portOffset    = 4567;
   int[] ports         = new int[20];
   for (int i = 0; i < 20; i++)
     ports[i]          = portOffset + i;

   // starts multiple servers (20 servers)
   var sandboxes       = Netick.Unity.Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Ports             = ports,
     TransportProvider = Transport,
     NumberOfServers   = 20
   });
```

To shut down Netick completely:

```csharp
Netick.Unity.Network.Shutdown();
```


## Connecting to the Server

To connect the client to the server:

```csharp
sandbox.Connect(port, serverIPAddress);
```

### Sending Request Data

You can send a payload with the connection request. This is useful for authentication, or custom client information. For example:

```cs
sandbox.Connect(port, serverIPAddress, authData);
```

## Connection Request Handling

When a client attempts to connect, the server can inspect and respond to the request before accepting or refusing it.
This is done via `OnConnectRequest` on `NetworkEventListener` or by subscribing manually to `Sandbox.Events.OnConnectRequest`.


```cs
// Option 1: Using a NetworkEventListener
public override async void OnConnectRequest(NetworkSandbox sandbox, NetworkConnectionRequest request)
{
    request.Defer();

    ArraySegment<byte> requestData = new ArraySegment<byte>(request.Data, 0, request.DataLength);
    bool allow = await ValidateClient(requestData);

    if (allow)
        request.Accept();
    else
        request.Refuse(); // or request.Refuse(customData);
}

// Option 2: Subscribing manually
Sandbox.Events.OnConnectRequest += MyOnConnectRequest;
```

> [!WARNING]
> You must call `Defer` on the request if you intend to process it in a non-synchronous manner. Otherwise, Netick will auto-accept the request when leaving the scope of this call.


### Handling Refusal Data on the Client
If the server refuses a connection and includes custom data:

```cs
public override void OnConnectFailed(NetworkSandbox sandbox, ConnectionFailedReason reason)
{
    if (sandbox.TryGetConnectionRefusalData(out ArraySegment<byte> data))
    {
        // Handle refusal data here (e.g. display a reason)
    }
}
```


## Disconnecting and Kicking Clients

### Disconnecting from the Server (Client-only)

```cs
Sandbox.Disconnect();
```

### Kicking Clients (Server-only)
The server can disconnect clients and optionally include custom data, such as a reason:

```cs
byte[] kickMessage = Encoding.UTF8.GetBytes("You were idle too long.");
Sandbox.Kick(someClient, kickMessage);
```

On the client, this data is accessible in `OnDisconnectedFromServer`:

```cs
public override void OnDisconnectedFromServer(NetworkSandbox sandbox, NetworkConnection server, TransportDisconnectReason reason)
{
    if (sandbox.TryGetKickData(out var kickData))
    {
        // Handle custom kick message
        string message = Encoding.UTF8.GetString(kickData);
    }
}
```

> [!Note]
> Support for async request handling, custom refusal data, and custom kick data must be implemented on the transport used in the game, otherwise they will not function.

