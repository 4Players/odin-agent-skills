---
name: odin-voice
description: |
  ODIN Voice SDK overview and concepts for real-time voice chat integration in games, apps, and websites.
  Use when: understanding ODIN architecture, authentication flow (access keys, tokens), room concepts,
  event lifecycle, use cases (multiplayer games, MMOs, VR/AR, virtual conferences), or choosing an SDK.
  For SDK-specific implementation, use the dedicated skills: odin-voice-unity, odin-voice-unreal,
  odin-voice-web, odin-voice-swift, or odin-voice-core.
---

# ODIN Voice SDK

Real-time voice chat SDK for games, apps, and websites by 4Players.

## Core Concepts

### Architecture
ODIN is a **client-server** voice communication system:
- 4Players hosts servers, handles networking and audio processing
- Pure client integration (no server-side code required from developers)
- Stateless, scalable, cross-platform (Windows, macOS, Linux, iOS, Android, Web)

### Key Terminology

| Term | Description |
|------|-------------|
| **Access Key** | 44-char Base64 authentication credential. Never embed in client code. |
| **Room Token** | Signed JWT generated from access key. Given to clients for authentication. |
| **Room** | Communication channel. Auto-created on first join, auto-deleted when empty. |
| **Peer** | A client connected to a room. Each client is a different peer in every room. |
| **Media** | Audio stream linked to physical device (microphone). |

### Connection Flow
```
1. Backend generates Room Token (from Access Key)
2. Client receives token
3. Client joins Room (becomes a Peer)
4. Client adds Media (microphone stream)
5. Media routed to all other Peers in room
```

## Event Lifecycle

### Room Events
| Event | Description |
|-------|-------------|
| `OnRoomJoin/Joined` | User started/completed joining room |
| `OnRoomLeave/Left` | User started/completed leaving room |
| `OnPeerJoined` | Another peer joined the room |
| `OnPeerLeft` | Another peer left the room |
| `OnPeerUpdated` | Another peer's user data changed |

### Media Events
| Event | Description |
|-------|-------------|
| `OnMediaAdded/Started` | Peer added audio stream to room |
| `OnMediaRemoved/Stopped` | Peer stopped audio stream |
| `OnMessageReceived` | Received arbitrary data/message |

## Authentication

### Token Generation (Server-Side Only)
```javascript
// Node.js example using @4players/odin-tokens
const { TokenGenerator } = require("@4players/odin-tokens");

const generator = new TokenGenerator("__YOUR_ACCESS_KEY__");
const token = generator.createToken("RoomName", "UserId");
// Return token to client
```

**Important**: Generate tokens on your backend. Never expose access keys in client code.

### Free Tier
- Up to 25 concurrent users
- No registration required
- Full SDK access

## 2D vs 3D Voice Chat

| Type | spatialBlend | Use Case |
|------|--------------|----------|
| **2D (Non-Spatial)** | 0.0 | Phone calls, radio, team chat - same volume for all |
| **3D (Spatial)** | 1.0 | Proximity voice - volume varies by distance/direction |

## Common Use Cases

### Multiplayer Shooter
```
Room: "A_Team_A"  → Team A private chat (2D)
Room: "A_World"   → World audio, 3D positional
```
```csharp
// Join team chat + world audio
OdinHandler.Instance.JoinRoom("A_Team_A");
OdinHandler.Instance.JoinRoom("A_World");
```

### Radio Channels (MMO)
```csharp
void OnRadioChannelChanged(int channel) {
    if (currentRadio != null)
        OdinHandler.Instance.LeaveRoom(currentRadio);
    currentRadio = $"Radio_Channel_{channel}";
    OdinHandler.Instance.JoinRoom(currentRadio);
}
```

### Direct Chat
```csharp
void CallPlayer(Player other) {
    string roomName = $"DirectChat_{this.Id}_{other.Id}";
    OdinHandler.Instance.JoinRoom(roomName);
    other.SendInvite(roomName);
}
```

### Open World (3D Proximity)
- All players in same room
- ODIN servers know positions, route audio only to nearby players
- Set `spatialBlend = 1.0` on AudioSource

## Available SDKs

| SDK | Platform | Skill |
|-----|----------|-------|
| **Unity** | Unity 2021.4+ | `odin-voice-unity` |
| **Unreal** | UE4.26+, UE5.x | `odin-voice-unreal` |
| **Web** | Browser (NPM/CDN) | `odin-voice-web` |
| **Swift** | iOS 9+, macOS 10.15+ | `odin-voice-swift` |
| **Core** | C/C++ (foundation) | `odin-voice-core` |

## Quick Reference

### Joining a Room (Pseudocode)
```
token = GetTokenFromBackend(roomName, userId)
room.Join(token)
room.AddMedia(microphone)
// Handle OnPeerJoined, OnMediaAdded events
```

### Sending Messages
```
room.SendMessage(data)  // Broadcast to all
room.SendMessage(data, [peerId])  // To specific peer
```

### Updating Position (3D)
```
room.SetPosition(x, y, z)  // For proximity-based audio culling
```

## Documentation References

For detailed SDK implementation, see the dedicated skills or documentation:
- Unity: https://docs.4players.io/voice/unity/ or `odin-voice-unity` skill
- Unreal: https://docs.4players.io/voice/unreal/ or `odin-voice-unreal` skill
- Web: https://docs.4players.io/voice/web/ or `odin-voice-web` skill
- Swift: https://docs.4players.io/voice/swift/ or `odin-voice-swift` skill
- Core: https://docs.4players.io/voice/core/ or `odin-voice-core` skill
