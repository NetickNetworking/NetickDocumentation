# RPCs vs Properties

RPCs can be used to replicate non-critical (often visual/cosmetic) events. In contrast, Network Properties are used to replicate critical gameplay state.

Network Properties are best when you have a variable that is constantly changing and whose exact value matters for the duration of the game, because properties will replicate to everyone.

**Example: a health property.**

On the other hand, RPCs are only relevant at the time of their execution, meaning anyone joining after that will never know anything about any RPCs before it.

**Example: a damage effect event.**

If an event happens infrequently and is merely visual (doesn’t affect gameplay, for example, a sound effect event) you can use an RPC for it.

However, you can, and should, avoid using RPCs even for events, and that’s by using a [change callback](change-callback.md) using \[OnChanged] attribute.

## Avoiding using RPCs

It's highly recommended to avoid using RPCs, and only use them when necessary. Especially server->client RPCs, if possible, they should be completely avoided. And client->server RPCs should only be used for sending a player's name or setting up things or similar actions.

- RPCs from the client can be a security concern. Since you can't control how the client calls them. And they are not tick-aligned, which can be a problem if an RPC is intended to be used for tick-accurate gameplay logic. You can use network inputs to handle most of your client->server actions. 

- RPCs from the server to every connected client are expensive. You can always find a way to mimic an RPC using a network property and an [OnChanged] event.
