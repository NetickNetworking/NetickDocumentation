# Listening to Network Events

---

Netick has several useful callbacks you can use:

| Callbacks                                                                                                                       | Description                                                                                                       | Invoke target |
| ------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ------------- |
| OnStartup(NetworkSandbox sandbox)                                                                                               | Invoked when Netick has been started.                                                                             | Client/Server |
| OnShutdown(NetworkSandbox sandbox)                                                                                              | Invoked when Netick has been shut down.                                                                           | Client/Server |
| OnInput(NetworkSandbox sandbox)                                                                                                 | Invoked to read inputs.                                                                                           | Client/Server |
| OnConnectRequest(NetworkSandbox sandbox, NetworkConnectionRequest request)                                                      | Invoked when a client tries to connect. Use request to decide whether or not to allow this client to connect.     | Server        |
| OnConnectFailed(NetworkSandbox sandbox, ConnectionFailedReason reason)                                                          | Invoked when the connection to the server was refused, or simply failed.                                          | Client        |
| OnConnectedToServer(NetworkSandbox sandbox, NetworkConnection server)                                                           | Invoked when the connection to the server has succeeded.                                                          | Client        |
| OnDisconnectedFromServer(NetworkSandbox sandbox, NetworkConnection server, TransportDisconnectReason transportDisconnectReason) | Invoked when connection to the server ended, or when a network error caused the disconnection.                    | Client        |
| OnPlayerJoined(NetworkSandbox sandbox, NetworkPlayerId player)                                                                  | Invoked when a specific player has joined the game.                                                               | Client/Server |
| OnPlayerLeft(NetworkSandbox sandbox, NetworkPlayerId player)                                                                    | Invoked when a specific player has left the game.                                                                 | Client/Server |
| OnSceneOperationBegan(NetworkSandbox sandbox, NetworkSceneOperation sceneOperation)                                             | Invoked when a scene operation has began.                                                                         | Client/Server |
| OnSceneOperationDone(NetworkSandbox sandbox, NetworkSceneOperation sceneOperation)                                              | Invoked when a scene operation has finished.                                                                      | Client/Server |
| OnObjectCreated(NetworkObject obj)                                                                                              | Invoked when a network object has been created/initialized.                                                       | Client/Server |
| OnObjectDestroyed(NetworkObject obj)                                                                                            | Invoked when a network object has been destroyed/recycled.                                                        | Client/Server |

You can override these methods on a class inheriting from `NetworkEventsListener`, and add it to an object in the scene, and Netick will find it automatically. Or, you can add it to a network prefab that you instantiate, and Netick will also find it can call methods on it.

You could also add the component `NetworkEvents` to an object, which does the same, but the difference is that you can plug your events right into it.

And finally, you can use `Sandbox.Events` to directly subscribe/unsubscribe network events from any script, including network behaviours:

```csharp
public override void NetworkStart()
{
  Sandbox.Events.OnPlayerJoined += OnPlayerJoined ;
}

public override void NetworkDestroy()
{
  Sandbox.Events.OnPlayerJoined -= OnPlayerJoined ;
}

private void OnPlayerJoined (....)
{
}
```

This is the cleanest way of using network events.
