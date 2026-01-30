---
name: odin-voice-core
description: |
  ODIN Voice Core SDK (C/C++) - the low-level foundation for all ODIN SDKs.
  Use when: building custom platform integrations, working with C/C++ directly,
  understanding the underlying ODIN architecture, or creating bindings for other languages.
  Enables seamless integration of real-time voice chat into multiplayer games and applications.
  This is the base SDK that Unity, Unreal, Web, and Swift SDKs are built upon.
license: MIT
---

# ODIN Voice Core SDK (C/C++)

Low-level C API foundation for all ODIN Voice SDKs, enabling real-time voice chat integration.

## Version Compatibility

> **ODIN 1.x and 2.x are not interoperable.** Clients using SDK 1.x cannot communicate with clients using SDK 2.x. All participants in a room must use the same major version.

## SDK Versions

| Version | Status               | Documentation                    |
| ------- | -------------------- | -------------------------------- |
| **2.x** | Current, Recommended | [versions/v2.md](versions/v2.md) |
| **1.x** | Legacy, Maintenance  | [versions/v1.md](versions/v1.md) |

### Which Version Should I Use?

- **New projects**: Use **2.x** - it has a cleaner API, implicit connection pooling, and better audio pipeline controls
- **Existing 1.x projects**: Continue with 1.x for stability, plan migration to 2.x for new features
- **Interoperating with existing deployments**: Match the version used by other clients in the room

## Quick Comparison

| Feature         | 1.x                      | 2.x                                    |
| --------------- | ------------------------ | -------------------------------------- |
| Connection Pool | Explicit management      | Implicit (automatic)                   |
| Event Format    | MessagePack-RPC          | JSON                                   |
| Audio Handling  | Media streams            | Encoder/Decoder pattern                |
| Room Creation   | Separate create + join   | Combined `odin_room_create()`          |
| Audio Pipeline  | Room-level APM config    | Per-encoder/decoder pipeline effects   |
| E2E Encryption  | Limited                  | Full support with `odin_crypto.h`      |
| Spatial Audio   | Basic                    | 64-channel support with 3D positioning |

## Common Features (All Versions)

- **Cross-platform**: Windows, Linux, macOS, Android, iOS (consoles on request)
- **Built with Rust**: Stable C API for FFI compatibility
- **Opus codec**: High-quality, low-latency audio compression
- **JWT authentication**: Ed25519-signed tokens
- **Audio processing**: VAD, echo cancellation, noise suppression
- **Proximity chat**: Spatial audio with position-based filtering

## Repository

**GitHub**: https://github.com/4Players/odin-sdk

```bash
git clone https://github.com/4Players/odin-sdk.git
cd odin-sdk
git lfs fetch
git lfs checkout
```

## Documentation

- **2.x Full Documentation**: [versions/v2.md](versions/v2.md)
- **1.x Documentation**: [versions/v1.md](versions/v1.md)
- **Online Docs**: https://docs.4players.io/voice/core/
- **Platform SDKs**: Unity, Unreal, Web, Swift, NodeJS (built on this Core SDK)
