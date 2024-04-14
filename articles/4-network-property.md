# 4 - Network Property

Network property allows us to replicate things between peers to make them keep in sync.
In this tutorial, we are going to replicate our mesh color between players using inputs and network property.

## Randomize Color Input
Let's add one more type of input which is a `bool` of randomizeColor. If the bool is true, then It will randomize a color, this bool indicate whether a button is being pressed.

```cs
public struct PlayerCharacterInput : INetworkInput
{
    //...    
    public bool RandomizeColor;    
}
```

Let's also modify our `GameplayManager` to also send the randomize color bool input using Space key code.
```cs
public class GameplayManager : NetworkEventsListener
{
    //...
    public override void OnInput(NetworkSandbox sandbox)
    {
        //...
        input.RandomizeColor = Input.GetKey(KeyCode.Space);

        sandbox.SetInput(input);
    }
    //...
}

```

## Defining Network Property

1. Create a new C# Script and name it `PlayerCharacterVisual`
2. Replace the parent class from `MonoBehaviour` to `NetworkBehaviour`
3. Declare a network property of a color. 
```cs
public class PlayerCharacterVisual : NetworkBehaviour
{
    [Networked] public Color MeshColor { get; set; }
}
```
When specifying networked properties, Netick will substitute the provided get and set stubs with custom code.

4. Add a Fetch Input logic to set the color to random

```cs
public class PlayerCharacterVisual : NetworkBehaviour
{
    [Networked] public Color MeshColor { get; set; }

    public override void NetworkFixedUpdate()
    {
        if (FetchInput(out PlayerCharacterInput input))
        {
            if (input.RandomizeColor)
                MeshColor = Random.ColorHSV(0f, 1f);
        }
    }
}
```

5. Declare a field of MeshRenderer

### Detecting Changes
Netick has a feature to automatically detect whenever a certain network property changes, we call it OnChanged.

1. Create a method and name it `OnChangedColor` with `OnChangedData` parameter
2. Add `[OnChanged]` attribute on top of the method
3. Supply the property name we want to detect inside the `[OnChanged]` attribute which is `MeshColor`
4. Update the material color on `OnChangedColor`

```cs
public class PlayerCharacterVisual : NetworkBehaviour
{
    [Networked] public Color MeshColor { get; set; }

    public MeshRenderer meshRenderer;

    public override void NetworkFixedUpdate()
    {
        if (FetchInput(out PlayerCharacterInput input))
        {
            if (input.RandomizeColor)
                MeshColor = Random.ColorHSV(0f, 1f);
        }
    }

    [OnChanged(nameof(MeshColor))]
    private void OnChangedColor(OnChangedData onChangedData)
    {
        meshRenderer.material.color = MeshColor;
    }
}
```

Don't forget to assign the `meshRenderer` field in our player component

<figure><img src="../images/getting-started/104-networked-color.gif" alt=""><figcaption></figcaption></figure>