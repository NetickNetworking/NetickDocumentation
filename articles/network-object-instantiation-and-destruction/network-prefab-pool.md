# Network Prefab Pool

Object pooling is a very effective technique to avoid run-time allocations (and thus, improve performance), by creating a pool of objects of the same type, at the start of the game. So that when you want to instantiate a certain prefab, you will not create a new object in memory. But rather, all instances of that prefab are already created, and you simply grab one out of the pool and initialize it. And when you want to destroy an instance, instead of removing it from memory (which causes GC), you put it back on the pool – recycling it.

Pooling is extremely useful and effective if you have a prefab in your game that you instantiate and destroy repeatedly. For instance, the bomb in Bomberman.

Netick has a built-in pooling system that you can use.

By default, all prefabs are not pooled. To enable pooling for a certain prefab, you must call `InitializePool` **(should be called at the start of Netick in `NetworkEventsListener` so that all prefab instances created are pooled)** on that prefab and pass it the initial amount to create:


```csharp
public override void OnStartup(NetworkSandbox sandbox)
{
     var bombPrefab = sandbox.GetPrefab("Bomb");
     sandbox.InitializePool(bombPrefab, 5);
}
```

> [!NOTE]
> Check out the Bomberman sample if you are confused. It demonstrates pooling of the bomb prefab.


And if this amount happens to be exceeded, Netick will simply create more objects in the pool automatically.

And you don’t need to use special instantiate or destroy methods to deal with pooled prefabs, it all works through the same `Sandbox.NetworkInstantiate` and `Sandbox.Destroy` methods.

Although you still need to reset your objects. However, Netick automatically resets all network properties to their declaration values.

#### Resetting Prefab Instances

You can simply use `NetworkStart` to reset your prefab instances.

<!-- To reset your object, override `NetworkReset` on your class inheriting from `NetworkBehaviour`:

```csharp
public override void NetworkReset()
{
     // reset all non-networked state which need to be reset so that your object is ready to be used again
}
``` -->
