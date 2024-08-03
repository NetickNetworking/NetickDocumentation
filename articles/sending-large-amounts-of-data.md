# Sending Large Amounts of Data (Byte Arrays)

It's true that RPCs can be used to send small amount of data, but they shouldn't be used to send anything bigger than 500 bytes. For that, the proper way to send data is through the Data Sending API of Netick.

## Usage Example

```cs
  const int MyDataId = 5;

  unsafe void Sending_Data_Example()
  {
    var    text  = "Trying to send some data!";
    byte[] bytes = Encoding.ASCII.GetBytes(text);

    // there are two variations of SendData, one that takes a pointer and one that takes a byte array. We are using the byte array version here.

    // sending to the server (in the client)
    Sandbox.ConnectedServer.SendData(MyDataId , bytes, bytes.Length, TransportDeliveryMethod.Unreliable);

    // sending to a certain player (in the server)
    NetworkConnection playerConn = someObject.InputSource as NetworkConnection;
    playerConn.SendData(5, bytes, bytes.Length, TransportDeliveryMethod.Unreliable);
  }

  // called by subscribing it to Sandbox.Events.OnDataReceived
  unsafe void OnDataReceived(NetworkSandbox sandbox, NetworkConnection sender, byte id, byte* data, int len, TransportDeliveryMethod transportDeliveryMethod)
  {
    if (id == MyDataId) // is the packet i want
    {
      // converting the data into a managed array (example)
      byte[] buffer = new byte[len]; // don't do this, it's just an example
      for (int i = 0; i < len; i++)
        buffer[i] = data[i];
    }
  }
```

> [!WARNING]
> This functionality is dependent on the underlying transport. Make sure `SendUserData` is implemented on the transport you are using. All of the available transports already implement it.
