# Managing Netick

## **Starting and Shutting Down Netick**

When you start Netick, you need to specify the mode you want to start it in. You can start it as a single sandbox, either a server or a client, like this:

As a client:

```csharp
var sandbox = Netick.Network.StartAsClient(Transport, Port);
```

As a server:

```csharp
var sandbox = Netick.Network.StartAsServer(Transport, Port);
```

Or you can start both a client and a server together:

```csharp
var sandboxes = Netick.Network.StartAsMultiplePeers(Transport, Port, startAServer:true, 1);
```

To shut down Netick completely:

```csharp
Netick.Network.Shutdown();
```

## **Connecting to the Server**

To connect the client to the server:

```csharp
sandbox.Connect(serverIPAddress);
```

## **Disconnecting from the server**

To disconnect the client:

```csharp
sandbox.Disconnect();
```

You are advised to have a game starting scene used for server finding/matchmaking.

## **Scene Switching**

To switch from the current scene to another:

```csharp
sandbox.SwitchScene(2);
```

To find the index of a scene, open the Build Settings window where you will see a list of all added scenes. If the desired scene is not present, open that scene and add it to the list.
