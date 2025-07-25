# Physics

---

When Netick starts and the `Physics Type` option in Netick Settings/Config is set to either `Physics2D` or `Physics3D` (instead of `None`), Netick takes control of physics stepping by setting `UnityEngine.Physics.simulationMode` (or `UnityEngine.Physics2D.simulationMode`) to `Script`. This ensures physics simulation remains synchronized with Netick’s networked fixed-time loop. Stepping is handled by the `PhysicsSimulationStep` component, which is automatically added to the sandbox GameObject upon Netick initialization.

Each sandbox instance has its own physics scene, which is simulated independently to other sandboxes. By default, only the physics scene of the main scene of the sandbox is stepped.

## Networked Rigidbodies 

To make a `Rigidbody` or `Rigidbody2D` part of the networked simulation, you must add a `NetworkRigidbody` or `NetworkRigidbody2D` component to the same GameObject (which has a `NetworkObject` on it or on one of its parents, of course). This component enables Netick to synchronize the transform and physics state of the object across the network.

## Physics Prediction

Physics prediction involves resimulating multiple physics steps within a single network tick. This is computationally expensive and therefore disabled by default. You can enable it via Netick -> Settings by checking the `Physics Prediction` option.

To enable or disable physics prediction on the client at runtime, use the `Sandbox.PhysicsPrediction` property.

### Performance Cost of Predicting PhysX (3D Physics)

Predicting 3D physics using PhysX is particularly expensive. Unity's integration with PhysX performs poorly when `PhysicsScene.Simulate` is called multiple times per frame, even for a small number of rigidbodies.

The cost of prediction scales primarily with:

* **Ping**: Higher ping requires simulating more ticks during resimulation.
* **Tickrate**: A higher tickrate reduces the time period between ticks, therefore increasing the number of ticks that must be simulated for a given latency.

In practice, enabling 3D physics prediction is not recommended. On some machines (phones, for example), simulating a few physics ticks with a ping of \~100ms at high tickrates (e.g., 33+) can take more than 10ms per frame, making it impractical.

### Alternatives to Physics Prediction

If prediction is not feasible, consider the following approaches:

* **Server-authoritative physics**: Disable physics prediction and rely solely on the server's simulation.
* **Client-authoritative physics**: Let clients simulate and send resulting states via RPCs or network inputs.
* **Integrating a third-party physics engine**: Explore other physics engines such as Bullet, Jolt, or Rapier.

## Local Non-Networked Physics

In many multiplayer games, it’s common to have locally simulated (non-networked) rigidbodies to reduce bandwidth usage. These objects are not synchronized across clients, so their behavior may differ per client — which is generally acceptable for rigidbodies that do not affect gameplay directly and only exist to enrich the physical experience of the game.

If physics prediction is disabled, these local objects can be safely simulated. However, when prediction is enabled, Netick steps *all* rigidbodies (due to how PhysX and Box2D work in Unity, it's outside Netick's control) — including non-networked ones — during each predicted tick. This leads to local rigidbodies being overstepped and behaving incorrectly, as they are not reset due to lack of rollback (only networked rigidbodies are rolled back).

To address this, Netick automatically tracks and restores the state of local rigidbodies before and after resimulation. However, **when applying forces to non-networked rigidbodies, ensure this is done in `NetworkFixedUpdate` (not Unity’s `FixedUpdate`)**. This guarantees compatibility with Netick’s internal tracking and avoids force loss or simulation issues.
