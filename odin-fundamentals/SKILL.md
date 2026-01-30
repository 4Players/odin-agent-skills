---
name: odin-fundamentals
description: |
  ODIN Platform Fundamentals - Core concepts, architecture, authentication, and pricing.
  Use when: learning about ODIN platform basics, understanding access keys vs tokens,
  exploring pricing tiers, understanding room-based architecture, working with authentication,
  or getting overview of ODIN Voice/Fleet/Rooms products. Covers platform fundamentals,
  authentication workflows, security best practices, and billing models for developers
  integrating real-time voice/video communication or game server hosting.
license: MIT
---

# ODIN Platform Fundamentals

Comprehensive guide to ODIN platform architecture, authentication, pricing, and core concepts.

## Platform Overview

ODIN is a unified real-time communication and server management platform by 4Players:

### ODIN Voice

Cross-platform voice chat SDK for games, apps, and websites.

**Platforms:** Unity, Unreal Engine, C/C++, Web (JS/TS), NodeJS, Swift/iOS/macOS

**Key Features:**
- 3D Spatial Audio (automatic in Unity/Unreal)
- Noise Suppression & Echo Cancellation
- Low Latency (optimized for gaming)
- Cross-Platform (mobile, web, desktop in same room)
- Data Channels (custom game data beyond voice)

### ODIN Fleet

Game-server hosting across a global network.

**Key Features:**
- Engine-agnostic architecture
- Automatic scaling
- API and CLI for deployment automation
- Dashboard for monitoring, logging, backups

### ODIN Rooms

Browser-based, decentralized video conferencing.

**Key Features:**
- End-to-end encrypted
- GDPR compliant
- Custom branding and self-hosting options

## Core Architecture

### Room-Based Model

- **Rooms**: Virtual spaces that route voice/video packets
- **Peers**: Individual users connected to a room
- **Media Streams**: Audio/video streams added/removed dynamically

### Infrastructure

- Globally hosted by 4Players
- HTTP/3 and WebTransport
- Automatic scaling and routing
- Optional self-hosted deployment

### Audio Processing (APM)

- Echo cancellation
- Noise suppression
- Automatic gain control (AGC)
- Voice Activity Detection (VAD)
- Volume gating

## Authentication & Security

### Access Keys

Server-side credentials used to generate tokens.

- **Server-Side Only**: Never embed in client code
- **Used to Generate Tokens**: Create signed JWTs for room access
- **Managed in Dashboard**: Available at console.4players.io

**Security:**
- Store in environment variables or secret managers
- Never expose in repositories or client code
- Use separate keys for dev/staging/production

### Access Tokens

Signed JWTs used by clients to join rooms.

**Token Structure:**
- `rid`: Room identifier
- `uid`: User identifier
- `lifetime`: Optional expiration (seconds)

**Backend Token Generation (Node.js):**
```javascript
const { TokenGenerator } = require('@4players/odin-tokens');
const generator = new TokenGenerator(process.env.ODIN_ACCESS_KEY);
const token = generator.createToken('room-id', 'user-123', { lifetime: 300 });
```

**Client Usage:**
```javascript
const { token } = await fetch('/api/odin-token').then(r => r.json());
await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
```

### Gateway Options

- **EU (Default):** `https://gateway.odin.4players.io`
- **US:** `https://gateway-us.odin.4players.io`

## Pricing Summary

| Tier | Cost | CCU Limit | Best For |
|------|------|-----------|----------|
| Trial | Free | 25 | Development, testing |
| Indie | €49/mo | 500 | Small studios (<€250K/yr) |
| Pro | Pay-as-you-go | Unlimited | Production at scale |

**Voice Pricing (Pro):** €0.19–€0.29 per Peak Concurrent User based on volume.

See [references/pricing-details.md](references/pricing-details.md) for full breakdown.

## Core Concepts

### Rooms

Virtual spaces where users communicate.

**Properties:** Room ID, Peers, Media Streams, User Data, Position (3D), Connection Status

**Operations:** Join/leave, send messages, add/remove audio, update position, monitor stats

### Peers

Individual users within a room.

**Types:** LocalPeer (you), RemotePeer (others)

**Properties:** Peer ID, User ID, User Data, Media Streams, Volume

### Media Streams

Audio/video streams transmitted within rooms.

**Audio Inputs:** Microphone with APM, volume control, VAD, power level monitoring

**Audio Outputs:** Remote peer playback with per-stream volume, pause/resume

### Data Channels

Binary data transmission alongside voice/video.

**Use Cases:** Game state sync, player positions, chat messages, custom events

## Integration Workflow

1. **Obtain Access Key**: Sign up at 4players.io/odin, create app, copy key
2. **Set Up Backend**: Install `@4players/odin-tokens`, create token endpoint
3. **Install SDK**: Platform-specific package (npm, Unity Asset Store, etc.)
4. **Initialize**: Create room, register events, fetch token, join
5. **Configure Audio**: Set output device, create input, add to room
6. **Handle Events**: Peer joined/left, audio start/stop, messages
7. **Cleanup**: Remove inputs, leave room, release resources

### Minimal Example

```javascript
// Backend
app.get('/api/odin-token', (req, res) => {
    const generator = new TokenGenerator(process.env.ODIN_ACCESS_KEY);
    res.json({ token: generator.createToken('room-id', req.user.id) });
});

// Client
const { token } = await fetch('/api/odin-token').then(r => r.json());
const room = new ODIN.Room();
room.onPeerJoined = (p) => console.log('Peer joined:', p.peer.userId);
await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
```

## Best Practices

### Security
- Never expose Access Keys in client code
- Always generate tokens server-side
- Use HTTPS for token exchanges
- Set appropriate token lifetimes (5-15 min)

### Performance
- Register events before joining
- Implement reconnection with exponential backoff
- Release resources properly

### User Experience
- Allow device selection
- Show connection status and peer list
- Implement mute/unmute and volume controls
- Show voice activity indicators

## Additional Documentation

- **Pricing Details**: See [references/pricing-details.md](references/pricing-details.md)
- **Integration Patterns**: See [references/integration-patterns.md](references/integration-patterns.md) for push-to-talk, reconnection, multi-room

## Resources

- **Main Docs**: https://docs.4players.io/
- **Voice**: https://docs.4players.io/voice/
- **Fleet**: https://docs.4players.io/fleet/
- **Rooms**: https://docs.4players.io/rooms/
- **Console**: https://console.4players.io
- **Discord**: https://4np.de/discord
- **Support**: support@4players.io

## Summary

ODIN provides a complete platform for real-time voice communication and game server hosting with:
- Simple Integration (few lines of code)
- Flexible Pricing (free tier for dev, scalable paid tiers)
- Global Infrastructure (hosted by 4Players)
- Cross-Platform (unified API)
- Enterprise Security (JWT auth, server-side keys)

For platform-specific implementation, refer to the corresponding SDK skill (odin-voice-web, odin-voice-unity, odin-voice-unreal, etc.).
