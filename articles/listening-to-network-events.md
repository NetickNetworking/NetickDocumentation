# Listening to Network Events

Netick has several useful callbacks you can use:

| Callbacks                                                                  | Description                                                                                                                | Invoke target |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- | ------------- |
| OnStartup(NetworkSandbox sandbox)                                          | Called when Netick has been started.                                                                                       | Client/Server |
| OnShutdown(NetworkSandbox sandbox)                                         | Called when Netick has been shut down.                                                                                     | Client/Server |
| OnInput(NetworkSandbox sandbox)                                            | Called to read inputs.                                                                                                     | Client/Server |
| OnConnectRequest(NetworkSandbox sandbox, NetworkConnectionRequest request) | Called on the server when a client tries to connect. Use request to decide whether or not to allow this client to connect. | Server        |
| OnConnectFailed(NetworkSandbox sandbox, ConnectionFailedReason reason)     | Called on the client when the connection to the server was refused, or simply failed.                                      | Client        |
| OnConnectedToServer(NetworkSandbox sandbox, NetworkConnection server)      | Called on the client when the connection to the server has succeeded.                                                      | Client        |
| OnClientConnected(NetworkSandbox sandbox, NetworkConnection client)        | Called on the server when a specific client has connected.                                                                 | Server        |
| OnClientDisconnected(NetworkSandbox sandbox, NetworkConnection client)     | Called on the server when a specific client has disconnected.                                                              | Server        |
| OnSceneLoaded(NetworkSandbox sandbox)                                      | Called on both the client and the server when the scene has been loaded.                                                   | Client/Server |
| OnSceneLoadStarted(NetworkSandbox sandbox)                                 | Called on both the client and the server before beginning to load the new scene.                                           | Client/Server |
| OnClientSceneLoaded(NetworkSandbox sandbox, NetworkConnection client)      | Called on the server when a specific client finished loading the scene.                                                    | Server        |
| OnObjectCreated(NetworkObject obj)                                         | Called when a network object has been created/initialized.                                                                 | Client/Server |
| OnObjectDestroyed(NetworkObject obj)                                       | Called when a network object has been destroyed/recycled.                                                                  | Client/Server |

You can override these methods on a class inheriting from NetworkEventsListener, and add it to an object in the scene, and Netick will find it automatically. Or, you can add it to a network prefab that you instantiate, and Netick will also find it can call methods on it.

You could also add the component NetworkEvents to an object, which does the same, but the difference is that you can plug your events right into it.
