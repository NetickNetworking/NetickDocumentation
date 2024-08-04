# Optimizing Large Numbers of Objects

Usually networked objects in a video game are simulated in one of two ways:


## Internal Simulation
Internal Simulation refers to objects which are simulated and changed from within. Objects such as these include the character of the player. They also include objects such as vehicles, all sorts of physics objects, and game management objects. Any object that is self-controlled and needs to run each tick is within this category.


## External Simulation
External Simulation refers to objects that never simulate or change themselves, instead they are controlled from the outside. There are many examples of such objects: doors, pickups, trees, buildings, etc. All these objects are altered from an external object that belongs to the first category (Internal Simulation). These objects don't need to run themselves or have `NetworkFixedUpdate`, `NetworkUpdate`, or `NetworkRender` run on them each tick/frame. Therefore we should simply exclude them from the network loop, meaning all of their network loop methods (`NetworkFixedUpdate`, `NetworkUpdate`, `NetworkRender`, etc) will not be invoked. This will save a lot of performance, potentially making some types of games possible when they wouldn't be.

Netick makes this very simple. To make an object externally simulated, disable `Add to Network Loop` on its NetworkObject. It will no longer be simulated each tick, but it will still be synced, and its `NetworkStart` and `NetworkDestroy` methods invoked. The object will have no CPU cost almost at all, you can have 1000 objects and shouldn't see a difference in CPU performance, excluding the rendering cost which is irrelevant.

It's important to note that you should stop using the built-in components such as `NetworkTransform` on these objects since these components assume the object is internally simulated. Instead, you would write simple scripts that react to networked properties using `[OnChanged]` callbacks to handle them.

## Example

This is an example of a script that is used to synchronize the position of an object that will only be simulated externally. By a script on the player object, for instance.

```cs
using UnityEngine;
using Netick;
using Netick.Unity;

public class CustomPosSyncer : NetworkBehaviour
{
  [Networked]
  public Vector3 Position { get; set; }

  [OnChanged(nameof(Position))]
  void OnPositionChanged(OnChangedData dat)
  {
    transform.position = Position;
  }
}
```