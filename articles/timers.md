# Timers

---

Netick provides several ways to work with time in your networked game.

## Converting Between Ticks and Time

Sometimes you need to convert between ticks and time (in seconds). Netick provides methods to do this conversion:

### Sandbox.TickToTime()

The `TickToTime()` method converts ticks to time in seconds. This is useful when you need to check if a certain amount of time has passed since an event occurred. For example, if you want to explode a bomb after a delay:

```csharp
public override void NetworkFixedUpdate()
{
    if (Sandbox.TickToTime(Sandbox.Tick - Object.SpawnTick) >= ExplosionDelay)
        Explode();
}
```

In this example, `Sandbox.Tick` is the current tick, `Object.SpawnTick` is the tick when the object was spawned, and `ExplosionDelay` is the delay in seconds before the explosion. The difference between the current tick and spawn tick determines the number of ticks that have passed since the object was spawned, which is then converted to seconds using `TickToTime()`.

### Sandbox.TimeToTick()

The `TimeToTick()` method does the opposite conversion, from time in seconds to ticks:

```csharp
// Convert time in seconds to ticks
Tick tickValue = Sandbox.TimeToTick(timeInSeconds);
```

## Using NetworkTimer

Netick provides a `NetworkTimer` class that makes it easier to work with time-based events in your networked game. The `NetworkTimer` class offers methods to check if a timer is running, if it has stopped, and how much time remains.

### Starting a Timer

You can start a timer using the `Sandbox.StartTimer()` method:

```csharp
// Start a 5-seconds timer
NetworkTimer myNetworkTimer = Sandbox.StartTimer(5.0f);
```

> [!Note]
> By default, the system uses predicted timing. You can use `usePredictedTiming: false` to opt out of this feature: `Sandbox.StartTimer(duration, usePredictedTiming: false)`.

### Checking Timer Status

You can check the status of a timer using the following methods:

```csharp
// Check if the timer is still running
if (myNetworkTimer.IsRunning(Sandbox))
{
    // Timer is still running
}

// Check if the timer has stopped
if (myNetworkTimer.IsStopped(Sandbox))
{
    // Timer has stopped/finished
}
```

### Getting Remaining and Elapsed Time

You can get the remaining time and elapsed time of a timer:

```csharp
// Get the remaining time in seconds
float remainingTime = myNetworkTimer.GetRemainingTime(Sandbox);

// Get the elapsed time in seconds
float elapsedTime = myNetworkTimer.GetElapsedTime(Sandbox);
```