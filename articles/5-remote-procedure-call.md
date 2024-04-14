# 5 - Remote Procedure Call
In this tutorial, we are going to use RPC (Remote Procedure Call) to set our nickname randomly. RPC is the most primitive way to sync a certain actions, and is not recommended most of the time, if the actions happened frequently.

[Read more about RPC on here](remote-procedure-calls-rpcs.md)

## UI Setup

1. In PlayerCharacter prefab, on the `Visual` transform, Create `UI > Text - TextMeshPro`
2. It might Prompts you to `Import TMP Essentials`, go ahead import and close the window afterwards.
3. Change the `Canvas` `Render Mode` from `Screen Space - Overlay` to `World Space`
4. Position your canvas to be 
    Pos X: 0
    Pos Y: 2 
5. Change the canvas scale to 0.005 for all axis

<figure><img src="../images/getting-started/105-canvas.png" alt=""><figcaption></figcaption></figure>

6. In your text, change font size to 128 with the color black with middle & center alignment. Make sure your text now is in the center

<figure><img src="../images/getting-started/105-tmp.png" alt=""><figcaption></figcaption></figure>

## PlayerCharacterNametag

1. Create a new script and name it `PlayerCharacterNametag`
2. Change parent class to `NetworkBehaviour`
3. Create a network property of Nickname type of `NetworkString32` (`string` works too, however NetworkString with a fixed capacity is much recommended)

## RPC Implementation

We're going to set the RPC source to `InputSource` and the targets would be `Owner` (or known as Server/Host). This indicate only the input source is able to call this RPC, but only the server will execute the RPC. We also want to set isReliable to true, what this will do is, it will re-send the packet if there's a packet loss.

```cs
public class PlayerCharacterNametag : NetworkBehaviour
{
    [Networked] public NetworkString32 Nickname { get; set; }

    [Rpc(RpcPeers.InputSource, RpcPeers.Owner, isReliable: true)]
    public void RPC_SetNicknameRandom()
    {
        Nickname = new NetworkString32($"Player_{Random.Range(1000, 9999)}");
    }
}
```

## Calling the RPC

RPC can be called from anywhere, but for simplicity we're going to call them inside the `NetworkUpdate` (not to be confused with `NetworkFixedUpdate`) which is just a regular Unity `Update`.
We only want to call the RPC if we have the input authority and if we press the Enter keycode in this case.
`Sandbox.InputEnabled` is optional if you want don't want to use sandboxing and using the traditional build & run method.

```cs
public class PlayerCharacterNametag : NetworkBehaviour
{
    //....

    public override void NetworkUpdate()
    {
        if (IsInputSource && Input.GetKeyDown(KeyCode.Return) && Sandbox.InputEnabled)
        {
            RPC_SetNicknameRandom();
        }
    }

    //...
}
```


## Nickname OnChanged
Then, we're going to utilize the OnChanged callback for our Nickname

```cs
public class PlayerCharacterNametag : NetworkBehaviour
{
    //...
    public TMP_Text TextNametag;
    //...

    [OnChanged(nameof(Nickname))]
    private void OnChangedNickname(OnChangedData onChangedData)
    {
        TextNametag.SetText(Nickname);
    }

    //...
}
```

Assign the TextNametag with the TextMeshPro UI we have created before

## Final Testing

Lets press the Enter key repeatedly to check if the RPC is working

<figure><img src="../images/getting-started/105-nametag.gif" alt=""><figcaption></figcaption></figure>