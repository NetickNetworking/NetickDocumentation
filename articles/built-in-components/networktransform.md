# Network Transform

<xref:Netick.Unity.NetworkTransform> is a built-in component to network the `Transform` component. It syncs the position and rotation of the `Transform`.

## Settings:

- Render Transform: assign here the transform that you want to use to display smoothed movement. Should be a child of this GameObject.
- Settings: the replications settings of the NetworkTransform. Choose what you want to sync, and whether you want to enable compression for it or not.
- Interpolation Source: read [here](../interpolation.md#interpolation-source) for a detailed explanation.
- Transform Space: the space used when replicating the data.
- Precision: the precision of the data compression. 

## Teleportation

Since `Render Transform` is always interpolated between two ticks, when you instantly move your object into another position, interpolation would still be active on that object, which is undesirable. To fix this, when you want to instantly move the object and disable interpolation for that duration, you must use the `Teleport` method:

### Teleporting Position
```csharp
MyNetworkTransform.Teleport(newPosition);
```

### Teleporting Rotation
```csharp
MyNetworkTransform.Teleport(newRotation);
```

### Teleporting Position and Rotation
```csharp
MyNetworkTransform.Teleport(newPosition, newRotation);
```