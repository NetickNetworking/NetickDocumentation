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

## Setting up Scene (WIP)
### Floor
Let's create a 3D Cube with a scale of (15, 1, 15) and name it "Floor"


### Camera
Let's position our camera to 
and change the rotation to 


## Gameplay Manager
Let's create our manager to handle the gameplay such as spawning the character when a certain player joins

Create a C# Script named `GameplayManager`

This script will inherit from `NetworkEventsListener`. By this, `GameplayManager` now has the ability to listen important events such as player join, player left.

## Player Character
Let's create our player character
1. 3D Object > Capsule
2. Add `NetworkObject` component


## Spawning our Player
```cs
public override void OnPlayerConnected(NetworkSandbox sandbox, Netick.NetworkPlayer player)
{
    sandbox.NetworkInstantiate(PlayerPrefab.gameObject, Vector3.zero, Quaternion.identity, player);
}
```