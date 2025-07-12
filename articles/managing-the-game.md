# Managing the Game

---
## Game Managers
In both single-player and multiplayer games, it's common to use the singleton pattern for game management systems. That typically involves static global references and `DontDestroyOnLoad` to persist objects across scene transitions.

In Netick, however, this pattern is streamlined through the use of the **Network Sandbox** — a dedicated `GameObject` (with `DontDestroyOnLoad`) designed to manage an instance of the game. The sandbox acts as a built-in singleton where you can attach all your management scripts directly by adding them to the sandbox prefab.

These scripts can be either standard `MonoBehaviour`s or `NetworkBehaviour`s, enabling the use of networked properties and RPCs as needed.

For example, within any `NetworkBehaviour`, you can easily access these management scripts using `GetComponent`:

```csharp
var manager = Sandbox.GetComponent<Manager>();
```

Sometimes, you may want to find which sandbox instance a non-networked GameObject belongs to:

```csharp
var sandbox = NetworkSandbox.FindSandboxOf(gameObject);
```

> [!Note]
> This method should only be called when you are certain that Netick is running and a valid sandbox instance exists. Calling it before initialization results in a null value.


### Persistent Network Objects

Another option for game-wide systems is to use persistent network objects. These must be instances of network prefabs with the `Is Persistent` property enabled. This ensures they can always be instantiated.

### Additive Scene Loading

Some developers prefer to use additive scenes to separate game management logic. While this approach is not recommended in most cases, it is supported. If you choose to use additive scenes, ensure that they are loaded **after** Netick has initialized — using Netick’s scene loading API.

## NetworkPlayer and NetworkPlayerId

In Netick, each connected player is represented by a `NetworkPlayer` instance. This abstraction differs slightly between the client and the server:

* On the server:

  * The host player is represented as a `NetworkPlayer`.
  * Each connected client is represented as a `ServerConnection`, which inherits from `NetworkPlayer`.

* On the client:

  * The local player is a `NetworkPlayer`.
  * The server (or host) is represented as a `ClientConnection`, which also inherits from `NetworkPlayer`.
  * Other remote clients are **not represented** as `NetworkPlayer` instances, since the client is only connected directly to the server—not to other clients.

To enable consistent player identification across the network, each `NetworkPlayer` is assigned a unique `NetworkPlayerId` (accessible via `NetworkPlayer.PlayerId`). This struct serves as a network-safe identifier.

For this reason:

* The `Sandbox.Players` list—used to track all connected players—is a synchronized list of `NetworkPlayerId`, not `NetworkPlayer` instances.
* Additionally, since reference types like `NetworkPlayer` cannot be serialized or synced across the network, `NetworkPlayerId` is required for networking purposes.

### Converting Between `NetworkPlayer` and `NetworkPlayerId`

If you have a `NetworkPlayerId` and need to retrieve the corresponding `NetworkPlayer`:

```csharp
var player = Sandbox.GetPlayerById(playerId);
```

However, on the **client**, this method only returns a valid result if the `playerId` corresponds to either:

* The **local player**, or
* The **server (host)** player.

Since clients do not maintain direct connections to other clients, `NetworkPlayer` instances for remote peers do not exist on the client side, and this method will return `null` for those cases.

