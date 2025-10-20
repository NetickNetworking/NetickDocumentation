# Full Game Replay

---

## File ReplayTransport

The replay system in Netick allows for recording the *entire networked state* of a game (including all network objects, RPCs, and scene state) on the server. This data can later be used to replay the full game exactly as it happened.

Unlike replay systems in games such as Counter-Strike, which store raw network packets and must process every packet sequentially to reach a target frame (making seeking very slow), Netick records snapshots of the full game state.
This approach is conceptually similar to video compression formats, allowing for instant seeking without needing to apply all previous data.

### Use Cases

* Game Analysis: Let players review full matches and analyze them.
* Cheat Review: Let admins investigate potential cheating (e.g., wallhacks, aimbots) by replaying suspicious matches.
* Debugging: Simulate late-joins or extended packet loss by skipping through the replay timeline.
* Videos: Both developers and players can screen record replays to create videos for various purposes. Unlike regular screen recording alone, replays are actual replays of the game, so it's possible to change the camera location and capture things not possible with regular screen recording alone.

### Overview

Recording in Netick’s replay system is **entirely server-side**.
To replay a recording, Netick must be started in a special mode called **Replay Client**.

A Replay Client behaves like a normal client, with a few key differences:

* It’s not connected to a live server, the replay snapshots are applied directly instead of receiving live network packets.
* The prediction loop is disabled.
* The local player id `Sandbox.LocalPlayer` is fixed to `NetworkPlayerId.ReplayPlayer`. This ensures RPCs sent to that id during recording will still exist when replaying.
* The local player does not appear in `Sandbox.Players`, as it’s only a dummy player that was never in the game when it was recorded.

Replays recorded on older versions of the game are incompatible and can't be replayed on newer versions. This limitation exists on all replay systems. While it's rarely done, one way to deal with this is to keep older builds of the game and use them to play older replays. However, Netick does not handle any of this and is up to the developer if needed.

---

## Replay-Safety

In most cases, achieving replay safety (making a project fully compatible with Netick’s Replay System) is straightforward. 
During replay playback, the game logic operates on recorded snapshots rather than live network packets. To handle any special cases, you can check whether the current session is a replay using:

```cs
if (Sandbox.IsReplay)
{
    // Logic specific to replay mode
}
```

This allows you to adjust or skip code that should not run during replay.

> [!Note]
> Viewing the game from the perspective of different players is not handled automatically. Developers must implement support for this functionality as needed.

## Replay API

The Replay System consists of two primary components:

* **Recording API** (server-side only)
* **Playback API** (for replay clients)

---

### Recording

Recording is only supported on the **server**.

**Start recording:**

```csharp
Sandbox.StartRecording(replayPath);
```

**Stop recording:**

```csharp
Sandbox.StopRecording();
```

If no path is provided, Netick automatically records to a file (named with the current date and time) inside:

```csharp
Path.Combine(Application.persistentDataPath, "replays", Network.GameVersion.ToString());
```

**Example (Windows path):**

```
C:\Users\<username>\AppData\LocalLow\<CompanyName>\<ProjectName>\replays\<gameVersionHash>\<replayFileName>
```

#### Replay Metadata

Including metadata in a replay file is often useful for capturing additional game-specific information, such as round start and end times, special events, or custom markers.

Netick provides a simple API for this purpose:

```cs
Sandbox.Replay.Record.SetReplayMetadata(metaDataByteArray);
```

The data here can be in a JSON format, for instance. When collecting this metadata, you can use `Sandbox.Replay.Record.FrameIndex` to determine the current replay frame being recorded. This allows you to associate metadata precisely with a specific moment in the replay timeline.

---

### Playback

#### Starting a Replay

To play a replay file, start Netick in **Replay Client** mode:

```csharp
var sandbox = Netick.Unity.Network.StartAsReplayClient();
```

Then begin playback:

```csharp
sandbox.StartPlayback(replayPath);
```

If no path is specified (`sandbox.StartPlayback()`), Netick automatically loads the **most recent replay file** from:

```
C:\Users\<username>\AppData\LocalLow\<CompanyName>\<ProjectName>\replays\<gameVersionHash>\
```

#### Replay Metadata

To get the replay metadata during playback:

```csharp
Sandbox.Replay.Playback.TryGetReplayMetadata(out byte[] data);
```

#### Validating Replay Files

Always check replay file info before starting playback:

```csharp
var replayFileInfo   = await FileReplayTransport.GetReplayFileInfoAsync(path);
var replayFileStatus = replayFileInfo.Status;
```

This asynchronous method returns a `ReplayFileInfo` struct, with a `Status` variable indicating the status of the replay file.

```csharp
public enum ReplayFileStatus
{
    Ok,
    NotFound,
    Invalid,
    VersionMismatch,
}
```

You can use this result to decide how to handle missing, corrupted, or incompatible replay files.

#### Version Compatibility

Netick prevents replaying files recorded with mismatched versions.
Replay validation checks the value of ` Netick.Unity.Network.GameVersion`, which is a hash of:

* **Netick version:** ` Netick.Unity.Network.Version`
* **Game version:** `Application.version` (configured in *Project Settings → Player*)

If either version changes, existing replay files become incompatible.

---

### Playback Control

Once playback begins with `Sandbox.StartPlayback(replayPath)`, you can control playback via the **Playback API**:

| Action                                  | API                                           |
| ----------------------------            | --------------------------------------------- |
| Seek / Skip (by time)                   | `Sandbox.Replay.Playback.SeekToTime(time);`   |
| Seek / Skip (by frame)                  | `Sandbox.Replay.Playback.SeekToFrame(frame);` |
| Get total duration (seconds)            | `Sandbox.Replay.Playback.Duration;`           |
| Get total frame count                   | `Sandbox.Replay.Playback.TotalFrames;`        |
| Set / Get time scale                    | `Time.timeScale;`                             |

Slowing down or speeding up the playback is done by simply increasing/decreasing `Time.timeScale`. Pausing the playback is accomplished by setting `Time.timeScale` to `0`.

`Sandbox.Replay.Playback.OnSeeked` event can be used for clean up or similar actions when seeking.

#### Replay Timeline UI

Netick includes a built-in helper script, `ReplayTimeline.cs`, that provides a timeline UI for replay playback.

> [!Note]
> In the replay system, the term “Frame” refers to snapshots of the game, not rendering frames. These replay frames correspond to ticks in the simulation, but are labeled as frames within the context of replays.


---

## Replay Compression

By default, Netick applies delta compression to replay snapshots. This allows for instant recording and playback with minimal processing overhead.

Despite that, developers can apply additional compression using generic algorithms (e.g., zlib, LZ4, Brotli). This can significantly reduce the size of replay files. However, it adds processing delay before being able to start playback when decompressing the replay file, and after recording is finished when compressing the file. Because delta compression alone already does a reasonable job at compressing the data, Netick does not internally do generic compression on the replay file. However, if needed, doing it is straightforward.
