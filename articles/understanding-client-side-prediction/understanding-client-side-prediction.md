# Understanding Client-Side Prediction (CSP)

## Tick-based Networking

Before diving into client-side prediction, it’s essential to first understand tick-based networking.

In a multiplayer environment, clients and the server may run at vastly different framerates. To ensure consistent and synchronized gameplay across all machines, networked game logic is executed at a fixed interval known as the tickrate. This fixed simulation rate—similar to Unity’s physics fixed timestep—ensures stable and proper behavior across the network.

Each step of simulation at this interval is called a tick, representing a discrete moment in simulated time. By tying actions and updates to specific ticks, synchronizing a networked game becomes a lot simpler regardless of the various framerates each connected client runs at. 

## Client-Side Prediction (CSP)

In a client-server model, clients are not allowed to authoritatively modify the state of networked objects. Instead, the server alone owns the truth and is responsible for applying any changes. Because ultimately the client’s executable can be tampered with or modified. In other words, only the server can ever change the true state of network variables. What the client does to affect changes to the networked state is send inputs which are later simulated by the server to produce the desired state which is sent back to all clients.

This model ensures security and fairness, as the server cannot be easily tampered with. However, it also introduces an issue: input delay. Due to internet latency (round-trip time), waiting for the server to simulate input leads to a sluggish and unresponsive experience.

To solve this, modern multiplayer systems use Client-Side Prediction (CSP).

<figure><img src="../../images/tick.png" alt="Client-Side Prediction"><figcaption></figcaption></figure>

With CSP, instead of waiting for the server to simulate inputs, the client predicts the outcome locally by executing its inputs immediately. When the actual state from the server arrives:

* The client rolls back to the last known server state.
* It resimulates all stored inputs from that point forward.

All of this happens within a single tick, ensuring a smooth and immediate experience for the player while still maintaining server authority.

### Simulation Logic

All tick-based simulation must be implemented within `NetworkFixedUpdate` in a `NetworkBehavior`. This method is invoked every network tick to advance the simulation.

* On the server, `NetworkFixedUpdate` is called once per new tick to process fresh inputs.
* On the client, it may be called multiple times in a single tick to resimulate all inputs beyond the last confirmed server tick.

#### Resimulation Targets

Resimulation on the client only applies to:

* Objects for which the client is the Input Source
* Objects with `PredictionMode` set to `Everyone`, meaning all clients predict ulate them, not just the Input Source

For all other objects, `NetworkFixedUpdate` is executed once per tick, with no resimulation.

> [!NOTE]
> The server never resimulates past ticks—it only processes new inputs for the current tick. CSP is strictly a client-side mechanism. From the server’s perspective, it’s akin to running a single-player game simulation.

For basic movement, the side effects of resimulation are typically negligible. However, when handling actions like shooting, playing audio, or showing visual-only effects, it’s crucial to ensure these actions only occur the first time an input is simulated. Failing to do this can result in actions being triggered multiple times due to repeated resimulations.

Note that it’s usually impractical to predict everything the client does in the game, and it’s simpler to:

* Avoid predicting specific features entirely (e.g., entering a vehicle or buying a weapon), and wait for the server response
* Make certain features client-authoritative, especially if mispredictions would be complex to reconcile

You don’t need to make the entire game server-authoritative. Only enforce authority where it’s critical to gameplay integrity—such as competitive actions, player movement, or sensitive game state.