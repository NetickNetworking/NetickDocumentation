# Physics Prediction (Unity)

Predicting physics means resimulating multiple physics steps in one tick. This can be very expensive and so by default physics prediction is turned off. To enable it, go to `Netick -> Settings` and enable `Physics Prediction`. 

To make a `Rigidbody`/`Rigidbody2D` predictable, add `NetworkRigidbody`/`NetworkRigidbody2D` to its GameObject. 

To enable/disable Physics Prediction in the client at runtime, use `Sandbox.PhysicsPrediction`.

## Cost of Predicting PhysX (Rigidbody3D)

It's very expensive to predict 3D physics as PhysX and its integration with Unity perform very badly when calling `PhysicsScene.Simulate` multiple times in one frame, even with small numbers of rigidbodies.

The cost of predicting physics increases with two factors:

- Ping
- Tickrate

As ping increases, you would need to simulate more ticks in one frame for rollback and resimulation. As tickrate increases, the time period between each tick becomes smaller, therefore you will need to simulate more ticks for smaller values of latency.

It's very much not recommended to enable 3D physics prediction, as it's almost impractical on some machines on relatively high tickrates (+33). It can take more than 10ms on some machines on 100ms ping just to resimulate a bunch of physics ticks.

If you don't want to predict physics, you have two other alternatives:

- Not predicting physics at all, and make it server-authoritative completely.
- Make your physics objects client-authoritative by sending the resultant states as RPCs or network inputs.