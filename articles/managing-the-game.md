# Managing the Game in Netick

---

In both single-player and multiplayer games, it's common to use the singleton pattern for game management systems. That typically involves static global references and `DontDestroyOnLoad` to persist objects across scene transitions.

In Netick, however, this pattern is streamlined through the use of the **Network Sandbox** — a dedicated `GameObject` (with `DontDestroyOnLoad`) designed to manage an instance of the game. The sandbox acts as a built-in singleton where you can attach all your management scripts directly by adding them to the sandbox prefab.

These scripts can be either standard `MonoBehaviour`s or `NetworkBehaviour`s, enabling the use of networked properties and RPCs as needed.

For example, within any `NetworkBehaviour`, you can easily access these management scripts using `GetComponent`:

```csharp
var manager = Sandbox.GetComponent<Manager>();
```

---

## Persistent Network Objects

Another option for game-wide systems is to use persistent network objects. These must be instances of network prefabs with the `Is Persistent` property enabled. This ensures they can always be instantiated.

---

## Additive Scene Loading

Some developers prefer to use additive scenes to separate game management logic. While this approach is not recommended in most cases, it is supported. If you choose to use additive scenes, ensure that they are loaded **after** Netick has initialized — using Netick’s scene loading API.

