# How To Write a Transport Wrapper

## Introduction

A transport is the low-level component that does the actual data sending, receiving and handling connections.

This guide will show how to implement a wrapper for the [Unity Transport](https://docs.unity3d.com/Packages/com.unity.transport@2.0/manual/index.html) 


## Defining the Connection Wrapper

First you need to define a connection class which you will pass to Netick. This represents a transport connection. It must implement several properties and a send method:

```csharp
public unsafe class NetickUnityTransportConnection : TransportConnection
{
  public NetickUnityTransport                         Transport;
  public Unity.Networking.Transport.NetworkConnection Connection;

  public override IEndPoint                           EndPoint => Transport._driver.GetRemoteEndpoint(Connection).ToNetickEndPoint();
  public override int                                 Mtu      => MaxPayloadSize;

  public int                                          MaxPayloadSize;

  public NetickUnityTransportConnection(NetickUnityTransport transport)
  {
    Transport = transport;
  }

  public unsafe override void Send(IntPtr ptr, int length)
  {
    if (!Connection.IsCreated)
      return;
    Transport._driver.BeginSend(NetworkPipeline.Null, Connection, out var networkWriter);
    networkWriter.    WriteBytesUnsafe((byte*)ptr.ToPointer(), length);
    Transport._driver.EndSend(networkWriter);
  }
}
```


The `Send` method is called by Netick when it wants to send a packet to this connection. `Transport` represents the UnityTransport transport class which we will talk about in a bit. `Connection` represents the UnityTransport connection that corresponds to this `NetickUnityTransportConnection` type that we will pass into Netick.

### Defining the End Point Wrapper

Let's also define an end point wrapper over UnityTransport `NetworkEndPoint`, and a extension class to do the conversion:

```csharp
public static class NetickUnityTransportExt                          
{ 
    public static NetickUnityTransportEndPoint ToNetickEndPoint(this NetworkEndpoint networkEndpoint) => new NetickUnityTransportEndPoint(networkEndpoint); 
}

public unsafe class NetickUnityTransport : NetworkTransport
{
  public struct NetickUnityTransportEndPoint : IEndPoint
  {
    public NetworkEndpoint EndPoint;
    string       IEndPoint.IPAddress => EndPoint.Address.ToString();
    int          IEndPoint.Port      => EndPoint.Port;
    public NetickUnityTransportEndPoint(NetworkEndpoint networkEndpoint)
    {
      EndPoint = networkEndpoint;
    }
    public override string ToString()
    {
      return $"{EndPoint.Address}";
    }
  }
```


## Defining the Transport Wrapper


```csharp
public unsafe class NetickUnityTransportConnection : TransportConnection
{

}
```

Let's add a few fields which will be important in the functionality of the transport.

```csharp
  private NetworkDriver                                                                            _driver;

  private Dictionary<Unity.Networking.Transport.NetworkConnection, NetickUnityTransportConnection> _connectedPeers        = new();
  private Queue<NetickUnityTransportConnection>                                                    _freeConnections       = new();
  private Unity.Networking.Transport.NetworkConnection                                             _serverConnection;

  private NativeList<Unity.Networking.Transport.NetworkConnection>                                 _connections;

```

`_driver` represents an instance of a UnityTransport manager. `_connectedPeers` contains the a dictionary that maps between the UnityTransport connection type, and the transport wrapper connection type. `_freeConnections` is a pool for free connections that we will use. `_serverConnection` is only relevant when the transport is started as a client, it represents the UnityTransport connection to the server. And `_connections` is the buffer that is used by UnityTransport for the connections.


```csharp
  private BitBuffer                                                                                _bitBuffer; 
  private byte*                                                                                    _bytesBuffer;
  private int                                                                                      _bytesBufferSize         = 2048;
  private byte[]                                                                                   _connectionRequestBytes  = new byte[200];
  private NativeArray<byte>                                                                        _connectionRequestNative = new NativeArray<byte>(200, Allocator.Persistent);
```


`_bitBuffer` is the buffer that is passed to Netick when receiving a packet. Netick only receives the packets in the form of a `BitBuffer`.

`_bytesBuffer` is an unsafe buffer that is used with `_bitBuffer`. `_connectionRequestBytes` is a managed buffer for the connection request.


In the constructor, we allocate `_bytesBuffer`. And we make sure to deallocate, in addition to disposing of `_connectionRequestNative`.

```csharp
  public NetickUnityTransport()
  {
    _bytesBuffer = (byte*)UnsafeUtility.Malloc(_bytesBufferSize, 4, Unity.Collections.Allocator.Persistent);
  }

  ~NetickUnityTransport()
  {
    UnsafeUtility.Free(_bytesBuffer, Unity.Collections.Allocator.Persistent);
    _connectionRequestNative.Dispose();
  }
```

Let's override the `Init` method. This method is called by Netick once to initialize the transport. We initialize the `_bitBuffer` and the UnityTransport network driver `_driver`, and also let's initialize `_connections` buffer.

```csharp
  public override void Init()
  {
    _bitBuffer      = new BitBuffer(createChunks: false);
    _driver      = NetworkDriver.Create(new WebSocketNetworkInterface());
    _connections = new NativeList<Unity.Networking.Transport.NetworkConnection>(Engine.IsServer ? Engine.Config.MaxPlayers : 0, Unity.Collections.Allocator.Persistent);
  }
```

The `Run` method is called by Netick when starting Netick. `Shutdown` is called when shuting down Netick.

```csharp
  public override void Run(RunMode mode, int port)
  {
    if (Engine.IsServer)
    {
      var endpoint = NetworkEndpoint.AnyIpv4.WithPort((ushort)port);

      if (_driver.Bind(endpoint) != 0)
      {
        Debug.LogError($"Failed to bind to port {port}");
        return;
      } 
      _driver.Listen();
    }

    for (int i = 0; i < Engine.Config.MaxPlayers; i++)
      _freeConnections.Enqueue(new NetickUnityTransportConnection(this));
  }

  public override void Shutdown()
  {
    if (_driver.IsCreated)
      _driver.   Dispose();    
    _connections.Dispose();
  }
```

`Connect` method is called by Netick in the client when wanting to connect to the server.

`Disconnect` method is called when you are kicking or disconnecting a connection.

```csharp
  public override void Connect(string address, int port, byte[] connectionData, int connectionDataLength)
  {
    var endpoint        = NetworkEndpoint.LoopbackIpv4.WithPort((ushort)port);
    if (connectionData != null)
    {
      _connectionRequestNative.CopyFrom(connectionData);
      _serverConnection = _driver.Connect(endpoint, _connectionRequestNative);
    }
    else
      _serverConnection = _driver.Connect(endpoint);
  }

  public override void Disconnect(TransportConnection connection)
  {
    var conn = (NetickUnityTransport.NetickUnityTransportConnection)connection;
    if (conn.Connection.IsCreated)
      _driver.Disconnect(conn.Connection);
  }
  ```

Now let's override the last method which is `PollEvents`. This is called by Netick each frame, to poll network events on the transport.


Here we are handling everything from making new connections, handling disconnections, and receiving packets, etc.


```csharp
 public override void PollEvents()
 {
   _driver.ScheduleUpdate().Complete();

   if (Engine.IsClient && !_serverConnection.IsCreated)
     return;

   // reading events
   if (Engine.IsServer)
   {
     // clean up connections.
     for (int i = 0; i < _connections.Length; i++)
     {
       if (!_connections[i].IsCreated)
       {
         _connections.RemoveAtSwapBack(i);
         i--;
       }
     }

     // accept new connections in the server.
     Unity.Networking.Transport.NetworkConnection c;
     while ((c = _driver.Accept(out var payload )) != default)
     {
       if (_connectedPeers.Count >= Engine.Config.MaxPlayers)
       {
         _driver.Disconnect(c);
         continue;
       }

       if (payload.IsCreated)
         payload.CopyTo(_connectionRequestBytes);
       bool accepted = NetworkPeer.OnConnectRequest(_connectionRequestBytes, payload.Length, _driver.GetRemoteEndpoint(c).ToNetickEndPoint());

       if (!accepted)
       {
         _driver.Disconnect(c);
         continue;
       }

       var connection        = _freeConnections.Dequeue();
       connection.Connection = c;
       _connectedPeers.Add(c, connection);
       _connections.   Add(c);

       connection.MaxPayloadSize = NetworkParameterConstants.MTU - _driver.MaxHeaderSize(NetworkPipeline.Null);
       NetworkPeer.    OnConnected(connection);
     }

     for (int i = 0; i < _connections.Length; i++)
       HandleConnectionEvents(_connections[i], i);
   }
   else
     HandleConnectionEvents(_serverConnection, 0);
 }


 private void HandleConnectionEvents(Unity.Networking.Transport.NetworkConnection conn, int index)
 {
   DataStreamReader  stream;
   NetworkEvent.Type cmd;

   while ((cmd = _driver.PopEventForConnection(conn, out stream)) != NetworkEvent.Type.Empty)
   {
     // game data
     if (cmd == NetworkEvent.Type.Data)
     {
       if (_connectedPeers.TryGetValue(conn, out var netickConn))
       {
         stream.     ReadBytesUnsafe(_bytesBuffer, stream.Length);
         _bitBuffer.    SetFrom(_bytesBuffer, stream.Length, _bytesBufferSize);
         NetworkPeer.Receive(netickConn, _bitBuffer);
       }
     }

     // connected to server
     if (cmd == NetworkEvent.Type.Connect && Engine.IsClient)
     {
       var connection = _freeConnections.Dequeue();
       connection.Connection = conn;

       _connectedPeers.Add(conn, connection);
       _connections.   Add(conn);

       connection.MaxPayloadSize = NetworkParameterConstants.MTU - _driver.MaxHeaderSize(NetworkPipeline.Null);
       NetworkPeer.    OnConnected(connection);
     }

     // disconnect
     if (cmd == NetworkEvent.Type.Disconnect)
     {
       if (_connectedPeers.TryGetValue(conn, out var netickConn))
       {
         TransportDisconnectReason reason = TransportDisconnectReason.Shutdown;

         NetworkPeer.     OnDisconnected(netickConn, reason);
         _freeConnections.Enqueue(netickConn);
         _connectedPeers. Remove(conn);
       }

       if (Engine.IsClient)
         _serverConnection   = default;
       if (Engine.IsServer)
         _connections[index] = default;
     }
   }
 }
```

## Defining the Transport Provider

Finally, we have to define a transport provider (a ScriptableObject), which will be used by Netick to create a new instance of the transport wrapper. 

```csharp
[CreateAssetMenu(fileName = "UnityTransportProvider", menuName = "Netick/Transport/UnityTransportProvider", order = 1)]
public class        UnityTransportProvider : NetworkTransportProvider 
{ 
    public override NetworkTransport  MakeTransportInstance() => new NetickUnityTransport(); 
}
```

We can then go to the Assets folder in Unity, and double click and go to Create->Netick->Transport->UnityTransportProvider. Assign the created instance to your GameStarter transport field, and you are done!