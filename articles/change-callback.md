# Change Callback

---

To react to changes in a networked variable, use the [<xref:Netick.OnChanged>] attribute on a method and provide it with the name of the variable. This allows your method to be automatically invoked whenever the specified variable is changed.

The method must include a single parameter of type [<xref:Netick.OnChangedData>], which provides optional access to the previous value of the variable.

## For Properties

Example:

```csharp
[Networked]
public int Health { get; set; }

[OnChanged(nameof(Health))]
private void OnHealthChanged(OnChangedData onChangedData)
{
  var previous = onChangedData.GetPreviousValue<int>();
}
```

## For Arrays

Example:

```csharp
[Networked(size: 16)]
public readonly NetworkArray<int> ArrayExample = new(16);

[OnChanged(nameof(ArrayExample))]
private void OnArrayExampleChanged(OnChangedData onChangedData)
{
  // getting the changed element value directly

  var changedPreviousElementValue = onChangedData.GetArrayPreviousElementValue<int>();

  // or just getting the index

  var changedPreviousElementIndex = onChangedData.GetArrayChangedElementIndex();

  // or maybe getting the previous value of another index we want

  var someRandomPreviousElementValue = onChangedData.GetArrayPreviousElementValue<int>(13);
}
```

## For Collections

Unlike network arrays, where the change callback is invoked for each modified element (by index), other network collections are treated as a single networked property. However, if needed, you can still access a snapshot of the collection's previous state for various purposes.

`NetworkDictionary` Example:

```csharp
[Networked(size: 16)] 
public readonly NetworkDictionary<int, int>  MyNetworkDictionary = new(16);

[OnChanged(nameof(MyNetworkDictionary))]
private void OnMyNetworkDictionaryChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection.
  var previous = onChangedData.GetPreviousNetworkDictionary(MyNetworkDictionary);
}
```

> [!WARNING]
> Don't use the array methods of `OnChangedData` on network collections. They only work on `NetworkArray<T>`.

> [!WARNING]
> Be careful when using these methods on `OnChangedData`, since they are unsafe and can cause a crash if you go outside array range or use an incorrect type.


## Handling Initial Replication with `IsCatchingUp`

`OnChangedData.IsCatchingUp` flag allows you to cull events/side-effects that should only occur when a change is "fresh," and not during the initial process (when late-joining) where the server synchronizes the object's state to the client for the first time, where the change may not be relevant anymore.

The flag is set to **`true`** in two cases:

1.  **Late-Joining:** When the client joins the game and is applying the first state data for an object.
2.  **Gaining/Regaining Interest:** When an object enters the client's interest and the client begins receiving the object's current state.

### Usage Example

You can use `IsCatchingUp` within your `[OnChanged]` callback to immediately return and skip logic that you don't want to run during those two cases. This is useful to prevent audio, particle effects, or other non-essential visual feedback from triggering when an object is merely being initialized or brought back into view.

```csharp
[OnChanged(nameof(SomeAudioTrigger))]
private void OnSomeAudioTriggerChanged(OnChangedData dat)
{
    // Return early when "catching up"
    if (dat.IsCatchingUp)
        return;

    // ...
    // The rest of the logic only runs for "live" changes.
}
```

## Invoke Behavior of `[OnChanged]` Callbacks

* When you change a variable in the server or in the client (on a predicted object), the `[OnChanged]` method will be invoked from the setter of the networked variable, therefore it's immediately invoked when changing the variable.

* In the client, when the client receives data for a networked variable that was changed in the server, the client will also invoke the callback, but only if the received value is different from the current value or when there was a misprediction. A misprediction means the value of the variable before rollback is not equal to the value after rollback and resimulation. Read the [article](understanding-client-side-prediction/understanding-client-side-prediction.md) on Client-Side Prediction to learn more.

* If the server changes a variable multiple times, but then back to the original value before all of this, the client will not invoke the callback, because to the client that networked variable never changed, but to the server it did but it eventually went back to the same value at the start of the tick. Therefore it's important to realize that in this case the callback is invoked multiple times in the server but never in the client.

## Invoking `[OnChanged]` Callbacks During Rollback & Resimulation

Read the [article](understanding-client-side-prediction/understanding-client-side-prediction.md) on Client-Side Prediction before reading this section.

By default, Netick will not invoke `[OnChanged]` callbacks when the client rolls back to the latest received server state, and neither during the resimulation stage of prediction. This is usually the desired behaviour because you only want the callback to fire when the value is changed for the first time (usually in the client, on predicted objects). However, sometimes you want the `[OnChanged]` callback to always be in sync with the value of the networked variable and have it also get invoked during rollback and during resimulation. This is easily possible by simply passing true to `invokeDuringResimulation` optional parameter of `[OnChanged]`.

## Finding Removed and Added Items to Collections

Using the previous snapshot (version) of the collection, we are able to compare the current collection against the previous snapshot to find the items that were added and the items that were removed.

### Example:

This example uses a `NetworkDictionary` but the same applies to other collections.

```cs

 [Networked(size: 16)]
 public readonly NetworkDictionary<int, int> NetworkDictionaryExample = new(16);

 [OnChanged(nameof(NetworkDictionaryExample))]
 void OnNetworkDictionaryExampleChanged(OnChangedData dat)
 {
   var previous = dat.GetPreviousNetworkDictionary(NetworkDictionaryExample);

   // finding the newly added items
   foreach (var item in NetworkDictionaryExample)
     if (!previous.ContainsKey(item.Key)) // does not exist in the previous version of the collection, meaning it's a new item.
       Debug.Log($"{item} was added!");

   // finding the newly removed items
   foreach (var item in previous)
     if (!NetworkDictionaryExample.ContainsKey(item.Key)) // if the current version of the collection does not have the item, it means it was removed.
       Debug.Log($"{item} was removed!");
 }
```