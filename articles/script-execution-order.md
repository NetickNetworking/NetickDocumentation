# Script Execution Order

---

The network methods on your Network Behavior classes are called from inside Netick, which means standard Unity `MonoBehaviour` script order control does not work here. To specify the order of execution, use the attributes:

```csharp
[ExecuteAfter(typeof(SomeOtherScript))]  /* to specify that this script executes after SomeOtherScript) */
[ExecuteBefore(typeof(SomeOtherScript))] /* to specify that this script executes before SomeOtherScript */
[ExecutionOrder(65)] /* to specify explicitly the order number
```

Example:

```csharp
 [ExecuteAfter(typeof(SomeOtherScript))]
 public class BomberController : NetworkBehaviour 
 {
      // ... 
 }
```

The methods that are affected by these attributes are

- `NetworkStart`
- `NetworkUpdate`
- `NetworkFixedUpdate`
- `NetworkRender`

The following methods are also affected, but only in respect to the object itself:

- `NetworkDestroy`
- `NetcodeIntoGameEngine`
- `GameEngineIntoNetcode`