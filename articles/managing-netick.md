# Managing Netick

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
sandbox.Connect(serverIPAddress);
```

## Disconnecting From the Server

To disconnect the client:

```csharp
sandbox.Disconnect();
```

You are advised to have a game starting scene used for server finding/matchmaking.

