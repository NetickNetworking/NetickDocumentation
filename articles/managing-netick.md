# Managing Netick

## Starting and Shutting Down Netick

When you start Netick, you need to specify the mode you want to start it in. Like this:

### Single Peer

#### Start as Client:

```csharp
var sandbox = Netick.Network.StartAsClient(Transport, Port);
```

#### Start as Host (a server with a local player):

```csharp
var sandbox = Netick.Network.StartAsHost(Transport, Port);
```

#### Start as Server

```csharp
var sandbox = Netick.Network.StartAsServer(Transport, Port);
```

#### Start As Single-Player (disables low level networking)

```csharp
var sandbox = Netick.Network.StartAsSinglePlayer();
```

### Multiple Peers (Sandboxing)

[Read More About Sandboxing](sandboxing.md)

You can start both a client and a host together:

```csharp
   var sandboxes = Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Port              = Port,
     TransportProvider = Transport,
     NumberOfServers   = 1,
     NumberOfClients   = 1
   });

```

Starting multiple servers:

```csharp

   int   portOffset = 4567;
   int[] ports      = new int[20];
   for (int i = 0; i < 20; i++)
     ports[i]  = portOffset + i;

   // starts multiple servers (20 servers)
   var sandboxes = Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Ports             = ports,
     TransportProvider = Transport,
     NumberOfServers   = 20
   });
```

To shut down Netick completely:

```csharp
Netick.Network.Shutdown();
```

## Connecting to the Server

To connect the client to the server:

```csharp
sandbox.Connect(serverIPAddress);
```

## Disconnecting from the server

To disconnect the client:

```csharp
sandbox.Disconnect();
```

You are advised to have a game starting scene used for server finding/matchmaking.

## Scene Loading

To switch from the current scene to another:

```csharp
sandbox.SwitchScene("sceneName");
```

### Additive Scenes (Experimental)

Loading an additive scene:

```csharp
sandbox.LoadSceneAsync("sceneName", LoadSceneMode.Additive);
```

Unload additive scene (this must only be called for unloading an additively loaded scene)
```csharp
sandbox.UnoadSceneAsync("sceneName");
```

To find the build index of a scene, open the Build Settings window where you will see a list of all added scenes. If the desired scene is not present, open that scene and add it to the list.
