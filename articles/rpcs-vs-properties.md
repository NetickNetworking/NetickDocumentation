# RPCs vs Properties

Remote Procedure Calls (RPCs) and Network Properties (Networked State) serve distinct purposes in a multiplayer game.

RPCs are an option for non-critical, often cosmetic events that occur at specific moments. In contrast, Network Properties are used to synchronize critical gameplay state, especially data that must persist and remain synced for all clients, including late joiners.

## When to Use Network Properties
Network Properties are designed for values that:

- Change frequently.
- Must remain synchronized for all players.
- Need to be known by clients who join mid-game.

Example: a player's health should be a Network Property, as its current value is critical for gameplay and must remain synced across all clients at all times.

## When to Use RPCs
RPCs are event-based, meaning they only execute at a specific point in time. Clients that join after an RPC is called will not receive that RPC. For this reason, they are best used for:
- Visual or cosmetic events.
- One-off setup instructions (e.g. sent from client to server).
- Chat messages or UI updates.

Example: a visual damage effect or sound effect that doesn't impact game logic can be triggered via an RPC.

However, you can, and should, avoid using RPCs even for such events, and that’s by using a [change callback](change-callback.md) using `[OnChanged]` attribute. Ideally, RPCs should only be used for setting up things in the server by the client (client->server RPC), or sending chat messages.

## Avoiding RPCs
In general, you should minimize your usage of RPCs — especially server->client RPCs, which are bandwidth-heavy and harder to manage.

- RPCs from the client can be a security concern. Since you can't control how the client calls them. And they are not tick-aligned, which can be a problem if an RPC is intended to be used for tick-accurate gameplay logic. You can use network inputs to handle most of your client->server actions. 
- RPCs from the server to every connected client are expensive. You can always find a way to mimic an RPC using a network property and an `[OnChanged]` event.

## Using [OnChanged] Callbacks for Events
[OnChanged callbacks](change-callback.md) are very powerful. Their use-cases are endless. For a couple examples:

#### Jump Sound
Use a `JumpCounter` Network Property. Increment it each time the player jumps. In the `[OnChanged]` callback, play the jump sound.

#### Death Effect
Monitor a `Health` Network Property. If it drops to zero (and was higher before), trigger a death animation or sound in the callback.

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