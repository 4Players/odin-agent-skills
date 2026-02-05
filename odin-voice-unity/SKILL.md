---
name: odin-voice-unity
description: |
  ODIN Voice Unity SDK for real-time voice chat in Unity games and XR experiences.
  Supports both 1.x (stable, Unity 2019.4+) and 2.x (beta, Unity 2021.4+) versions.
  Use when: implementing voice chat in Unity projects, handling peer/media events,
  configuring 2D/3D spatial audio, or integrating microphone input.
  Check versions/ for v1.x vs v2.x specific documentation.
  For ODIN concepts, see odin-fundamentals skill.
license: MIT
---

# ODIN Voice Unity SDK

Unity SDK for real-time 3D positional voice chat integration.

## Version Compatibility

> **ODIN Plugin 1.x and 2.x are not interoperable.** Clients using SDK 1.x cannot communicate with clients using SDK 2.x. All participants in a room must use the same major version.

## SDK Versions

| Version | Status               | Documentation                    |
| ------- | -------------------- | -------------------------------- |
| **1.x** | Current, Stable      | [versions/v1.md](versions/v1.md) |
| **2.x** | Pre-release Beta     | [versions/v2.md](versions/v2.md) |

### Which Version Should I Use?

- **New projects with Unity 2019.4+**: Use **1.x** for production stability
- **New projects testing beta features**: Use **2.x** (requires Unity 2021.4+, beta access)
- **Existing 1.x projects**: Continue with 1.x for stability
- **Interoperating with existing deployments**: Match the version used by other clients in the room

## Quick Comparison

| Feature              | 1.x                                   | 2.x                                  |
| -------------------- | ------------------------------------- | ------------------------------------ |
| **Main Component**   | `OdinHandler` (singleton)             | `OdinRoom` (non-singleton)           |
| **Architecture**     | Singleton pattern, DontDestroyOnLoad  | Component-based, multi-room support  |
| **Room Join**        | `OdinHandler.Instance.JoinRoom()`     | `odinRoom.Token = token;`            |
| **Playback**         | Manual `AddPlaybackComponent()`       | Auto-created `OdinPeer`/`OdinMedia`  |
| **Events**           | `OnCreatedMediaObject`                | `OnPeerJoined`, `OnMediaAdded`       |
| **Audio Pipeline**   | Room-level configuration              | Per-encoder/decoder effects          |
| **Unity Version**    | 2019.4+                               | 2021.4+                              |
| **Status**           | Stable release                        | Pre-release beta                     |

## Common Features (All Versions)

- **Full C# integration**: Native Unity components and events
- **Cross-platform**: Windows, Linux, macOS, Android, iOS
- **Spatial audio**: 3D positioning with Unity's audio engine
- **Audio processing**: VAD, echo cancellation, noise suppression
- **Sample projects**: Ready-to-use examples included

## System Requirements

- **1.x SDK**: Unity 2019.4 or later
- **2.x SDK**: Unity 2021.4 or later

## Installation

### Version 1.x (Stable)

```
Window > Package Manager > + > Add package from git URL
URL: https://github.com/4Players/odin-sdk-unity.git
```

Or download `.unitypackage` from [GitHub releases](https://github.com/4Players/odin-sdk-unity/releases).

### Version 2.x (Beta)

1. Request ZIP file from 4Players (beta access required)
2. Extract ZIP to local folder
3. Package Manager > + > "Add package from disk"
4. Select `package.json` from extracted folder

> **Important**: Remove previous versions before upgrading. Unity does not remove old files automatically.

## Security Note

> **CRITICAL**: Only embed access keys in client code for testing purposes, **NEVER** in production code. Generate room tokens on a server for production deployments.

## Documentation

- **1.x Full Documentation**: [versions/v1.md](versions/v1.md)
- **2.x Full Documentation**: [versions/v2.md](versions/v2.md)
- **Troubleshooting**: [references/troubleshooting.md](references/troubleshooting.md)
- **Online Docs (1.x)**: https://docs.4players.io/voice/unity/
- **Online Docs (2.x)**: https://docs.4players.io/voice/unity/next/
- **GitHub**: https://github.com/4Players/odin-sdk-unity
- **Discord**: [`#odin-unity` channel](https://4np.de/discord)
