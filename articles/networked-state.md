# Networked State

Networked state is the data of the game that you want to replicate to players. In Netick, networked state is delta compressed, therefore only changes are replicated. If a field of a struct changes, only that field is replicated. If a counter increases, only the delta to the previous value is replicated. If your counter was at 32534536, and now it is at 32534537, it will be replicated as a delta of one. It applies to vectors and quaternions too. Thus, using as little bandwidth as possible.

Every networked variable can be predicted and interpolated too. Allowing you to create complex networked systems easily.

## Network Properties

A network property is a C# property which is replicated across the network. For a property to be networked, the attribute [<xref:Netick.Networked>] must be added to it. 

Examples of networked properties:

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

> [!WARNING]
> If you are intending on building your game using `IL2CPP`, you must use `NetworkBool` instead of `bool`.

> [!Note]
> Don't use types with sizes lower than 4 bytes such as `byte` or `short`, instead simply use `int`. Netick already compresses everything to only the required bits of the current value of a variable. So if an `int` variable current value is `5`, it will only be serialized as a few bits, not anywhere near 4 bytes (the byte size of `int`).

## Network Arrays

Network arrays are just like regular C# arrays, but their syntax is a bit different. They are defined using the <xref:Netick.Unity.NetworkArray`1> generic class.

Example of a network array:

```csharp
[Networked(size: 5)]
public readonly NetworkArray<int> IntArrayExample = new NetworkArray<int>(5) { 55, 66, 77 };
```

> [!WARNING]
> `size` of `[Networked(size: 32)]` must be the same as the value that is passed to the array constructor `new NetworkArray<int>(32)`.

## Network Array Structs

Netick 2 introduces a new type of network array, network arrays that are completely value types - Network Array Structs. These are fixed-size struct arrays available only in 4 fixed sizes: 8, 16, 32, and 64.

Network Array Structs are pretty useful since they can used as members of another struct, or even nested inside other arrays. In addition, they can be sent as RPC parameters.

```csharp
// Network Array Struct Examples

[Networked]
public NetworkArrayStruct8<int>                      IntFixedArray { get; set; } = new int[] {1 , 4 ,5}.ToNetworkStructArray8();

[Networked]
public NetworkArrayStruct8<NetworkArrayStruct8<int>> ArrayOfArrays { get; set; };
```

> [!Note]
> Network Array Structs are treated as if they were simple struct types like `int` or `float`, so they must be defined as a property not as a field (like `NetworkArray<T>` that is non-fixed size).

### Changing Elements of Network Array Struct

Because Network Array Structs are structs, the whole array will be replaced even when you change a single element. To avoid bugs, this should be how you change array elements:

```csharp
IntFixedArray = IntFixedArray.Set(index, value);
// as you can see, we are reassigning the property with the new changed array which has the change.
```

## Network Collections

In addition to `NetworkArray<T>`, Netick also has alternatives to C# collections that are fully synced, predicted, and interpolated.

- `NetworkLinkedList<T>`
- `NetworkDictionary<TKey,TValue>`
- `NetworkHashSet<T>`
- `NetworkUnorderedList<T>`
- `NetworkStack<T>`
- `NetworkQueue<T>`

In terms of bandwidth usage, the most expensive collection is `NetworkDictionary`, while the cheapest is `NetworkUnorderedList` (excluding `NetworkArray`). The order from the most expensive to the cheapest is: `NetworkDictionary` > `NetworkLinkedList` > `NetworkHashSet` > `NetworkQueue` > `NetworkStack` > `NetworkUnorderedList`. However, note that this only relates to the bandwidth usage when adding/removing elements, as all collections (or any networked variable) use no bandwidth or CPU time when they're idle and not changing.

### Usage examples:

```csharp
[Networked(size: 5)] 
public readonly NetworkDictionary<int, int>  MyNetworkDictionary    = new NetworkDictionary<int, int>(5);

[Networked(size: 5)]
public readonly NetworkHashSet<int>          MyNetworkHashSet       = new NetworkHashSet<int>(5);

[Networked(size: 5)]
public readonly NetworkLinkedList<int>       MyNetworkLinkedList    = new NetworkLinkedList<int>(5);

[Networked(size: 5)]
public readonly NetworkUnorderedList<int>    MyNetworkUnorderedList = new NetworkUnorderedList<int>(5);

[Networked(size: 5)]
public readonly NetworkStack<int>            MyNetworkedStack       = new NetworkStack<int>(5);

[Networked(size: 5)]
public readonly NetworkQueue<int>            MyNetworkedQueue       = new NetworkQueue<int>(5);
```

Removing and adding elements is the same as with C# collections.

> [!Note]
> The `size` that you pass to `[Networked]` and the constructor represents the fixed capacity of the collection. The collections don't support resizing as all network state sizes are set at compile time.

## Network Structs

Netick can synchronize any struct that does not contain class-based arrays or references. Which includes all C# primitive types and Unity/Godot/Flax primitive types. 

Example:

```csharp
[Networked]
public struct MyNestedStruct
{
    public int                      Int;
    public NetworkBool              Bool;
    [Networked]
    public float                    Float    { get; set;}
    [Networked]
    public Vector3                  Position { get; set;} 
    [Networked]
    public Quaternion               Rotation { get; set;} 
    [Networked]
    public Color                    Color    { get; set;} 
    public NetworkString8           Name;
}

[Networked]
public struct MyStruct
{
    public MyNestedStruct           MyNestedStruct;
    public NetworkArrayStruct8<int> StructArray;
    public int                      Int;
    public NetworkBool              Bool;
    [Networked]
    public float                    Float    { get; set;}
    public double                   Double;
}

[Networked]
public MyStruct MyStructProperty {get; set;}
```

> [!Note] 
> `string` is not supported as a type that can be used inside a struct. Use `NetworkString` instead.

> [!Note] 
> `[Networked]` attribute on structs is optional. However, when adding it to a struct, it allows float-based types (such as `float` or `Vector3`, which also have `[Networked]` on them) of a struct to have extra compression on them.


## Networking References to `NetworkObject` and `NetworkBehaviour`

Since you can't directly synchronize class references, we provide two helper structs that are used to synchronize a reference to `NetworkObject` and `NetworkBehaviour`:

### `NetworkObjectRef`

Usage Example:

```csharp
    public class PlayerController : NetworkBehaviour
    {
        [Networked]
        public NetworkObjectRef MyPlayer { get; set;} 

        public override void NetworkStart()
        {
            // assigning the ref
            MyPlayer = Object.GetRef(); 
        }

        public void ExampleOfUsingTheRef()
        {
            // getting the object from the ref
            var netObj = MyPlayer.GetObject(Sandbox); // or TryGetObject
        }
    }
```

### `NetworkBehaviourRef`

Usage Example:

```csharp
    public class PlayerController: NetworkBehaviour
    {
        [Networked]
        public NetworkBehaviourRef<PlayerController> MyPlayer { get; set;} 

        public override void NetworkStart()
        {
            // assigning the ref
            MyPlayer = this.GetRef<PlayerController>; 
        }

        public void ExampleOfUsingTheRef()
        {
            // getting the behaviour from the ref
            var playerController = MyPlayer.GetBehaviour(Sandbox); // or TryGetBehaviour
        }
    }
```

<!-- > [!WARNING]
> Under the hood, these structs use ids which are `NetworkObject.Id` and `NetworkBehaviour.BehaviourId`, and since ids are recycled between objects, it's possible for a `NetworkObjectRef` and `NetworkBehaviourRef<T>` to refer to incorrect objects if they are not cleared properly when those objects are destroyed. -->

## Replication Relevancy

You can choose to replicate a property only to the Input Source client of the object. This is done using the optional parameter `relevancy` of [<xref:Netick.Networked>]. 

Example:

```csharp
[Networked(relevancy: Relevancy.InputSource)] 
public int              Ammo     {get; set;}
```

## State Synchronization

Updates to the network state are atomic, it's not possible for a property to update in the client without other changed properties to update alongside it. If you change two (or more) properties in the server at the same time (or in two subsequent ticks), you are ensured to have both replicate together in the client. This makes it so that you don't have to worry about packet loss and possible race conditions that might occur due to some properties updates arriving while others arriving later. This simplifies how you program your game as you never have to worry about such things happening.

This also means that when you create an object in the server, assign some initial values to some network properties, when this object is created in the client, inside `NetworkStart` of that object you will have full initial state that you assigned in the server. 

