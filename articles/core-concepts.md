# Core Concepts

### Network Sandbox

<xref:Netick.Unity.NetworkSandbox> is what controls the whole network game. It can be thought of as representing an instance of the game. You can have more than one network sandbox in a single Unity game, and that happens when you start both a client and a server on the same project. This can be extremely useful for testing/debugging, because it allows you to run a server and a client (or multiple thereof) in the same project and therefore see what happens at both at the same time, without interference.

- Therefore you can think of a sandbox as representing a server or a client.
- You can show/hide the current sandboxes from the Network Sandboxes panel.

### Network Object

Any GameObject which needs to be synced/replicated must have a <xref:Netick.Unity.NetworkObject> component added to it. If you want to see something on everyone’s screen, it has to have a <xref:Netick.Unity.NetworkObject> component added to it. It’s the component that tells Netick that a GameObject is networked. The <xref:Netick.Unity.NetworkObject> component by itself just informs Netick that the object is networked. To add networked gameplay-logic to it, you must do so in a component of a class derived from <xref:Netick.Unity.NetworkBehaviour>. Netick comes with a few essential built-in components:

- <xref:Netick.Unity.NetworkTransform>: used to sync position and rotation
- <xref:Netick.Unity.NetworkRigidbody>: used to sync controllable physical objects
- <xref:Netick.Unity.NetworkAnimator>: used to sync Unity’s animator’s state

### Network Behaviour

The <xref:Netick.Unity.NetworkBehaviour> class is your old friend `MonoBehaviour`, just the networked version of it. To implement your networked functionality, create a new class and derive it from <xref:Netick.Unity.NetworkBehaviour>. You have several methods you can override which correspond to Unity’s non-networked equivalents (they must be used instead of Unity’s equivalents when doing anything related to the network simulation):

- [NetworkStart](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkStart)
- [NetworkDestroy](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkDestroy)
- [NetworkFixedUpdate](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkFixedUpdate)
- [NetworkUpdate](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkUpdate)
- [NetworkRender](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkRender)

Example:

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Netick;
using Netick.Unity;

public class MyBehaviour : NetworkBehaviour
{
    [Networked]
    public int   IntPropertyExample   { get; set;}
    [Networked]
    public float FloatPropertyExample { get; set;}

    public override void NetworkStart()
    {
        // Called when this object has been added to the simulation.
    }

    public override void NetworkDestroy()
    {
        // Called when this object has been removed from the simulation.
    }

    public override void NetworkUpdate()
    {
        // Called every frame. Executed before NetworkFixedUpdate.
    }

    public override void NetworkFixedUpdate()
    {
        // Called every fixed-time network step. Any changes to the networked state should happen here.
        // Check out the chapter named "Writing Client-Side Prediction code" to learn more about this method.
    }

    public override void NetworkRender()
    {
        // Called every frame. Executed after NetworkUpdate and NetworkFixedUpdate.
        // IMPORTANT NOTE: properties (which can be interpolated) marked with [Smooth] attribute will return interpolated values when accessed in this method.
    }
}
```

Don’t forget to include `using Netick` and `using Netick.Unity`.

A class derived from <xref:Netick.Unity.NetworkBehaviour> is almost useless without the utilization of Network Properties, which are the building blocks of your networked synced state. Network Properties are delta compressed, letting you create objects with complex states and not worry about it.
