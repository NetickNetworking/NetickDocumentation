# Writing Client-Side Prediction Code

## Network Input 

Network Input describes what the player wants to do, which will be used to simulate the state of objects they want to control. This ensures that the client can’t directly change the state – the change happens by executing the input, which, even if tampered with, won’t be game-breaking.

## Defining Inputs

To define a new input, create a struct that implements `INetworkInput` interface:

```csharp
[Networked]
public struct MyInput : INetworkInput
{
   [Networked] // adding [Networked] to a struct field and making it a property allows Netick to provide extra compression to it.
   public Vector2     Movement { get; set;}
   public NetworkBool Shoot;
}
```

### Network Input Constraints:
- Must not have class-based network collections as fields. Only `NetworkArrayStruct` variants are allowed as network input fields. 
- Must not have reference types as fields.
- Must not have `string` as a field. Instead, you can use `NetworkString` variants.

## Setting Inputs 

To set the fields of an input, you first need to acquire the input struct of the current tick, using `Sandbox.GetInput`.
Then, you can set it inside `NetworkUpdate` on `NetworkBehaviour`:

```csharp
public override void NetworkUpdate()
{
    var input       = Sandbox.GetInput<MyInput>();
    input.Movement += new Vector2(Mathf.Clamp(Input.GetAxis("Horizontal"), -1f, 1f), Mathf.Clamp(Input.GetAxis("Vertical"), -1f, 1f));
    input.Shoot    |= Input.GetMouseButton(0);
    Sandbox.SetInput(input);
}
```

You could also set them on `OnInput` of `NetworkEventsListener`, which is preferred.

## Simulating (Executing) Inputs

To drive the gameplay based on the input struct, you must do that in `NetworkFixedUpdate`:

```csharp
public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input))
    {
        // clamp movement inputs
        input.Movement     = new Vector3(Mathf.Clamp(input.Movement.x, -1f, 1f), Mathf.Clamp(input.Movement.y, -1f, 1f));

        // movement
        var movementInputs = transform.TransformVector(new Vector3(input.Movement.x, 0, input.Movement.y)) * Speed;
        _CC.Move(movementInputs * Time.fixedDeltaTime);

	    // shooting
        if (input.ShootInput == true && !IsResimulating)
            Shot();
    }
}
```

To solidify your understanding, here's another more abstracted example:

```cs
public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input))
    {
        // predicted action (movement).
        Move(input);

        // non-predicted action (interacting with a world object like a vehicle or a pickup).
        if (IsServer)
            if (input.Interact)
                TryToInteract();
        
        // predicted action, but not resimulated (shooting).
        if (!IsResimulating)
            if (input.ShootInput)
                Shot();
    }
}
```

Here, we see three types of actions:

1. **Predicted**: used for actions like movement. It's the default case.

2. **Non-Predicted**: used for actions that are best left unpredicted. Let's use riding a vehicle as an example. Even though this action can be predicted, it's usually not, to avoid conflicts where multiple players predict that they entered the vehicle as a driver only for it to be mispredicted because another player did the action first, which can be very frustrating and look bad. This is accomplished by making the code only runs in the server using `IsServer`.

3. **Predicted but not resimulated**: used for actions that must only happen during the first time ever when a tick is executed, and not during its resimulations. Usually you use this for things like shooting, to avoid the sound effect playing more than once or the visual effects spawning multiple times.


> [!NOTE]
> Everything that is modified around `FetchInput`, and also affects the networked state, must be networked using `[Networked]`. For example, if you have a variable that is changing over time which can affect the speed of the player, it must be networked.

> [!NOTE]
> When `IsResimulating` equals to true, every `[Networked]` variable has an older value, since we are resimulating past ticks. The first resimulated tick will have the server state applied to every networked variable.

> [!WARNING]
> `Sandbox.GetInput` and `Sandbox.SetInput` are used to read and set the user inputs into the input struct. While `FetchInput` is used to actually use the input struct in the simulation.

> [!NOTE]
> Make sure to clamp inputs to prevent malicious attempts to alter inputs to have big magnitudes leading to speedhacks. Inputs are the only thing the client has authority over.


`FetchInput` tries to fetch an input for the state/tick being simulated/resimulated. It only returns true if either:

1. We are providing inputs to this object – meaning we are the Input Source of the object.
2. We are the owner (the server) of this object – receiving inputs from the client who’s the Input Source. And only if we have an input for the current tick being simulated. If not, it would return false. Usually, that happens due to packet loss.

And to avoid the previous issue we talked about, we make sure that we are only shooting if we are simulating a new input, by checking `IsResimulating`.

## Input Source

For a client to be able to provide inputs to be used in an Object’s `NetworkFixedUpdate`, and hence take control of it, that client must be the Input Source of that object. Otherwise, `FetchInput` will return false. To check if you are the Input Source, use `IsInputSource`.

The server can also be the Input Source of objects, although it won’t do any CSP, since it needs not to, after all, it’s the server.

You can set the Input Source of an object when instantiating it:

```csharp
Sandbox.NetworkInstantiate(PlayerPrefab, spawnPos, Quaternion.identity, client);
```

### Changing the Input Source

To set the Input Source of the object (must only be called on the server):

```csharp
Object.InputSource = newInputSource;
```

To remove the Input Source of the object (must only be called on the server):

```csharp
Object.InputSource = null;
```

## Callbacks

There are two methods you can override to run code when Input Source has changed or left the game:

1. `OnInputSourceChanged`: called on the Input Source and server when the Input Source changes.
2. `OnInputSourceLeft`: called on the owner (server) when the Input Source client has left the game.


## RPCs vs Inputs for Client->Server Actions

Other networking solution rely heavily on the usage of RPCs. However, Netick allows for a much more easier, robust, and safer approach that will make most RPCs obsolete.

Let's take an example to understand this.

Say you want to interact with an object in the game world. A pickup, let's say. Now, instead of sending an RPC to this pickup, which will use additional bandwidth, in addition to being easily cheat-able (by being vulnerable to malicious users who will send RPCs to pickups that are not even nearby), you can simply use an `Interact` input field to implement the logic for it.

```cs
public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input))
    {
        // movement
        ExecuteMovementLogic(input);
        
        if (input.Interact && IsServer) // adding IsServer when you don't want the client to predict this action.
        {
          if (Raycast(..., out var objectToInteractWith)
          {
            // do things
          }
        }
    }
}
```

This way, you have full server-authority on what objects the client can interact with, and the code is very simple to read and debug. It's almost the exact same code you would use for a single-player game, excluding the input fetching logic.

## Framerate Lower than Tickrate

When the framerate is lower than the tickrate, there will be more ticks than frames. Therefore, two or more ticks need to use the same input of one frame. Not handling this can cause the player character to move slower during very low FPS. 

### Enabling `Input Reuse At Low FPS` in Netick Settings

Enabling this option would automatically let Netick reuse/duplicate the same input of one frame to one or more ticks. You can also know if the input fetched is a duplicated input or not as follows:

```csharp
public override void NetworkFixedUpdate()
{
    if (FetchInput(out MyInput input, out bool isDuplicated))
    {
        if (!isDuplicated)
        {
            // do stuff when this input is not a duplicate.
        }
    }
}
```

## Detailed look at what happens in a single tick

> [!NOTE]
> If you are new to Netick, you can ignore this section.

To deeply understand CSP within Netick, let's take a look at what happens during a single tick:

#### A. Start of Tick

The start of a new tick.

#### B. Rollback

The client applies the latest received authoritative server state. `Sandbox.Tick` is changed into `Sandbox.AuthoritativeTick`. Prior to this, `Sandbox.Tick` matched `Sandbox.PredictedTick`.

During this step:

* `NetcodeIntoGameEngine` of `NetworkBehaviour` is invoked on all predicted objects to integrate the networked state into Unity components (e.g., Transform).
* As a result, all predicted objects, in the interest list of the client, are reverted to a past state—this is the essence of rollback.
* Non-predicted objects, in the interest list of the client, receive new states from the server.
* Additionally, any non-predicted objects that were modified locally on the client (but haven't received any new state) will also be rolled back to match the authoritative state.

#### C. Resimulation (Resim)

* `NetworkFixedUpdate` is called on all predicted objects, repeated for the number of times specified by `Sandbox.Resimulations`.
* `Sandbox.IsResimulating` is equal to true.
* This advances the simulation from the rolled-back state to the predicted tick.

At the end of this step:

*  `Sandbox.Tick` is again equal to `Sandbox.PredictedTick`.
* The predicted state is now reconciled with the server. If no mispredictions occurred, the predicted objects will appear unchanged compared to the original state at A. However, non-predicted objects may have changed due to updated server data.

This entire process occurs on the client only. While the following step happens for both the client and the server.

#### D. Forward Tick (Advancing The Simulation Forward)

This marks the actual progression of the game state:

* `NetworkFixedUpdate` is called for all objects (predicted and non-predicted).
* `Sandbox.IsResimulating` is equal to false.

At the end of this step:

* `GameEngineIntoNetcode` of `NetworkBehaviour` is invoked on all objects on the server, and only on predicted objects on the client. This ensures that the network state of components like `NetworkTransform` is updated to reflect the latest state of their corresponding Unity components (e.g., syncing `transform.position` and `transform.rotation` into the internal network variables of `NetworkTransform`).
* `Sandbox.Tick`/`Sandbox.PredictedTick` is incremented by one.

Note: A predicted object refers to any object for which the local player is the Input Source, or any object configured with the Everyone prediction mode.
