# Network Object Instantiation and Destruction

## Instantiate

To Instantiate a network prefab:

```csharp
sandbox.NetworkInstantiate(prefab, position, Quaternion.identity);
```

This must be called only in the server.

<!-- Normally, it’s only possible to instantiate network prefabs on the server. However, it’s possible to spawn-predict them on the client, check out the next section for that. -->

## Destroy

To destroy a network object:

```csharp
sandbox.Destroy(obj)
```
This will destroy `obj` and all of its nested Network Objects. 

This must be called only in the server.

<!-- This will destroy `obj` and all of its nested Network Objects. Should be called only from the server/owner, although it can also be used to destroy spawn-predicted objects on the client with invalid Ids. -->

> [!WARNING]
> Make sure to never use Unity’s instantiate/destroy methods to create/destroy a network object, only Netick’s methods.
> [!WARNING]
> Make sure that all your prefabs are registered by Netick in Netick Settings panel. And also make sure the prefab list is identical in both the client and the server (if you are running two Unity editors), otherwise, weird stuff will occur.

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
