# Parenting

The parent of a network object can only be changed if you are the Input Source of the object, or if you are the owner (server).

To change the parent of an object: `Object.SetParent(newParent);`

> [!CAUTION]
> The original hierarchy of network prefab instances shouldn’t be changed at run-time. In other words, you shouldn’t unparent the original children of a prefab. Although, you can parent objects to them, just not unparenting them (prefab children) from their original parent.

