# Sandboxing

Sandboxing (also known as multi-peer) allows you to start multiple Netick instances in a single Unity process for various purposes:
- Starting multiple clients and a server.
- Starting multiple servers in a single Unity process.

Sandboxes panel can be accessed by going to `Netick > Sandboxes`

<figure><img src="../images/sandboxing.png" alt="Interpolation"><figcaption></figcaption></figure>

## Starting Netick as Multiple Peers

### Starting a Client and a Server

```csharp
   var sandboxes       = Netick.Unity.Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Port              = Port,
     TransportProvider = Transport,
     NumberOfServers   = 1,
     NumberOfClients   = 1
   });
```

### Starting Multiple Servers

```csharp
   int   portOffset    = 4561;
   int[] ports         = new int[20];
   for (int i = 0; i < 20; i++)
     ports[i]          = portOffset + i;

   var sandboxes       = Netick.Unity.Network.Launch(StartMode.MultiplePeers, new LaunchData()
   {
     Ports             = ports,
     TransportProvider = Transport,
     NumberOfServers   = 20
   });
```

## Making Your Project Sandbox-Safe

Running any project in multiple-peers mode does not always work, because of how some projects are set up. We call scripts or projects that can work with multiple sandboxes without issues as sandbox-safe.

Notes on how to make your project sandbox-safe:

- Try to completely avoid using `static` variables. This is because you will run more than one instance of the game and the `static` variable will conflict between the different Netick instances, since each Netick instance must have its own copy of that variable. If you were using `static` for singleton types, you can do the same by using a Sandbox Prefab. Attach all your singleton-like components to your sandbox prefab and you can access them from any network behaviour using `var mySingleton = Sandbox.GetComponent<TypeOfScript>();`. This way each Netick instance will have its own singleton-like scripts.

- Use `Sandbox.Physics.Raycast` instead of `Physics.Raycast` when wanting to perform a raycast, same thing goes for other physics queries too. Using `Sandbox.Physics.Raycast` lets you query against the physics scene associated with this sandbox. Since `Physics.Raycast` simply uses the main Unity physics scene that is created when starting Unity, which is not sandbox-safe since it would raycast against objects in the first sandbox only (the sandbox that has the main Unity physics scene associated with it).

- When you want to disable a render/audio component on a GameObject, use `SetEnabled` instead of `enabled`. This method respects the running sandboxes so when a hidden sandbox enables a mesh renderer, for instance, it will not be visible because that sandbox is hidden.

- When you want to instantiate a non-networked GameObject, use `Sandbox.Instantiate` instead of `GameObject.Instantiate`. `Sandbox.Instantiate` respects the running sandboxes, so when a hidden sandbox instantiates a new GameObject, it will not be visible because that sandbox is hidden.

- Use `Sandbox.Log`, `Sandbox.LogWarning`, and `Sandbox.LogError` instead of Unity equivalents. These will include the name of the sandbox at the start of the log message.

- Use `Sandbox.FindObjectOfType`, `Sandbox.FindObjectsOfType`, `Sandbox.FindGameObjectWithTag`, and `Sandbox.FindGameObjectsWithTag` instead of Unity equivalents. The Netick methods respect the running sandboxes, and only try to find objects in the Sandbox's scene.

Some useful properties and events for working with multiple sandboxes:
```cs
// is the sandbox visible
Sandbox.IsVisible 

// is input enabled for the sandbox
Sandbox.InputEnabled 

// invoked when a sandbox visibility state (shown/hidden) changes.
Sandbox.Events.OnVisibilityChanged
```