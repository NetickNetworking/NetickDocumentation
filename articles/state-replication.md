# State Replication

In Netick, the network state of the game is delta compressed. Updates to the game are atomic, it's not possible for a property to update in the client without other changed properties to update alongside it. If you change two properties in the server at the same time, you are ensured to have both replicate together in the client. This makes it so that you don't have to worry about packet loss and possible race conditions that might occur due to some data arriving while other not arriving. This simplifies how you program your game as you never have to worry about such things happening.

This also means that when you create an object in the server, assign some initial values to some network properties, when this object is created in the client, inside `NetworkStart` of that object you will the full initial state that you assigned in the server.

