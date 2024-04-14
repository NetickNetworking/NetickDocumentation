# Sandboxing

Sandboxing (also known as multi-peer), allow you to start multiple Netick instances in a single Unity process for various purposes:
- Starting multiple clients and a server.
- Starting multiple servers in a single Unity process.

Sandbox Management panel can be accessed by going to `Netick > Settings > Sandboxes`

<figure><img src="../images/sandboxing.png" alt="Interpolation"><figcaption></figcaption></figure>

Important notes while working with multiple sandboxes:

- Try to completely avoid using `static` fields. This is because you will run more than one instance of the game and a single `static` field will conflict between the different Netick instances. If you were using `static` for singleton types, you can do the same by using a Sandbox Prefab. Attach all your singleton-like components to your sandbox prefab and you can access them from any network behaviour using `var mySingleton = Sandbox.GetComponent<TypeOfScript>();`. This way each Netick instance will have its own singleton-like scripts.

- When you want to disable a component on a GameObject, use `SetEnabled` instead of `enabled`. This method respects the running sandboxes so when a hidden sandbox enables a mesh renderer, for instance, it will not be visible because when that sandbox is hidden.


## Starting Netick as Multiple Peers

- Starting a client and a server

```csharp
   var sandboxes = Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Port              = Port,
     TransportProvider = Transport,
     NumberOfServers   = 1,
     NumberOfClients   = 1
   });

```

- Starting multiple servers, and a client:

```csharp
   int[] ports = new int[20];
   for (int i = 0; i < 20; i++)
     ports[i]  = Port + i;

   var sandboxes = Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Port              = Port,
     Ports             = ports,
     TransportProvider = Transport,
     NumberOfServers   = 20,
     NumberOfClients   = 1
   });
```

Here are the useful properties on `NetworkSandbox` for working multiple sandboxes:
```cs
// If the sandbox is visible
Sandbox.IsVisible { get; }

// if input is enabled for the sandbox
Sandbox.InputEnabled { get; }
```