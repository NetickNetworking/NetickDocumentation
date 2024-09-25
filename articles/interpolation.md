# Interpolation

Netick runs at a fixed-time step, equal to the inverse of the tick rate used for the simulation, which you can specify in Netick Settings. Because of that, the motion of network objects will appear unsmooth and jittery. The reason for this is that, usually, your update rate (render rate) is way higher than your fixed network tick rate. The solution to this problem is called interpolation, which means filling in the gaps between these fixed-time steps/ticks:

<figure><img src="../images/interpolation.png" alt="Interpolation"><figcaption></figcaption></figure>

So, for example, at tick 6, the value of a network property is 2.0. And at tick 7, it becomes 3.0.\
Since there are 5 frames between two ticks, the values at each frame would be:

- Frame 1: 2.0 — Beginning of tick 6
- Frame 2: 2.25
- Frame 3: 2.5
- Frame 4: 2.75
- Frame 5: 3 — End of tick 6, beginning of tick 7

## Interpolation of Network Transform

For moving objects, this is important to deal with. Every <xref:Netick.Unity.NetworkTransform> has a slot for a Render transform, which is basically the smoothed/interpolated mesh of the object, while the parent would be the simulated/non-interpolated object.

So, you must break your moving objects into a parent (which has the <xref:Netick.Unity.NetworkTransform>), and a child which is the interpolated object, and has the mesh/s. Then you specify that child in the NetworkTransform RenderTransform property in the inspector. Check the samples if you are confused.


## Interpolation Source

The source of interpolation data can be of two options:

- **Local/Predicted Snapshot**: This is called `Local Interpolation`. It means using the local predicted snapshots for interpolation. This is what you usually use for your local player as you want to use the local predicted snapshots, and it's chosen by default for objects the client is the Input Source of when `Interpolation Source` is set to `Auto`. 

- **Remote Snapshot**: This is called `Remote Interpolation`. It means using the received snapshots from the server for interpolation. This data is delayed, and that's why it's called remote. This is what you usually use for other objects (not your own player) as you want to use the smoothed and buffered server snapshots, and it's chosen by default for objects the client is not the Input Source of when `Interpolation Source` is set to `Auto`.

> [!NOTE]
> When you want the server only to move your local player object, you must switch `Interpolation Source` to `Remote Snapshot`, to keep smooth rendering of the object as it's being controlled remotely and the prediction buffers will contain jittery data as the object is not being moved locally in the client.


## Interpolation of Network Properties

To interpolate a property, add the [<xref:Netick.Smooth>] attribute to its declaration:

```csharp
[Networked][Smooth]
public Vector3 Movement {get; set;}
```

## Automatic Interpolation

To access the interpolated value, by referencing the property in [NetworkRender](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkRender), you automatically get interpolated values:

```csharp
public override NetworkRender()
{
    var interpolatedValue = Movement;
}
```

Automatic Interpolation is implemented by Netick on these types:

- `Float`
- `Double`
- `Vector2`
- `Vector3`
- `Quaternion`
- `Color`
- `Int`
- `NetworkBool`

> [!WARNING]
> Currently this is only supported in Unity. Use Manual Interpolation in other engines.

## Manual Interpolation

To manually interpolate a network property or network array, you can do that using the <xref:Netick.Interpolator> struct. You also have to pass false to `[Smooth]` to inform Netick we want to manually interpolate the property.

### Interpolating Properties

```csharp
[Networked][Smooth(false)]
public MyType MyType {get; set;}

public override NetworkRender()
{
    var    interpolator      = FindInterpolator(nameof(MyType));
    bool   didGetData        = interpolator.GetInterpolationData<MyType>(InterpolationSource.Auto, out var from, out var to, out float alpha);

    MyType interpolatedValue = default;

    // if we were able to get interpolation data
    if (didGetData)
        interpolatedValue    = LerpMyType(from,to,alpha);
    else // if not we just use the non-interpolated value
        interpolatedValue    = MyType;
}

private MyType LerpMyType(MyType from, MyType to, float alpha)
{
    // write the interpolation code here
}
```

### Interpolating Arrays

```csharp
[Networked (size: 10)][Smooth(false)]
public readonly NetworkArray<MyType> MyTypeArray = new NetworkArray<MyType>(10);

public override NetworkRender()
{
    var          interpolator      = FindInterpolator(nameof(MyTypeArray));
    int          index             = 5;
    bool         didGetData        = interpolator.GetInterpolationData<MyType>(InterpolationSource.Auto, index, out var from, out var to, out float alpha);

    MyType       interpolatedValue = default;

    // if we were able to get interpolation data
    if (didGetData)
        interpolatedValue = LerpMyType(from, to, alpha);
    else // if not we just use the non-interpolated value
        interpolatedValue = MyTypeArray[index];
}

private MyType LerpMyType(MyType from, MyType to, float alpha)
{
    // write the interpolation code here
}
```

> [!NOTE]
> You should cache the result to `FindInterpolator` on [NetworkStart](xref:Netick.Unity.NetickBehaviour#Netick_Unity_NetickBehaviour_NetworkStart), instead of calling it repeatedly every frame (NetworkRender is called every frame), since it might be a bit slow.


<!-- #### Teleportation for Manual Interpolation


Teleportation in this context refers to when you want to teleport or snap your character position or variable value, and disable interpolation in that duration.

```csharp

[Networked]
public Tick   TeleportTick {get; set;} // used to sync the teleportation

[Networked][Smooth(false)]
public MyType MyType       {get; set;}

public override NetworkRender()
{
    var    interpolator      = FindInterpolator(nameof(MyType));
    bool   didGetData        = interpolator.GetInterpolationData<int>(InterpolationSource.Auto, out var from, out var to, out float alpha);

    MyType interpolatedValue = default;


    if (interpolation.From.TickValue < TeleportTick)
    {
        interpolatedValue    = MyType;
        return;
    }

    // if we were able to get interpolation data
    if (didGetData)
        interpolatedValue    = LerpMyType(from,to,alpha);
    else // if not we just use the non-interpolated value
        interpolatedValue    = MyType;
}

private MyType LerpMyType(MyType from, MyType to, float alpha)
{
    // write the interpolation code here
}
```

if (interpolation.From.TickValue < TeleportTick || (Vector3.Distance(fromPos, toPos) >= TeleportDistance))
{
  if (_syncPosition)
    RenderTransform.position = transform.position;
  if (_syncRot)
    RenderTransform.rotation = transform.rotation;
  return;
} -->
