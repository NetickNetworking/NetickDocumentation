# Prediction In-Depth

Let's start this by doing a recap on what prediction means.

Prediction is when the client tries to guess what the game (the networked state of the game) looks like in the server, starting from the latest received server world snapshot as a starting point. The client uses its local inputs (that have yet to be acknowledged and processed by the server) to simulate forward up to the latency (ping/RTT) between it and the server. Therefore, the client runs ahead of the server time, so that the inputs of the client arrive just when they are needed in the server. Put differently, from the perspective of the client, prediction means predicting the future state of the server world.

However, for many types of games, namely First Person Shooters, it's common to only predict one (and few more) objects, notably the local player character.

When we do this, our local player character object will live in the local (predicted) timeline. While remote objects (including remote players) will live in the remote timeline. What do we mean by these terms?

- Remote Timeline: Remote Timeline refers to the old or delayed timeline we see proxy objects in the client. It's delayed because of ping/latency, incoming data from the server takes a bit of time to arrive. A proxy object is any object that the client is not providing inputs to (not an Input Source of), and the client merely observes the incoming server state snapshots for it. Because the server data is delayed/old (due to latency), what we see for these objects is delayed by `half RTT + interpolation delay`.

- Local/Predicted Timeline: Local/Predicted Timeline refers to the timeline predicted objects in the client live in. The local timeline differs to the remote timeline by being ahead of the remote timeline by `RTT + additional buffering (due to adaptation to non-ideal network conditions)`. 

Therefore, there is a discrepancy between the two timelines. Some objects will be in the local timeline, and others will be in the remote timeline. The remote timeline is out-of-sync with the local timeline, which gets worse with ping. Even though this might seem bad, but that's how almost every First Person Shooter works. The local player in an FPS game is in the predicted timeline, while other players (remote players) are in the remote timeline. Why is that though? Why not put all objects in the predicted timeline for an FPS game?

The problem with this is that, usually, the acceleration rate of an FPS character is so fast that the prediction will always be wrong, and it results in a poor gameplay experience. You will see a player come out of a corner and suddenly disappear, due to mispredictions and their subsequent corrections. In addition, server-authoritative bullet hit-detection is tricky on predicted remote objects. Since, due to mispredictions, missed shots will be common. In contrast, [lag-compensation](lag-compensation.md), which is a technique that only works with the remote timeline, allows for perfect hit-detection.

An important fact to emphasize is that it's impossible for the client to accurately predict the inputs of other clients. However, players usually don't drastically change their inputs from one moment to the other, which is a good thing.

But, does this mean remote prediction is to be avoided? No. For many types of games, remote prediction provides a better gameplay experience than putting every other object (except the local player) in the remote timeline. Notably anytime you want to do comprehensive interactions (mostly physical interactions) between players, remote prediction will result in a better experience. This is because, without prediction, if you were to collide with a remote object, you will only see the effect of the collision by RTT time. Physics-based games, fighting games, and racing games are all examples of games where predicting remote objects is a better strategy.

By default, Netick only predicts objects that the client is the Input Source of. To understand what it means to predict remote/proxy objects, let's explore an example.

<figure><img src="../images/proxy-prediction.png" alt="Client-Side Prediction"><figcaption></figcaption></figure>

In Rocket Cars, we not only predict the local car, but also the other (remote) cars and the ball. Now, let's see what that means, and also let's see what happens when we don't do that.

Look at the previous image. In this scenario, we assume that each car starts moving at the exact same time. Let's also assume that the cars were moving at the  exact same speed for some time, and we are looking at what they look like after that amount of time. In addition, let's say that the ball was also moving in the same direction as the cars. Therefore, all the objects in this scenario are moving, and at the same direction.

In the left-side figure, we see that all cars are aligned with each other, which is what we expect if they started moving at the exact same time and at the exact same speed. Everything looks correct. This is because the clients are not changing their inputs, so the input prediction is correct. But this is usually not the case in practice. However, this shows that prediction will converge to the correct state if the clients are not changing their inputs too much.

Now, let's see what the game looks like if we didn't predict remote/proxy objects. Let's look at the right-side figure above. What we see here is that, now, only our local car is in the predicted position. Other cars are, to us, delayed. They are out-of-sync with the local player car. The gray ghost shapes show where the cars should be, if they were to be predicted. The difference in position here is the amount of positional discrepancy between the local/predicted timeline against the remote timeline, which is proportional to RTT/latency.

So, the conclusion here is that neither approach is perfect. Not predicting remote objects will result in delayed collisions. Predicting them will result in mispredictions. This is the reality of game-networking, there is not a one-size-fit-all solution. You choose the lesser evil.

The lesser evil for this game is to predict remote objects. Therefore, it's a matter of choosing which approach works better for a particular game.

In conclusion, let's see the pros and cons of each approach:

### Without Proxy/Remote Prediction

#### Pros
* Accurate Snapshots: the states of remote objects are correct, since they come directly from the received server snapshots.
* Lag Compensation: you can have perfect server-authoritative hit-detection using lag compensation for clients since what the clients see is what actually happened, without mispredictions.
* Low CPU Overhead: since you only simulate the local player, CPU performance will be better.

#### Cons
* Weak Player-to-Player Interactions: usually the best approach is to disable collisions between players.
* Multiple Timelines: the local player is out-of-sync with remote players, due to being in the Local Timeline, while they are in the Remote Timeline.

### With Proxy/Remote Prediction

#### Pros
* Good Player-to-Player Interactions: you can have smooth and responsive interactions between players, such as collisions. 
* Single Timeline: all objects live in the same timeline, which is the local/predicted timeline. No desync between objects.
* Simpler Code: by being able to simulate other objects and have them all in the same timeline, the coding experience will be closer to single-player development.

#### Cons
* Mispredictions: the rendered states of predicted remote objects are not necessarily states that actually happened in the server, due to mispredictions. One player can report seeing different things compared to other players, creating contradictory perspectives on what happened. Mispredictions get worse with higher pings, so clients with very high pings (+300) might have almost an unplayable experience.
* No Lag Compensation: you can't perform lag compensation on predicted objects. However, because all objects are in the same timeline, there is no need for lag compensation. But, due to mispredictions, the client hits will often miss. 
* High CPU Overhead: predicting more objects will use more CPU time, and the cost of that increases with ping and tickrate.

## Predicting Remote/Proxy Objects

Now, let's see how we can actually predict remote objects, in practice. Using Netick, this is quite simple. Simply change the `Prediction Mode` of an object to be equal to `Everyone` in the inspector. This will cause the `NetworkFixedUpdate` of this object to execute multiple times due to resimulation.

However, this is not all. To be able to predict the input of other players, we need to sync their inputs. The following code snippet is taken from Rocket Cars.

```cs
  [Networked] public GameInput   LastInput  { get; set; } // We sync the last input for the player. So we can use it to predict remote players cars.
  public override void NetworkFixedUpdate()
  {
    if (FetchInput(out GameInput input))
      LastInput          = input;

    SimulateVehicle(LastInput);
  }
```

That's all there is to it. `FetchInput` only returns true on the Input Source itself, and the server. So, by simply defining a network property to store the input in, we are able to sync the input to everyone, including observing (proxies) players.

Notice that we don't actually try to predict the input, we simply use the last input for prediction. Because predicting that the client pressed something it never did is a lot worse than simply assuming the client is still pressing the same buttons.

Rocket Cars serves as an excellent example of how Proxy/Remote Prediction works.

## Prediction Error Correction Smoothing

By default, correcting mispredictions is instantaneous. This will cause the predicted remote objects to snap somewhere else when a player changes their movement direction suddenly. And as we said, the magnitude of mispredictions is proportional to latency. Therefore, for a smooth visual experience, we must smooth out the prediction correction. Netick implements a smooth correcter in `NetworkTransfrom`/`NetworkRigidbody`. By enabling it, it will smooth out the corrections over multiple frames. There are a few settings for it which will need to be fine-tuned to find what is best for your object.

## Input Delay

A powerful technique to reduce mispredictions on remote objects is to to delay the inputs of everyone by a specific amount/ticks. This will make the game almost perfectly synced without mispredictions for players with pings roughly below the input delay. 

Example:

```cs
public float                                 InputDelay         = 100; // in milliseconds
public const int                             InputQueueCapacity = 6;
[Networked(size: InputQueueCapacity)]
public readonly NetworkQueue<MyInput>        InputQueue        = new(InputQueueCapacity);
[Networked] 
public MyInput                               LastInput         { get; private set; }

public override void NetworkFixedUpdate()
{
  if (FetchInput(out MyInput i))
  {
    if (InputQueue.Count == InputQueueCapacity)
       InputQueue.Dequeue();
     InputQueue.Enqueue(i);
   }

  int inputDelayInTicks = (int)Mathf.Round((InputDelay / 1000f) / Sandbox.FixedDeltaTime);

  if (InputQueue.Count > 0 && (Sandbox.Tick - InputQueue.Peek().Tick >= inputDelayInTicks))
     LastInput = InputQueue.Dequeue();

  // logic
  Move(LastInput);
}
```

Note that we've added a field called `Tick` to the input struct, which we assign it the value of `Sandbox.Tick` when setting the input struct fields.