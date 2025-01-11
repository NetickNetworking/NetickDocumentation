# Scene Management 

## Scene Loading

To switch from the current scene to another:

```csharp
Sandbox.SwitchScene("sceneName");
```

### Additive Scenes

Loading an additive scene:

```csharp
Sandbox.LoadSceneAsync("sceneName", LoadSceneMode.Additive);
```

Unloading an additive scene:
```csharp
Sandbox.UnloadSceneAsync("sceneName");
```

> [!WARNING]
> All scene load/unload methods must only be called in the server.

> [!WARNING]
> `UnloadSceneAsync` must only be called for unloading additively loaded scenes. To unload the main scene, use `SwitchScene` or `LoadSceneAsync` with `LoadSceneMode.Single`.

> [!NOTE]
> To find the build index of a scene, open the `Build Settings` window where you will see a list of all added scenes. If the desired scene is not present, open that scene and add it to the list.

## Scene Events

When you call `Sandbox.LoadSceneAsync` in the server, for instance, `OnSceneOperationBegan` event will be invoked in both the client and the server. You can use the `NetworkSceneOperation` parameter to know information about the scene load/unload operation like the current progress.

`OnSceneOperationDone` will be invoked when that scene operation finishes. `NetworkSceneOperation` struct includes a `Scene` getter you can use to access the ` UnityEngine.SceneManagement.Scene` struct.


### Using `NetworkEventsListener`

On a script inheriting from `NetworkEventsListener`, you can run code for when a certain scene operation has began and when it has finished. 

```cs
public override void OnSceneOperationBegan(NetworkSandbox sandbox, NetworkSceneOperation sceneOperation)
 {
  // invoked in both the client and the server when when you call Sandbox.LoadSceneAsync, Sandbox.UnloadSceneAsync, or Sandbox.SwitchScene.
  // sceneOperation lets you know information about the scene operation like the current progress of the scene load/unload.
 }

 public override void OnSceneOperationDone(NetworkSandbox sandbox, NetworkSceneOperation sceneOperation)
 {
  // invoked in both the client and the server when a scene operation caused by calling Sandbox.LoadSceneAsync, Sandbox.UnloadSceneAsync, or Sandbox.SwitchScene finishes.
 }
```

### Using `Sandbox.Events`

Or you can manually subscribe/unsubscribe on a `NetworkBehaviour`.

```cs
public override void NetworkAwake()
{
  Sandbox.Events.OnSceneOperationBegan += OnSceneOperationBegan;
  Sandbox.Events.OnSceneOperationDone  += OnSceneOperationDone;
}

public override void NetworkDestroy()
{
  Sandbox.Events.OnSceneOperationBegan -= OnSceneOperationBegan;
  Sandbox.Events.OnSceneOperationDone  -= OnSceneOperationDone;
}

private void OnSceneOperationBegan(NetworkSandbox sandbox, NetworkSceneOperation sceneOperation)
{
}
private void OnSceneOperationDone(NetworkSandbox sandbox, NetworkSceneOperation sceneOperation)
{
}
```
