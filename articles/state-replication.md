# State Replication

In Netick, network state is synced (replicated) using one of these two methods:

## Optimistic Replication (OR):

Optimistic Replication is the replication method used in Netick 1, and its only one.

### How it Works:

All network properties changes are only sent once, and only ever resent when they are lost. It's being optimistic that data will always be received on the client, so it only sends a change once, hoping the client will get it.

### Pros:

This works very well if the network properties that are changing each tick are not the same, like assume you have 1000 properties and each tick a random one of them changes. You are mostly sending one single network property data each time you send a packet, because only one network property is changing. And it is not effected by ping.

### Cons:

This can make coding your game challenging, because you are not assured to always have all changed network properties sent together. If two properties change in two subsequent ticks, and you only receive the last one, you will only have data for that one and not for the previous one. You can easily see how this can cause issues. The way to handle these issues is by coding your game to be eventually synced. Though that might be a bit hard for newbies.

## Pessimistic Replication (PR):

Pessimistic Replication is the replication method used in Netick 2, and for now the only one.

### How it Works:

All network property changes that occur are always sent together and until the client acknowledges the server that it has received all of them. It's being pessimistic that data will always be lost on the client, so it always sends the full set of changed data in case the client has lost some of it.

### Pros:

Because of the guarantee that the client will always have the latest full state, that makes applying techniques like Delta-Encoding pretty easy, since we can have a single reference to what state exists on the client and encode for it. Delta Encoding can cut off the bandwidth by more than half. And we no longer have to account for potential bugs caused by not always having the full state on the client, so this method is overall easier to work with and safer.

### Cons:

Pessimistic Replication is effected by ping. So higher ping might cause more data being sent. Because all network properties that are changing are accumulating on the server waiting for the client to acknowledge them. But this is not as bad as it sounds at all.

## Conclusion:

Because the most bandwidth heavy thing in games is usually spatial data like position and rotation, it does not matter if we only send when the data changes, because it will change each tick anyway. Position and rotation are mostly changing each tick.

Which means Pessimistic Replication will gives us better results for real-time games with many moving entities, which is what most people want to make anyway.
