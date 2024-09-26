# Understanding Client-Side Prediction (CSP)

## Tick-based Networking
Before talking about Client-Side Prediction, it's important to first understand tick-based networking. 

Simply put, because each client could be running at a very different framerate from each other (and from the server), the only way to keep all of them in sync is by running the networked game logic at a fixed rate called the tickrate. Therefore, all clients and the server run at this fixed tickrate. The tickrate functions similarly to the fixed simulation rate of the physics engine in Unity, for instance. Unity runs the physics at a fixed rate for accurate and stable physics simulation, we use a fixed tickrate for accurate and proper network synchronization.

Each fixed-time step executed is called a tick, which represents a point in time in the network loop. By being able to attribute actions to specific ticks, synchronizing a networked game becomes a lot simpler regardless of the various framerates each connected client runs at. 

## Client-Side Prediction (CSP)

In the Client-Server model, to be able to change the state (values of properties/arrays) of a network object, that change must be authoritatively done on the server. This is to ensure a secure and cheat-free gameplay experience, because ultimately the client’s executable can be tampered with or modified. **Only the server can ever change the true state of network variables.** What the client does to affect changes to the networked state is send inputs which are later executed/simulated by the server to produce the desired state which is sent back to the client/s.

This is obviously not practical due to internet latency (round-trip time), as the latency increases, input delay increases. This will, without a doubt, lead to a very unpleasant and unresponsive gameplay experience. The solution to this is what’s commonly known as **Client-Side Prediction**.

<figure><img src="../../images/tick.png" alt="Client-Side Prediction"><figcaption></figcaption></figure>

Client-Side Prediction basically means that the client, instead of waiting for the server to simulate its inputs and send the resultant states to it, the client executes them locally (in other words, predicts their outcome), and when the resultant state comes in, it applies that state (rolls back to the old server state) and resimulate all saved inputs that are targeted to ticks that are newer than that received state tick. All this happens in one tick, instantly.

This ensures that the server still has the final say on the authority of the game (because, eventually, the client will overwrite its local state with whatever the server says), but at the same time allows the client to locally predict their input outcome and enjoy a lag-free experience.

All simulation code must be done inside `NetworkFixedUpdate` on `NetworkBehavior`. This method is called every network tick to step forward the simulation. **On the server, this method is only called for new inputs.** While on the client, it can and will be called several times in one network tick to resimulate all saved inputs (up to the current predicted tick) when applying the incoming server state. See the previous figure to fully understand this.\
\
On what objects do resimulations happen?

* Objects the client is the Input Source for.
* Objects which has their Prediction Mode set to Everyone, instead of Input Source. Meaning not only the client who’s the Input Source predict them, but all other clients too.

For other objects, it will only be called once for every network step/tick.\
\
**Don’t forget that the server only ever simulates new ticks, it never resimulates previous ticks/inputs. CSP is exclusive to clients. To the server, it’s just like it’s a single-player game.**

For movement code, being aware of resimulations is unimportant. However, for things like shooting and other similar events, it’s vital to make sure that they only happen when the input is being simulated for the first time ever, otherwise, you would shoot several times for one bullet on the client, due to resimulations. This hazard is important to understand and deal with.

Note that it’s usually impractical to predict everything the client does in the game, and it’s sometimes way easier to not let the client predict some stuff (due to the complexity that is associated with correcting some predictions), and wait for the server state. And for other things, simply making them client-authoritative saves a lot of headaches. You don’t have to make the game completely server-authoritative. _Only the bits which are vital to the game experience._
