# Timers

------------------------------------------------------------------------

Netick provides several ways to work with time in your networked game.

## Converting Between Ticks and Time

Sometimes you need to convert between ticks and time (in seconds).
Netick provides methods to do this conversion.

### `Sandbox.TickToTime()`

The `TickToTime()` method converts ticks to time in seconds. This is
useful when you need to check if a certain amount of time has passed
since an event occurred. For example, if you want to explode a bomb
after a delay:

``` csharp
public override void NetworkFixedUpdate()
{
    if (Sandbox.TickToTime(Sandbox.Tick - Object.SpawnTick) >= ExplosionDelay)
        Explode();
}
```

In this example, `Sandbox.Tick` is the current tick, `Object.SpawnTick`
is the tick when the object was spawned, and `ExplosionDelay` is the
delay in seconds before the explosion. The tick difference is converted
to seconds using `TickToTime()`.

### `Sandbox.TimeToTick()`

`TimeToTick()` performs the opposite conversion:

``` csharp
Tick tickValue = Sandbox.TimeToTick(timeInSeconds);
```

## Using `NetworkTimer`

`NetworkTimer` is a utility struct type for simplifying time-based
actions in a networked game. It can be
used to implement both **countdown timers** (timers with a duration that
finish) and **stopwatch timers** (timers that measure time since a start
moment).

Network Property Example:

```csharp
[Networked]
public NetworkTimer Timer { get; set; }
```

### Countdown Timers vs Stopwatch Timers

-   A **countdown timer** is created by starting a timer with a positive
    duration.
-   A **stopwatch timer** is effectively a countdown timer started with
    **0 duration** --- it never "finishes", so you only care about the
    elapsed time since it started.

Because of this:

-   `GetRemainingTime()` only makes sense for countdown timers (duration > 0).
-   `GetElapsedTime()` works for both timer types:
    -   For countdowns: it returns the time since the timer started. If
        the timer has finished, it continues increasing, giving you
        "time since it finished."
    -   For stopwatches: it returns the time since the stopwatch
        started.

### Starting a Timer

You can start a timer using `Sandbox.StartTimer()`:

``` csharp
// Start a 5-second countdown timer
CountdownTimer = Sandbox.StartTimer(5.0f);

// Start a stopwatch timer
StopwatchTimer = Sandbox.StartTimer(0.0f);
```

### Checking Timer Status

``` csharp
// Check if the timer is still running (only relevant for countdown timers)
if (CountdownTimer.IsRunning(Sandbox))
{
    // Timer has not yet reached its duration
}

if (StopwatchTimer.IsStopped(Sandbox))
{
    // Timer finished (for countdown timers)
    // For stopwatches (duration = 0), this is always true immediately
}
```

### Getting Remaining and Elapsed Time

``` csharp
// Remaining time (countdown timers only)
float remainingTime = CountdownTimer.GetRemainingTime(Sandbox);

// Elapsed time (valid for both countdown and stopwatch timers)
float elapsedTime = StopwatchTimer.GetElapsedTime(Sandbox);
```

For countdowns:

-   `GetRemainingTime` decreases until it reaches zero.
-   `GetElapsedTime` increases beyond the duration, allowing you to track
    "time since completion."

For stopwatches (duration = 0):

-   `GetRemainingTime` is always zero.
-   `GetElapsedTime` is the time since the stopwatch started.
