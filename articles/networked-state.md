# Networked State

Networked state is the data of the game that you want to replicate to players. In Netick, networked state is delta compressed, therefore only changes are replicated. If a field of a struct changes, only that field is replicated. If a counter increases, only the delta to the previous value is replicated. If your counter was at 32534536, and now it is at 32534537, it will be replicated as a delta of one. It applies to vectors and quaternions too. Thus, using as little bandwidth as possible.

Every network property/array can be predicted and interpolated too. Allowing you to create complex networked systems easily.

### Network Properties

A network property is a C# property which is replicated across the network. For a property to be networked, the Attribute [<xref:Netick.Networked>] must be added to it. Examples of networked properties:

```csharp
[Networked]
public int              Health   {get; set;}

[Networked]
public float            Speed    {get; set;}

[Networked]
public Vector3          Velocity {get; set;}

[Networked]
public int              Ammo     {get; set;}

[Networked]
public NetworkBool      IsAlive  {get; set;}

[Networked]
public NetworkString32  Name     {get; set;}
```

> [!WARNING]
> Reference types are not networkable.

### Network Arrays

Network arrays are just like regular C# arrays, but their syntax is a bit different. They are defined using the <xref:Netick.NetworkArray`1> generic class.

Example of a network array:

```csharp
[Networked(size: 32)]
public NetworkArray<int> IntArrayExample = new NetworkArray<int>(32) { 55, 66, 77 };
```

> [!WARNING]
> `size` of [<xref:Netick.Networked>(size: 32)] must be the same as the value that is passed to the array constructor `new NetworkArray<int>(32)`

### Network Array Structs

Netick 2 introduces a new type of network array, network arrays that are completely value types - Network Array Structs. These are fixed-size struct arrays available only in 4 fixed sizes: 8, 16, 32, and 64.

Network Array Structs are pretty useful since they can used as members of another struct, or even nested inside other arrays. In addition, they can be sent as RPC parameters.

```csharp
// Network Struct Array Examples

[Networked]
public NetworkArrayStruct8<int>                      IntFixedArray { get; set; } = new int[] {1 , 4 ,5}.ToNetworkStructArray8();

[Networked]
public NetworkArrayStruct8<NetworkArrayStruct8<int>> ArrayOfArrays { get; set; };
```

> [!Note]
> Network Array Structs are treated as if they were simple struct types like `int` or `float`, so they must be defined as a property not as a field (like normal NetworkArray that is non-fixed size).

#### Changing elements of Network Array Struct

Because Network Array Structs are structs, the whole array will be replaced even when you change a single element. To avoid bugs, this should be how you change array elements:

```csharp
IntFixedArray = IntFixedArray.Set(index, value);
// as you can see, we are reassigning the property with the new changed array which has the change.
```

## Network Collections

In addition to NetworkArray, Netick also has alternatives to C# collections that are fully synced, predicted, and interpolated.

- NetworkLinkedList<T>
- NetworkStack<T>
- NetworkQueue<T>

### Usage examples:

```csharp
[Networked(size: 5)]
public NetworkLinkedList<int>  MyNetworkedList = new NetworkLinkedList<int>(5);

[Networked(size: 5)]
public NetworkQueue<int>       MyNetworkedQueue = new NetworkQueue<int>(5);

[Networked(size: 5)]
public NetworkStack<int>       MyNetworkedStack = new NetworkStack<int>(5);
```

Removing and adding elements is the same as with C# collections.

> [!Note]
> The `size` that you pass to [Networked] and the constructor represents the fixed capacity of the collection. The collections don't support resizing as all network state sizes are set in compile time.

## Network Structs

Netick can synchronize any struct that does not contain class-based arrays or references. Which includes all C# primitive types and Unity/Godot/Flax primitive types.

Example:

```csharp
public struct MyNestedStruct
{
    public int                      Int1;
    public NetworkBool              Bool1;
    public float                    Float1;
    public double                   Double1;
    public Vector3                  Position;
    public Quaternion               Rotation;
    public Color                    Color;
    public NetworkString8           Name;
}

public struct MyStruct
{
    public MyNestedStruct           MyNestedStruct;
    public NetworkArrayStruct8<int> StructArray;
    public int                      Int1;
    public NetworkBool              Bool1;
    public float                    Float1;
    public double                   Double1;
}

[Networked]
public MyStruct MyStructProperty {get; set;}
```

> [!Note] 
> `string` is not supported as a type that can used inside a struct. Use `NetworkString` instead.

## Replication Relevancy

You can choose to replicate a property only to the Input Source client of the object. This is done using the optional parameter `relevancy` of [<xref:Netick.Networked>]. 

Example:

```csharp
[Networked(relevancy: Relevancy.InputSource)] 
public int              Ammo     {get; set;}
```