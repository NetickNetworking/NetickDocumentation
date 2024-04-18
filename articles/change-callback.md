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
public NetworkArray<int> ArrayExample = new NetworkArray<int>(32);

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
public readonly NetworkLinkedList<int> ListExample = new NetworkArray<int>(32);

[OnChanged(nameof(ListExample))]
private void OnListExampleChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection
  var previous = onChangedData.GetPreviousNetworkStack(ListExample);
}
```

### NetworkQueue<T>

Example:

```csharp
[Networked(size: 32)]
public readonly NetworkQueue<int> QueueExample = new NetworkQueue<int>(32);

[OnChanged(nameof(QueueExample))]
private void OnQueueExampleChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection
  var previous = onChangedData.GetPreviousNetworkQueue(QueueExample);
}
```

### NetworkStack<T>

Example:

```csharp
[Networked(size: 32)]
public readonly NetworkStack<int> StackExample = new NetworkStack<int>(32);

[OnChanged(nameof(StackExample))]
private void OnStackExampleChanged(OnChangedData onChangedData)
{
  // getting a snapshot of the previous state of the collection
  var previous = onChangedData.GetPreviousNetworkStack(StackExample);
}
```

> [!WARNING]
> Don't use the array methods of `OnChangedData` on network collections. They only work on `NetworkArray<T>`.

> [!WARNING]
> Be careful when using these methods on `OnChangedData`, since they are unsafe and can cause a crash if you go outside array range or use an incorrect type.
