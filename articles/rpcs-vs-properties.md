# RPCs vs Properties

RPCs can be used to replicate non-critical (often visual/cosmetic) events. In contrast, Network Properties are used to replicate critical gameplay state.

Network Properties are best when you have a variable that is constantly changing and whose exact value matters for the duration of the game, because properties will replicate to everyone regardless of whether they joined the game in the middle or the start of it.

**Example: a health property.**

On the other hand, RPCs are only relevant at the time of their execution, meaning any client joining after the call will never receive anything about any RPCs before its time of joining.

**Example: a damage effect event.**

If an event happens infrequently and is merely visual (doesn’t affect gameplay, for example, a sound effect event) you can use an RPC for it. However, you can, and should, avoid using RPCs even for such events, and that’s by using a [change callback](change-callback.md) using `[OnChanged]` attribute. Ideally, RPCs should only be used for setting up things in the server by the client (client->server RPC), or sending chat messages.

## Avoiding using RPCs

It's highly recommended to avoid using RPCs, and only use them when necessary. Especially server->client RPCs, if possible, they should be completely avoided. And client->server RPCs should only be used for sending a player's name or setting up things or similar actions.

- RPCs from the client can be a security concern. Since you can't control how the client calls them. And they are not tick-aligned, which can be a problem if an RPC is intended to be used for tick-accurate gameplay logic. You can use network inputs to handle most of your client->server actions. 

- RPCs from the server to every connected client are expensive. You can always find a way to mimic an RPC using a network property and an `[OnChanged]` event.

## Using `OnChanged` for Events

[OnChanged callbacks](change-callback.md) are very powerful. Their use-cases are endless. For a couple examples:

- A jump counter network property to sync jump audio. Increment it every time you jump, resulting in a callback that you can use to play jump audio.
- A health network property to sync death event. Check if the previous value is higher than the current value, and the current value has turned to 0. This means the player has died, and you can use the callback to create effects or play audio.

## Circular Buffers

One of the easiest ways to avoid many types of RPCs is to use [Circular Buffers](https://en.wikipedia.org/wiki/Circular_buffer). A circular buffer is a `NetworkArray` that you update in a circular/ring fashion. When you reach the end of the array, you start over at the beginning of the array - this is accomplished by using the modulo operator when indexing the array, using an incrementing value. Using this, you can synchronize many short-lived things such as projectiles, hit indicators, and more.

Example:

This example showcases how we can use circular buffers to synchronize hit indicators, usually seen in FPS games. Hit indicators are rotating arrows around the middle of the screen that point to the locations of where you were last being damaged from.

```cs
public struct DamageIndicatorData
{
    public NetworkObjectRef AttackerPlayer;
    public int              Incrementor;
}

private int                                       _hitIncrementor;

[Networked(size: 5)]
public readonly NetworkArray<DamageIndicatorData> DamageIndicatorsSources = new NetworkArray<DamageIndicatorData>(5);

[OnChanged(nameof(DamageIndicatorsSources))]
private void OnDamageIndicatorsSourcesChanged(OnChangedData info)
{
    // invoking an event when the array changes. We subscribe to this event in a UI script to show the damage indicators and fade them overtime. 
    OnDamageIndicatorsSourcesChangedEvent?.Invoke(info.GetArrayChangedElementIndex());
}

// This is an example of modifying the circular buffer.
public void ApplyDamage(NetworkObjectRef AttackerPlayer, int damageAmount)
{
    // ................
    // other code
    if (IsClient)
        return;
    
    DamageIndicatorsSources[_hitIncrementor % DamageIndicatorsSources.Length] = new DamageIndicatorData()
    {
        AttackerPlayer = AttackerPlayer,
        Incrementor    = _hitIncrementor // we included the incrementor variable as part of the struct to force the OnChanged callback to fire again if the same attacker player was assigned.
    };

    _hitIncrementor++;
}


```