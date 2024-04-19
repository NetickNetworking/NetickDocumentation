# Scene Management 

## Scene Loading

To switch from the current scene to another:

```csharp
sandbox.SwitchScene("sceneName");
```

### Additive Scenes (Experimental)

Loading an additive scene:

```csharp
sandbox.LoadSceneAsync("sceneName", LoadSceneMode.Additive);
```

Unloading additive scene (this must only be called for unloading an additively loaded scene)
```csharp
sandbox.UnloadSceneAsync("sceneName");
```

To find the build index of a scene, open the Build Settings window where you will see a list of all added scenes. If the desired scene is not present, open that scene and add it to the list.
