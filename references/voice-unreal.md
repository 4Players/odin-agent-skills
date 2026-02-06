
# ODIN Voice Unreal SDK

Unreal Engine plugin for real-time voice chat with full Blueprint and C++ support.

## Version Compatibility

> **ODIN Plugin 1.x and 2.x are not interoperable.** Clients using SDK 1.x cannot communicate with clients using SDK 2.x. All participants in a room must use the same major version.

## SDK Versions

| Version | Status               | Documentation                    |
| ------- | -------------------- | -------------------------------- |
| **2.x** | Current, Recommended | [voice-unreal-v2.md](voice-unreal-v2.md) |
| **1.x** | Legacy, Maintenance  | [voice-unreal-v1.md](voice-unreal-v1.md) |

### Which Version Should I Use?

- **New projects**: Use **2.x** - cleaner API, encoder/decoder pattern, audio pipeline effects, 64-channel support
- **Existing 1.x projects**: Continue with 1.x for stability, plan migration to 2.x for new features
- **Interoperating with existing deployments**: Match the version used by other clients in the room

## Quick Comparison

| Feature                | 1.x                                          | 2.x                                         |
| ---------------------- | -------------------------------------------- | ------------------------------------------- |
| Audio Handling         | Media objects (`UOdinPlaybackMedia`)         | Encoder/Decoder pattern                     |
| Room Join              | Latent `JoinRoom` with callbacks             | Sync `ConnectRoom()` + events               |
| Audio Setup            | `OnMediaAdded` > `Odin_AssignSynthToMedia()` | `OnPeerJoined` > `RegisterDecoder()`        |
| APM Configuration      | Room-level during construction               | Per-encoder `UOdinPipeline` effects         |
| Key Events             | `OnMediaAdded`, `OnMediaRemoved`             | `OnRoomPeerJoined`, `OnRoomJoined`          |
| Spatial Audio Channels | Basic positioning                            | 64-channel support with channel masks       |
| E2E Encryption         | No Encryption                                      | Full support with `OdinCipher`              |

## Common Features (All Versions)

- **Full Blueprint support**: Visual scripting nodes for all functionality
- **C++ integration**: Direct access to all classes and functions
- **Cross-platform**: Windows, Linux, macOS, Android, iOS
- **Spatial audio**: 3D positioning with Unreal's audio engine
- **Audio processing**: VAD, echo cancellation, noise suppression
- **Audio middleware**: FMOD and Wwise adapter plugins available

## System Requirements

- Unreal Engine 4.26 or later vor v1.x (including UE5.x)
- Unreal Engine 5.3 or later vor v2.x
- Audio Capture Plugin (Epic's official plugin)

## Installation

**Manual Installation (Recommended)**
1. Download plugin from [GitHub Releases](https://github.com/4Players/odin-sdk-unreal/releases)
2. Extract to your project's `Plugins` folder
3. Enable the plugin in Unreal Editor

**Repository Cloning**
```bash
git clone https://github.com/4Players/odin-sdk-unreal.git
cd odin-sdk-unreal
git lfs fetch
git lfs checkout
```

**Fab Marketplace**: One-click installation through Unreal's Fab Marketplace (slower release cycle)

> **Blueprint-Only Projects**: May encounter packaging errors. Install plugin in Engine's Marketplace directory or convert to C++ project.

## Security Note

> **CRITICAL**: Only embed access keys in client code for testing purposes, __NEVER__ in production code. Generate room tokens on a server for production deployments.

## Documentation

- **2.x Full Documentation**: [voice-unreal-v2.md](voice-unreal-v2.md)
- **1.x Documentation**: [voice-unreal-v1.md](voice-unreal-v1.md)
- **Troubleshooting**: [voice-unreal-troubleshooting.md](voice-unreal-troubleshooting.md)
- **Online Docs**: https://docs.4players.io/voice/unreal/
- **Blueprint Reference**: https://docs.4players.io/voice/unreal/blueprint-reference/
- **GitHub**: https://github.com/4Players/odin-sdk-unreal
- **Discord**: [`#odin-unreal` channel](https://4np.de/discord)

