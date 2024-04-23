# Networked Procedural Generation

When you want to procedurally generate a level of static objects, you don't need to network any of the objects. Since the procedural generation operation can be deterministic, and therefore all you have to sync to be able to independently create the level in the client is the initial random seed.

Example:

```csharp
    public class MapCreator: NetworkBehaviour
    {
        [Networked]
        public int RandomSeed { get; set;}
        public override void NetworkStart()
        {
            if (IsServer)
              RandomSeed = Random.Range(0,1000);
           
            // setting the seed
            Random.InitState(RandomSeed);
            CreateMap();
        }

        public void CreateMap()
        {
            // add here the logic for procedurally generating the map/level.
        }
    }
```