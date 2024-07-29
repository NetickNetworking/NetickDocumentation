# Change Callback

You can have a method get called whenever a networked variable changes, which is very useful. To do that, add the attribute [<xref:Netick.OnChanged>] to the method and give it the name of the variable. The method must must have a parameter of <xref:Netick.OnChangedData> type which can be used to retrieve the previous variable value.

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
[Networked(size: 32)]
public readonly NetworkArray<int> ArrayExample = new NetworkArray<int>(32);

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


### NetworkLinkedList<T>

Example:

```csharp
[Networked(size: 32)]
public readonly NetworkLinkedList<int> MyNetworkLinkedList = new NetworkArray<int>(32);

[OnChanged(nameof(ListExample))]
private void OnMyNetworkLinkedListChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection
  var previous = onChangedData.GetPreviousNetworkLinkedList(MyNetworkLinkedList);
}
```

### NetworkDictionary<TKey,TValue>

Example:

```csharp
[Networked(size: 5)] 
public readonly NetworkDictionary<int, int>  MyNetworkDictionary    = new NetworkDictionary<int, int>(5);

[OnChanged(nameof(MyNetworkDictionary))]
private void OnMyNetworkDictionaryChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection.
  var previous = onChangedData.GetPreviousNetworkDictionary(MyNetworkDictionary);
}
```

### NetworkHashSet<T>

Example:

```csharp
[Networked(size: 5)] 
public readonly NetworkHashSet<int>  MyNetworkHashSet    = new NetworkHashSet<int>(5);

[OnChanged(nameof(MyNetworkHashSet))]
private void OnMyNetworkHashSetChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection.
  var previous = onChangedData.GetPreviousNetworkHashSet(MyNetworkHashSet);
}
```

### NetworkUnorderedList<T>

Example:

```csharp
[Networked(size: 5)] 
public readonly NetworkUnorderedList<int>  MyNetworkUnorderedList    = new NetworkUnorderedList<int>(5);

[OnChanged(nameof(MyNetworkUnorderedList))]
private void OnMyNetworkUnorderedListChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection.
  var previous = onChangedData.GetPreviousNetworkUnorderedList(MyNetworkUnorderedList);
}
```


### NetworkQueue<T>

Example:

```csharp
[Networked(size: 5)]
public readonly NetworkQueue<int> MyNetworkQueue = new NetworkQueue<int>(5);

[OnChanged(nameof(MyNetworkQueue))]
private void OnMyNetworkQueueChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection.
  var previous = onChangedData.GetPreviousNetworkQueue(MyNetworkQueue);
}
```

### NetworkStack<T>

Example:

```csharp
[Networked(size: 5)]
public readonly NetworkStack<int> MyNetworkStack = new NetworkStack<int>(5);

[OnChanged(nameof(MyNetworkStack))]
private void OnMyNetworkStackChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection.
  var previous = onChangedData.GetPreviousNetworkStack(MyNetworkStack);
}
```

> [!WARNING]
> Don't use the array methods of `OnChangedData` on network collections. They only work on `NetworkArray<T>`.

> [!WARNING]
> Be careful when using these methods on `OnChangedData`, since they are unsafe and can cause a crash if you go outside array range or use an incorrect type.
