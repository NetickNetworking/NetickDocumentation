# 2 - Starting a Session

## Game Starter
To Launch a session on Netick, we can call these methods.

```cs
//Start a session with a player
Network.StartAsHost();

//Join a session
Network.StartAsClient();

//Start a session
Network.StartAsServer();
```

For quick testing or development purpose, we can use `GameStarter` component built-in from Netick.

1. Create a new empty GameObject
2. Add the `GameStarter` Component

After adding the component, there are several fields we need to take care of

<figure><img src="../images/getting-started/102-game-starter.png" alt=""><figcaption></figcaption></figure>

## Network Sandbox
The first field ask about a `Sandbox Prefab`

1. Create a new empty GameObject
2. Add `NetworkSandbox` component to it
3. Rename the prefab to `GameNetworkSandbox` (optional)
4. Save the sandbox as a prefab and assign in to the `GameStarter`

Network sandbox is where we do most of the things to manage the game session such as Spawning and Destroying.

## Creating Transport
Now we need to fill out the transport, Netick uses the LiteNetLib by default. To utilize this transport, we can right click on empty assets folder `Create > Netick > Transport > LiteNetLibTransportProvider`.
Then Assign the `Transport` field to the Game Starter

<figure><img src="../images/getting-started/102-create-transport.png" alt=""><figcaption></figcaption></figure>

## Setting up Scene
### Floor
Let's create a 3D Cube with a scale of (15, -1.5, 15) and name it "Floor"

<figure><img src="../images/getting-started/102-floor.png" alt=""><figcaption></figcaption></figure>

### Camera
Modify the camera's position to (0, 10, 12) and adjust its rotation to (45, 180, 0).

<figure><img src="../images/getting-started/102-camera.png" alt=""><figcaption></figcaption></figure>

## Gameplay Manager
Let's create our manager to handle the gameplay such as spawning the character when a certain player joins

Create a C# Script named `GameplayManager` then add it to the GameStarter GameObject

This script will inherit from `NetworkEventsListener`. By this, `GameplayManager` now has the ability to listen important events such as player join, player left.

```cs
// Change parent class from MonoBehaviour to NetworkEventsListener
public class GameplayManager : NetworkEventsListener
{
    
}
```

## Player Character
Let's create our player character
1. Right click on the hierarchy `3D Object > Capsule`
2. Add `NetworkObject` component
3. Rename the file to `PlayerCharacter`

NetworkObject will give a identity across the network, so all players have the reference of It. 

## Spawning our Player
1. Add a field to hold the player character prefab in our gameplay manager
2. Then, let's also spawn the character when a player is connected to the session
On the `NetworkInstantiate` it ask about inputSource. 

Input source is a indicator who has the authority to send input over this object, in this case we should only give the authority to the joined appropriate player.

3. Don't forget to assign the player prefab to the GameplayManger component!

```cs
public class GameplayManager : NetworkEventsListener
{
    public NetworkObject PlayerPrefab;

    public override void OnPlayerConnected(NetworkSandbox sandbox, Netick.NetworkPlayer player)
    {
        sandbox.NetworkInstantiate(PlayerPrefab.gameObject, Vector3.zero, Quaternion.identity, player);
    }
}
```

> [!Note]
> Unity has old API back then when UNet was still around and sometimes giving a wrong signature info on `OnPlayerConnected` in your IDE. This is harmless and can be ignored.

## Testing

Let's go ahead enter play mode and see our player spawning by Clicking on "Start Host"

<figure><img src="../images/getting-started/102-player-spawning.gif" alt=""><figcaption></figcaption></figure>