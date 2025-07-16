# Creating and Destroying Network Objects

---

## Instantiate

To Instantiate a network prefab:

```csharp
Sandbox.NetworkInstantiate(prefab, position, Quaternion.identity);
```

This must be called only in the server.

<!-- Normally, it’s only possible to instantiate network prefabs on the server. However, it’s possible to spawn-predict them on the client, check out the next section for that. -->

## Destroy

To destroy a network object:

```csharp
Sandbox.Destroy(obj)
```
This will destroy `obj` and all of its nested Network Objects. 

This must be called only in the server.

<!-- This will destroy `obj` and all of its nested Network Objects. Should be called only from the server/owner, although it can also be used to destroy spawn-predicted objects on the client with invalid Ids. -->

> [!WARNING]
> Make sure to never use Unity’s instantiate/destroy methods to create/destroy a network object, only Netick’s methods.
> [!WARNING]
> Make sure that all your prefabs are registered by Netick in Netick Settings panel. And also make sure the prefab list is identical in both the client and the server (if you are running two Unity editors), otherwise, weird stuff will occur.

## Network Prefab Pool

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



<!-- ## Spawn-Prediction
> [!CAUTION]
> Spawn-Prediction is WIP in Netick 2. It's non-functional at the moment.

Spawn-Prediction allows you to instantiate/spawn network objects on the client, without having to wait for the server to spawn them. The client would create a local copy of the object, and when the server actually creates the object and the confirmation arrives on the client, the client version of that object would then obtain a valid network Id – meaning it now exists on the network and has been successfully spawn-predicted.

### Usage

To use Spawn-Prediction, you must pass a key `SpawnPredictionKey` to the `NetworkInstantiate` method.

Notes on the key:

- The key must be unique between calls, and usually also between different clients.
- The key must be the same key when the NetworkInstantiate method is called on both the client and the server, for the object to correctly be spawned and confirmed on the network.

When the client locally instantiates the object, and before the confirmation arrives from the server, the object would have an Id of -1 (invalid id). That means it has yet to be confirmed to have been spawned on the server. Using this knowledge, you can know whether or not the object has been successfully spawn-predicted at this point in time. If after a relatively long period the server has yet to create, and therefore confirm, the object, you can choose to destory it on the client.

But It’s important to know that you mustn’t destroy objects, successfully spawn-predicted or not, on the client. Only objects, prefab instances precisely, which have -1 Id are allowed to be destroyed on the client. In the case where you destroy a pending spawn-predicted object which happen to be spawned on the server later on, that object would be re-created on the client.

Notes:

- Netick does not destroy spawn-predicted objects on the client which weren’t spawned on the server. You must destroy them yourself.
- Netick would only call `NetworkStart` once for the spawn-predicted object on the client, and it’s when the client spawns it. It won’t be called when the object has been confirmed. Instead, you can override `OnSpawnPredictionSucceeded` for that.
- Netick automatically destroys all pending spawn-predicted (yet to be confirmed) network objects when input loss occurs on the client.

Check out the Bomberman sample to see the usage of Spawn-Prediction on the Bomb prefab.\

### Spawn-Prediction Example

```csharp
public override void NetworkFixedUpdate()
        {
            if (FetchInput(out BombermanInput input))
            {
                if (!IsResimulating && input.PlantBomb)
                {
                    var spawnKey = new SpawnPredictionKey((byte)Sandbox.Tick.TickValue, (byte)InputSource.PlayerId);
                    Sandbox.NetworkInstantiate(_bombPrefab, Round(transform.position), Quaternion.identity, spawnPredictionKey: spawnKey);
                }
            }
        }
``` -->
