# Coming from Netick 1 (Unity)

This is a guide to help you migrate from Netick 1 to Netick 2, for Unity users. It shows you what has changed in Netick 2 and it also shows you many of the new features that Netick 2 brings to your toolset.

First of all, please make a back-up copy of your project. Then carefully read each section of this article. If you need help, please feel free to join our [discord](https://discord.com/invite/uV6bfG66Fx).

## Importing Netick 2

Assuming you have already downloaded Netick 2 package, delete the root folder of Netick 1 from your project, which is located at `Assets/Netick`. After that, simply unpack/copy Netick 2 into your project. It is recommended to do this in your operating system's File Explorer instead of Unity Project Panel.

## Project Settings

Go to Project Settings -> Player -> Other Settings and change these settings to be as follows:

- Allow 'unsafe' code: `true`
- Api compatibility level: `.NET Standard 2.1`

## API Naming Changes:

| Netick 1                          | Netick 2                               |
| --------------------------------- | -------------------------------------- |
| NetworkSandbox.GetRpcCaller       | NetworkSandbox.CurrentRpcCaller        |
| NetworkSandbox.RpcSource          | NetworkSandbox.CurrentRpcSource        |
| NetworkEventsListner              | NetworkEventsListener                  |
| NetworkBehaviour.ApplyToBehaviour | NetworkBehaviour.GameEngineIntoNetcode |
| NetworkBehaviour.ApplyToComponent | NetworkBehaviour.NetcodeIntoGameEngine |
| NetHit                            | LagCompHit                             |

## Game Starter

Now the transport is specified when starting Netick and not using NetickConfig. A field has been added to GameStarter for that.

## Network Events Listener

A parameter for disconnection reason (<xref:Netick.TransportDisconnectReason>) has been added to OnClientDisconnected:

# [Netick 1](#tab/1)

```csharp
public override void OnClientDisconnected(NetworkSandbox sandbox, NetworkConnection client)
{
}
```

# [Netick 2](#tab/2)

```csharp
public override void OnClientDisconnected(NetworkSandbox sandbox, NetworkConnection client, TransportDisconnectReason reason)
{
}
```

---

## Network Behaviour

Add `using Netick.Unity` to every script that you have which inherits from <xref:Netick.Unity.NetworkBehaviour>.

```csharp
using Netick;
using Netick.Unity;

public class MyScript : NetworkBehaviour
{
    ...
}
```

## Network Input

Network inputs are now structs instead of classes, which makes it easy to sync them as network properties if needed.

# [Netick 1](#tab/1)

```csharp
public class MyInput : NetworkInput
{
    public bool       ShootInput;
    public float      MoveDirX, MoveDirY;
}
```

# [Netick 2](#tab/2)

```csharp
public struct MyInput : INetworkInput
{
    public bool       ShootInput;
    public float      MoveDirX, MoveDirY;
}
```

---

Because they are now value types instead of reference types, this means the previous method of populating them won't work anymore. Instead, you have to use another call to update the input.

# [Netick 1](#tab/1)

```csharp
public override void OnInput(NetworkSandbox sandbox)
{
    var input        = sandbox.GetInput<BombermanInput>();
    input.Movement   = GetMovementDir();
    input.PlantBomb |= Input.GetKeyDown(KeyCode.Space);
}
```

# [Netick 2](#tab/2)

```csharp
public override void OnInput(NetworkSandbox sandbox)
{
    var input        = sandbox.GetInput<BombermanInput>();
    input.Movement   = GetMovementDir();
    input.PlantBomb |= Input.GetKeyDown(KeyCode.Space);

    // since this is a struct, you have to call this method too to update input
    sandbox.SetInput<FPSInput>(input);
}
```

---

## OnChanged

Now OnChanged methods must have a parameter of <xref:Netick.OnChangedData> type which can be used to retrieve the previous property value:

# [Netick 1](#tab/1)

```csharp
[Networked]
public int Health { get; set; }

[OnChanged(nameof(Health ))]
private void OnHealthChanged(int previous)
{
      // Something that happens when the Health property changes
}
```

# [Netick 2](#tab/2)

```csharp
[Networked]
public int Health { get; set; }

[OnChanged(nameof(Health ))]
private void OnHealthChanged(OnChangedData onChangedData)
{
    var previous = onChangedData.GetPreviousValue<int>();
}
```

---

It also now supports retrieving previous array values:

```csharp
[Networked(size: 32)]
public NetworkArray<int> ArrayExample = new NetworkArray<int>(32);

[OnChanged(nameof(IntArray))]
private void OnArrayExampleChanged(OnChangedData onChangedData)
{
  // getting the changed element value directly

  var changedPreviousElementValue = onChangedData.GetArrayPreviousElementValue<int>();

  // or just getting the index

  var changedPreviousElementIndex = onChangedData.GetArrayChangedElementIndex();

  // or maybe getting the previous value of another index we want

  var someRandomPreviousElementValue = onChangedData.GetArrayPreviousElementValue<int>(13);
}
```

### Behavioral Change

[OnChanged] methods now will be called for all non-default initialization values - property definition assignments and inspector values. And this happens for the first time when the object is first created, before NetworkStart is called. So if you try to access a class instance variable inside the [OnChanged] method which is initialized inside NetworkStart, it can cause a null reference exception - because NetworkStart is invoked after [OnChanged] method, not before. To fix this, transfer all class instance variables initialization into NetworkAwake (which is called before the first [OnChanged] ever).

## Network Arrays

Network arrays syntax has changed a little bit. They are now field members instead of property members.

# [Netick 1](#tab/1)

```csharp
[Networked (size: 3)]
public NetworkArray<int> IntArrayExample { get; set; }
```

# [Netick 2](#tab/2)

```csharp
[Networked(size: 32)]
public NetworkArray<int> IntArrayExample = new NetworkArray<int>(32) { 55, 66, 77 };
```

---

> [!WARNING]
> Regarding network arrays for Netick 2: `size` of [<xref:Netick.Networked>(size: 32)] must be the same as the value that is passed to the array constructor `new NetworkArray<int>(32)`

As you can see, it's now possible to have initialization values for network arrays.

### Network Array Struct

Netick 2 introduces a new type of network array, network arrays that are completely value types - Network Array Structs. These are fixed-size struct arrays available only in 4 fixed sizes: 8, 16, 32, and 64.

Network Array Structs are pretty useful since they can used as members of another struct, or even nested inside other arrays.

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

## Network Structs

Now all structs are networked by default, so you don't need to add [<xref:Netick.Networked>] to them or even implement custom equality. You no longer have a limit size for a single struct too. You can also now have nested structs. So this works as expected:

```csharp
public struct MyNestedStruct
{
    public int                      Int1;
    public bool                     Bool1;
    public float                    Float1;
    public double                   Double1;
  }

public struct MyStruct
{
    public MyNestedStruct           MyNestedStruct;
    public NetworkArrayStruct8<int> StructArray;
    public int                      Int1;
    public bool                     Bool1;
    public float                    Float1;
    public double                   Double1;
}
[Networked]
public MyStruct MyStructProperty {get; set;}
```

## Input Source

Now, to change the input source of an object you do that directly using the InputSource property setter:

# [Netick 1](#tab/1)

```csharp
// assigning an input source to network object:

Object.PermitInput(myNewInputSource);

// removing the input source from the object:

Object.RevokeInput();
```

# [Netick 2](#tab/2)

```csharp
// assigning an input source to network object:

Object.InputSource = myNewInputSource;

// removing the input source from the object:

Object.InputSource = null;
```

---

Callbacks of NetworkBehaviour, OnInputPermitted and OnInputRevoked, have been removed and replaced by one single callback:

# [Netick 1](#tab/1)

```csharp
public override void OnInputPermitted()
{
  // called on the InputSource machine when InputSource is now equal to this machine.
}

public override void OnInputRevoked()
{
  // called on the InputSource machine when this machine is no longer the InputSource.
}
```

# [Netick 2](#tab/2)

```csharp
public override void OnInputSourceChanged(NetworkPlayer previous)
{
  // this method is called on the server not only on the client InputSource, so the behaviour is different from OnInputPermitted and OnInputRevoked.
  // to imitate the behaviour of OnInputPermitted and OnInputRevoked:

  if (IsInputSource) // same as OnInputPermitted
  {
     // called on the InputSource machine when InputSource is now equal to this machine.
  }

  else if (previous == Sandbox.LocalPlayer) // same as OnInputRevoked
  {
    // called on the InputSource machine when this machine is no longer the InputSource.
  }

}
```

---

## Rpcs

At this moment in time, `string` is not supported as a parameter to Rpcs. Instead, fixed-size structs can be used:

# [Netick 1](#tab/1)

```csharp
[Rpc]
public void MyRpc(string myString)
{

}
```

# [Netick 2](#tab/2)

```csharp
[Rpc]
public void MyRpc(NetworkString32 myString)
{
  string asString = myString.ToString();
}


// this is how you would call the rpc:

var myString = "Hello World!";

myRpc(new NetworkString32(myString));
```

---

Now, you can have static Rpcs on NetworkBehaviour classes which can be pretty useful.

```csharp
[Rpc]
public static void MyStaticRpc(NetickEngine engine, int someRpcPara)
{
  var sandbox = engine.UserObject as NetworkSandbox;
}


// this is how you would call the rpc:

MyStaticRpc(Sandbox.Engine, 56);
```

Note that they must have a NetickEngine as the first parameter.

## Lag Compensation

LagCompensation component class has been removed.

# [Netick 1](#tab/1)

```csharp
Sandbox.GetComponent<LagCompensation>().Raycast(...);
```

# [Netick 2](#tab/2)

```csharp
Sandbox.Raycast(...);
```

---

## Interpolation

<xref:Netick.Interpolator> is now an non-generic struct.

To find an <xref:Netick.Interpolator>, now you simply use the name of the property instead of using an Id.

# [Netick 1](#tab/1)

```csharp
[Networked][Smooth(6)]
public MyType SomeProperty {get; set;}
public override void NetworkStart()
{
    var interpolator = FindInterpolator<MyType>(6);
}
```

# [Netick 2](#tab/2)

```csharp
[Networked][Smooth]
public MyType SomeProperty {get; set;}
public override void NetworkStart()
{
    var interpolator = FindInterpolator(nameof(SomeProperty));
}
```

---

Also, now [<xref:Netick.Smooth>] takes a parameter to specify if it should give auto-interpolated values inside NetworkRender or not, by specifying a true or false value for `auto` parameter of [<xref:Netick.Smooth>].

### Accessing Interpolation Data

To get interpolation data, now instead of using To, From, and Alpha fields of <xref:Netick.Interpolator>, you use GetInterpolationData method of <xref:Netick.Interpolator> struct:

```csharp
bool didGetData = interpolator.GetInterpolationData<int>(InterpolationMode.Auto, out var from, out var to, out float alpha);
```

It also now supports getting interpolation data for network arrays:

```csharp
int myIndex = 4;
bool didGetData = interpolator.GetInterpolationData<int>(InterpolationMode.Auto, myIndex, out var from, out var to, out float alpha);
```

## Replication

Netick 2 introduces a new replication method called Pessimistic Replication (in contrast to Optimistic Replication, which was the only replication method in Netick 1), which ensures that the client always receives the full state together, not partial, but always the full state. In addition, this new replication method uses delta encoding to highly reduce the bandwidth required. This replication method eliminates the burden of having to account for the potential bugs caused by not always having the entire changed state together using Optimistic Replication.

As of now, this is the default and only replication method. But the old Optimistic Replication will come back later in the future. Pessimistic Replication as of now works with AoI by disabling delta encoding, but this will change in the future. When that happens, Pessimistic Replication will be better than Optimistic Replication for almost every single situation. This is why it has not been a priority to make Optimistic Replication present in Netick 2 from the start.

## Network Transport

> [!NOTE]
> If you are not a transport or a transport wrapper developer, you can ignore this section.

In Netick 1, your network transport main script was inheriting from <xref:Netick.NetworkTransport>, which by itself was inheriting from ScriptableObject. But now that's not possible anymore, since ScriptableObject is a Unity class.

Now, <xref:Netick.NetworkTransport> does not inherit from ScriptableObject, which means you no longer can have assets on your project representing a transport like in Netick 1.

To solve this, a wrapper class has been added <xref:Netick.NetworkTransportProvider>, which inherits from ScriptableObject and wraps the network transport:

```csharp
[CreateAssetMenu(fileName = "LiteNetLibTransportProvider", menuName = "Netick/Transport/LiteNetLibTransportProvider", order = 1)]
public class LiteNetLibTransportProvider : NetworkTransportProvider
{
  public override NetworkTransport MakeTransportInstance() => new LiteNetLibTransport();
}
```

`MakeTransportInstance` is called by Netick to create an instance of the transport.

Netick now only receives data in the form of <xref:Netick.BitBuffer>. BitBuffer.SetFrom is used to set a pointer to the data which BitBuffer will use. Take a look at the new LiteNetLib transport to understand how it all works.

```csharp
public unsafe void INetEventListener.OnNetworkReceive(NetPeer peer, NetPacketReader reader, DeliveryMethod deliveryMethod)
{
  if (_clients.TryGetValue(peer, out var c))
  {
    var len = reader.AvailableBytes;
    reader.GetBytes(_bytes, 0, reader.AvailableBytes);

    fixed(byte* ptr = _bytes)
    {
      _buffer.SetFrom(ptr, len, _bytes.Length);
      NetworkPeer.Receive(c, _buffer);
    }
  }
}
```
