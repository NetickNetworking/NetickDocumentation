# Script Execution Order

The network methods on your Network Behavior classes are called from inside Netick, which means standard Unity MonoBehaviour script order control does not work here. To specify the order of execution for classes inheriting from Network Behavior, use the attributes:

```csharp
[ExecuteAfter(typeof(SomeOtherScript))]  /* to specify that this script executes after SomeOtherScript) */
[ExecuteBefore(typeof(SomeOtherScript))] /* to specify that this script executes before SomeOtherScript */
```

Example:

```csharp
 [ExecuteAfter(typeof(SomeOtherScript))]
 public class BomberController : NetworkBehaviour 
 {
      // ... 
 }
```
