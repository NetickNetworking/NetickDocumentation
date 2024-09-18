# Lag Compensation [Pro]

## Understanding the Need for Lag Compensation

Due to varying latencies (ping) of connected players, each player will see the world at a different point in time than the server. For instance, when the client sends an input to the server to shoot its weapon, the target that the client was aiming at would be at a different place in the client than the server. Therefore the client would miss its shoots. Because, usually, from the perspective of the client, the positions of other objects (players) are in the _**remote snapshot timeline**_, which is always in the past compared to the timeline of the player-controlled character, which’s the _**predicted snapshot timeline**_.\
\
On the server, **everything is in the present**. While on the client, **only the player-controlled character is in the present, while other players’ (proxies) positions are in the past.** Though this is not always the case, because due to the ability of Netick to do full-world prediction, it’s possible to put proxies in the predicted snapshot timeline, in which case lag compensation wouldn’t be needed.\
\
**So, what’s Lag Compensation?**

Lag Compensation basically means going back in time to what the client was seeing at the time of the shooting, and simulating its shooting in that past view.\
\
**Question**: why not just let the client tell the server the target that it hit and how much damage it dealt?\
**Answer**: we can’t trust the client. We should never trust the client, especially in game-critical aspects like applying damage. **Lag Compensation gives us authority over hit detection.**\
\
Watch this video for a visual explanation:

> [!Video https://www.youtube.com/embed/6EwaW2iz4iA]

## Lag Compensation in Netick

To use Lag Compensation in your project, you first need to enable it in Netick Settings. Go to `Netick Settings -> Lag Compensation` and turn on `Enable`.

### **Setting up your character for Lag Compensation**

You have to add a HitShape component (commonly known as a hitbox) on every part of your character which can move. And in the root of your character, you have to add a `HitShape Container` component which will register all child HitShapes.

<figure><img src="https://netick.net/wp-content/uploads/2022/11/image-1-1024x600.png" alt=""><figcaption><p>

HitShape on each bone

<figure><img src="https://netick.net/wp-content/uploads/2022/11/image-3.png" alt="" height="391" width="407"><figcaption><p>

HitShape Container on the root of the character.

The hierarchy should be as follows:

```
> Root (with NetworkObject)
    > `HitShape Container`
        > Render Transform
            > Character Rig 
                > Character Bone (with HitShape) 
```

> [!WARNING]
> Make sure to enable Lag Compensation in Netick Settings.


### **Performing a Lag-Compensated Raycast in Unity**

```csharp
          // lag-compensated Raycast
            if (Sandbox.Raycast(
                shootPos,
                shootDirection,
                out var hit,
                Object.InputSource,
                Mathf.Infinity,
                includeUnityColliders: true,
                queryTriggerInteraction: QueryTriggerInteraction.Ignore))
            {
                if (hit.HitShape != null)
                {
                    // code to be executed when a HitShape was hit
                }
            }
```

### **Performing a Lag-Compensated OverlapSphere in Unity**

```csharp
          // lag-compensated OverlapSphere
            List<LagCompHit>  overlapSphereHits = new List<LagCompHit>(32);
            Sandbox.OverlapSphere(point,
                _projectileBlastRadius,
                overlapSphereHits,
                InputSource,
                queryTriggerInteraction: QueryTriggerInteraction.Ignore);
```

For a practical example, you might want to get our comprehensive Arena Shooter sample which covers everything we talked about and more: [https://netick.net/arena-shooter-sample/](https://netick.net/arena-shooter-sample/)
