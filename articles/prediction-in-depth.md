# Prediction in-depth

Before diving in, let's do a recap on what prediction means.

Prediction is the act of the client to predict what the game (the networked state of the game) looks like in the server, starting from the latest received server snapshot as a baseline. The client uses its local inputs that have yet to be acknowledged and processed by the server to simulate up to the time difference between it and the server, so that the client sees the same state as the server, at almost the same time. From the perspective of the client, prediction means predicting the future state of the server, since to the client, due to RTT, the server is in the future.

However, for many types of games, namely First Person Shooters, we only predict one or a few more objects, notably our own local player character.

When we do this, our local player character object will live in the local (predicted) timeline. While remote objects (including remote players) will live in the remote timeline. What do we mean by these terms?

- Remote Timeline: Remote Timeline refers to the delayed (by half RTT) timeline we see proxy objects in the client. A proxy object is any object that the client is not providing inputs to (not an Input Source of), and is fully controlled by the server, and the client merely observes the incoming server state snapshots for it. Because the server data is delayed, what we see for these objects is delayed by `half RTT + interpolation delay`.

- Local/Predicted Timeline: Local/Predicted refers to the timeline predicted objects in the client live in. The local timeline differs to the remote timeline by being ahead of the remote timeline by `RTT + additional buffering` (due to adaptation to non-ideal network conditions). 

What we understand here is that there is a discrepancy. Some objects will be in the local timeline, and others will be in the remote timeline. The remote timeline is out-of-sync with the local timeline. Even though this might seem bad, but that's how almost every First Person Shooter works. The local player in an FPS game is in the predicted timeline, while other players (remote players) are in the remote timeline. Why is that though? Why not put all objects in the predicted timeline for an FPS game?

The problem with this is that, usually, the acceleration speeds of an FPS character are too fast that the prediction will always be wrong, and it results in a poor gameplay experience. You will see a player come out of a corner and suddenly disappear, due to mis-predictions.

An important fact to emphasize is that it's impossible for the client to correctly predict remote/proxy objects. It will keep mis-predicting. This is because it's not possible for the client to predict the input of other clients, due to latency.

But, does this mean remote prediction is bad? No. For many types of games, remote prediction offers a better gameplay experience than putting every other object (except the local player) in the remote timeline. Notably anytime you want to do comprehensive interactions (mostly physical interactions) between players and the environment, remote prediction will result in a better experience. This is because, without prediction, if you were to collide with a remote object, you will only see the effect of the collision by RTT time. Physics-based games, fighting games, and racing games are all examples of games where predicting remote objects is a better strategy.

By default, Netick only predicts objects that the client is the Input Source of. To understand what it means to predict remote/proxy objects, let's explore an example.

<figure><img src="../../images/proxy-prediction.png" alt="Client-Side Prediction"><figcaption></figcaption></figure>

In Rocket Cars, we not only predict the local car, but also the other (remote) cars and the ball. Now, let's see what that means, and also let's see what happens when we don't do that.

Look at the previous image. In this scenario, we assume that each car starts moving at the exact same time. Let's also assume that the cars were moving at the same exact speed for some time, and we are looking at what they look like after that amount of time. In addition, let's say that the ball was also moving in the same direction as the cars. Therefore, all the objects in this scenario are moving, and at the same direction.

In the right-side figure, we see that all cars are aligned with each other, which is what we expect if they started moving at the exact same time and at the exact same speed. Everything looks correct. This is because the clients are not changing their inputs, so the input prediction is correct. But this is usually not the case in practice. However, this shows that prediction will converge to the correct state if the clients are not changing their inputs too much.

Now, let's see what the game looks like if we didn't predict remote/proxy objects. Let's look at the left-side figure above. What we see here is that, now, only our local car is in the predicted position. Other cars are, to us, delayed. The gray ghost shapes show where the cars should be, if they were to be predicted. The difference in position here is the amount of positional discrepancy between the local/predicted timeline against the remote timeline, which is proportional to RTT/latency.

So, the conclusion here is that neither approach is perfect. Not predicting remote objects will result in delayed collisions. Predicting them will result in mis-predictions.

But, for this game, the better approach is to predict remote objects. Therefore, it's a matter of choosing which approach works better for a particular game.

## Predicting Remote/Proxy Objects

Now, let's see how we can actually predict remote objects, in practice. Using Netick, this is quite simple. Simply change the `Prediction Mode` of an object to be equal to `Everyone` in the inspector. This will cause the `NetworkFixedUpdate` of this object to execute multiple times due to resimulation.

However, this is not all. To be able to predict the input of other players, we need to sync their inputs. The following code snippet is taken from Rocket Cars.

```cs
  [Networked] public GameInput   LastInput  { get; set; } // We sync the last input for the player. So we can use it to predict remote players cars.
  public override void NetworkFixedUpdate()
  {
    if (FetchInput(out GameInput input))
      LastInput          = input;

    // clamp movement inputs
    input.Movement       = new Vector3(Mathf.Clamp(input.Movement.x, -1f, 1f), Mathf.Clamp(input.Movement.y, -1f, 1f), Mathf.Clamp(input.Movement.z, -1f, 1f));
    SimulateVehicle(input);
  }
```

That's all there is to it. `FetchInput` only returns true on the Input Source itself, and the server. So, by simply defining a network property to store the input in, we are able to sync the input to everyone, including observing (proxies) players.

Notice that we don't actually try to predict the input, we simply use the last input for prediction. Because predicting that the client pressed something it never did is a lot worse than simply assuming the client is still pressing the same buttons.

Rocket Cars serves as an excellent example of how Proxy/Remote Prediction works.

## Prediction Error Correction Smoothing

By default, correcting mis-predictions is instantaneous. This will cause the predicted remote objects to snap somewhere else when a player changes their movement direction suddenly. And as we said, the magnitude of mis-predictions is proportional to latency. Therefore, for a smooth visual experience. we must smooth out the prediction correction. Netick impalements a smooth correcter in `NetworkTransfrom`/`NetworkRigidbody`. By enabling it, it will smooth out the corrections over multiple frames. There are a few settings for it which will need to be fine-tuned to find what is best for your object.