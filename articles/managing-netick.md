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
Sandbox.Connect(port, serverIPAddress);
```

### Sending Request Data

You can send a payload with the connection request. This is useful for authentication, or custom client information. For example:

```cs
Sandbox.Connect(port, serverIPAddress, authData);
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
> If your validation or decision isn’t made immediately within `OnConnectRequest`, call `Defer` first. This applies even if you’re not using async/await - any delayed handling counts. Without `Defer`, Netick will automatically accept the request once the callback returns.

## Handling Connection Failures (Client-side)

If the connection fails (for example, if the server is unreachable or refuses the connection),
`OnConnectFailed` network event will be invoked on the client.

```cs
public override void OnConnectFailed(NetworkSandbox sandbox, ConnectionFailedReason reason)
{
    // Handle failed connection (e.g., show a retry option or an error message)

    if (sandbox.TryGetConnectionRefusalData(out var data))
    {
        // Optional: process custom refusal data from the server
    }
}
```

## Disconnecting from the Server (Client-side)

A connected client can disconnect at any time:
```cs
Sandbox.Disconnect();
```

## Kicking Clients (Server-side)

The server can disconnect a specific client through the `Kick` method.
This is useful for enforcing rules, timeouts, or admin actions.

```cs
Sandbox.Kick(someClient);
```

You may also include optional data with the kick (for example, a reason message):
```cs
byte[] message = Encoding.UTF8.GetBytes("You were idle too long.");
Sandbox.Kick(someClient, message);
```

On the client, you can handle this in `OnDisconnectedFromServer`:

```cs
public override void OnDisconnectedFromServer(NetworkSandbox sandbox, NetworkConnection server, TransportDisconnectReason reason)
{
    if (sandbox.TryGetKickData(out var kickData))
    {
        string msg = Encoding.UTF8.GetString(kickData);
        // Display message to user
    }
}
```

> [!Note]
> Support for async request handling, custom refusal data, and custom kick data must be implemented on the transport used in the game, otherwise they will not function.

