# 3 - Player Character Logic

Since Netick is based on server-authoritative netcode, we can't directly move the player character object on the client for security reasons. To move the player based on our input, here's how it works: we send input to the server, the server fetches our input, and then uses it to move our character.

## Input dataset
Consider the type of player inputs required for our gameplay. In this tutorial, we only utilize vector movement and jumping.

1. Create a C# Script call it `PlayerCharacterInput`
2. Change the type into `struct` from `class`
3. Make sure to implement `INetworkInput`

This input will be sent to the server, and can be processed later on.

```cs
using Netick;

public struct PlayerCharacterInput : INetworkInput
{
    public Vector2 Movement;    
    public bool Jump;    
}
```

## Sending Input
There are couple of ways to send input from client to server. The preferred way is on `OnInput` on `NetworkEventsListener`

1. Modify the `GameplayManager` component
2. Override the OnInput method
3. Use `sandbox.SetInput` to set your input

```cs
using Netick.Unity;
using UnityEngine;

public class GameplayManager : NetworkEventsListener
{
    // ...
    
    public override void OnInput(NetworkSandbox sandbox)
    {
        PlayerCharacterInput input = new();
        input.Movement = GetMovement();
        input.Jump = Input.GetKey(KeyCode.Space);

        sandbox.SetInput(input);
    }
    private Vector2 GetMovement()
    {
        float horizontal = Input.GetAxis("Horizontal");
        float vertical = Input.GetAxis("Vertical");

        return new Vector2(horizontal, vertical);
    }
}

```

[Read more about sending inputs](understanding-client-side-prediction/writing-client-side-prediction-code.md)

## Fetch Input

After receiving the input on the server, the server now can check if the input is valid on that current time (or tick)
It is very recommended to do gameplay logic inside `NetworkFixedUpdate`. To get access to this methods, we have to change the parent class from `MonoBehaviour` to `NetworkBehaviour`

Because our input is type of Vector2, we need to translate it to 3D World first before we can supply it as our movement axis

We also need a move speed variable, declare it using type of float and set the default value to 5.

```cs
public class PlayerCharacterMovement : NetworkBehaviour
{
    public float moveSpeed = 5;

    public override void NetworkFixedUpdate()
    {
        if (FetchInput(out PlayerCharacterInput input))
        {
            Vector3 movement = new Vector3(input.Movement.x, 0, input.Movement.y);

            transform.position += movement * Sandbox.FixedDeltaTime * moveSpeed;
        }
    }
}
```

In a singleplayer game, we usually uses Time.deltaTime to move our player to make it frame independent. The difference here is that instead of using Time.deltaTime, we use `Sandbox.FixedDeltaTime`, which represents the time between ticks.

> [!Note]
> Do not confuse `Sandbox.FixedDeltaTime` with `Sandbox.DeltaTime` (equal to Unity's Time.deltaTime) 


[Read more about Network Behaviour](core-concepts.md#network-behaviour)

## Network Rigidbody
Adding `NetworkRigidbody` allowing us to sync the positions and colliding between characters or object

1. Add `NetworkRigidbody` component to our player character prefab
2. Toggle on Freeze Rotation on all Axis (X, Y, Z)

[Read more about NetworkRigidbody](built-in-components/networkrigidbody.md)

### Gameplay & Visual Seperation
In the `NetworkRigidbody` component, there is a `Render Transform` field which ask a Transform.
Because Netick is a tick-based netcode. Meaning an update is executed with a certain time, the position of the player character won't move smoothly. To fix this, we uses interpolation technology which allows us to give smoothed position for our player character visual from position A to position B.

1. Create a child on the player and name It "Visual"
2. Delete & Move the `Capsule (Mesh Filter)` and `Mesh Renderer` component to the Visual
3. Assign Visual object to the `NetworkRigidbody` transform

[Read more about Interpolation](interpolation.md)

Here's what our player character object looks like now

<figure><img src="../images/getting-started/103-player-character.png" alt=""><figcaption></figcaption></figure>

## Multiplayer Testing
Let's try the multiplayer of our characters. In Netick, there is a Sandboxing feature (or multipeer) that allows us to simulate multiple peers on a single instance, meaning we don't have to build the game to test multiplayer functionality

1. Enter play mode
2. Click on `Run Host + Client`
3. Click `Connect`

You can change peers/manage sandboxing in `Netick > Settings > Sandboxes`

[Read more about Sandboxing](sandboxing.md)

<figure><img src="../images/getting-started/103-multipeer.gif" alt=""><figcaption></figcaption></figure>