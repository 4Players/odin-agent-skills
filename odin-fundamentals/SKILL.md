---
name: odin-fundamentals
description: |
  ODIN Platform Fundamentals - Core concepts, architecture, authentication, and pricing.
  Use when: learning about ODIN platform basics, understanding access keys vs tokens,
  exploring pricing tiers, understanding room-based architecture, working with authentication,
  or getting overview of ODIN Voice/Fleet/Rooms products. Covers platform fundamentals,
  authentication workflows, security best practices, and billing models for developers
  integrating real-time voice/video communication or game server hosting.
---

# ODIN Platform Fundamentals

Comprehensive guide to ODIN platform architecture, authentication, pricing, and core concepts for integrating real-time communication and server hosting into your applications.

## Platform Overview

ODIN is a unified real-time communication and server management platform developed by 4Players, comprising three core products:

### ODIN Voice

Cross-platform voice chat SDK for games, apps, and websites.

**Supported Platforms:**
- Unity (with automatic 3D spatial audio)
- Unreal Engine 4/5 (Blueprint and C++ support)
- C/C++ (Core SDK foundation)
- Web (JavaScript/TypeScript, React, Angular, Vue, Svelte)
- NodeJS
- Swift/SwiftUI (iOS and macOS via OdinKit)

**Key Features:**
- **3D Spatial Audio**: Automatic positional audio in Unity and Unreal
- **Noise Suppression**: Built-in audio processing for clean voice quality
- **Low Latency**: Optimized for real-time gaming and communication
- **Minimal CPU Load**: Zero FPS impact on games
- **Cross-Platform**: Same room supports mobile, web, desktop simultaneously
- **Optional Data Channels**: Synchronize custom game data beyond voice
- **Automatic Routing**: Connects to nearest data centers for optimal quality

**Architecture:**
- Standalone client-server model hosted by 4Players
- Players connect to game server (standard) + ODIN servers (voice routing)
- Pure client-side integration - no backend changes required
- Voice participation is optional per player

### ODIN Fleet

Game-server hosting across a global network.

**Key Features:**
- Deploy stateful, real-time game servers worldwide
- Engine-agnostic architecture (supports any game backend)
- Automatic scaling based on player numbers
- Powerful API and CLI for deployment automation
- Dashboard for monitoring, logging, and backups
- Custom server images with CPU/memory configuration

### ODIN Rooms

Browser-based, decentralized video conferencing solution.

**Key Features:**
- End-to-end encrypted communications
- No metadata storage
- GDPR compliant infrastructure
- Custom branding and self-hosting options
- Integrated whiteboard and text chat
- Free tier for small businesses and NGOs

## Core Architecture

### Room-Based Model

ODIN uses a room-based architecture where:
- **Rooms**: Virtual spaces that route voice/video packets among participants
- **Peers**: Individual users connected to a room
- **Media Streams**: Audio/video streams that can be added/removed dynamically
- **Connection Pool**: Efficient management of multiple concurrent connections

### Technical Infrastructure

**Hosting:**
- Globally hosted by 4Players with decades of server expertise
- Built on modern technologies including HTTP/3 and WebTransport
- Automatic server maintenance and scaling
- Zero configuration required from developers
- Optional self-hosted on-premise deployment

**Event Architecture:**
- RPC (Remote Procedure Calls) with MessagePack-encoded data
- Event-driven callbacks for joins, media changes, messages
- Voice Activity Detection updates via encoder/decoder callbacks
- Separate signaling and media transport layers

**Audio Processing Module (APM):**
- Echo cancellation
- Noise suppression
- Automatic gain control (AGC)
- Voice Activity Detection (VAD)
- Volume gating
- Configurable via platform-specific settings

## Authentication & Security

### Access Keys

Access Keys are sensitive credentials used to authenticate your application with the ODIN platform.

**Key Characteristics:**
- **Server-Side Only**: Never embed in client-side code or commit to version control
- **Used to Generate Tokens**: Access Keys create signed JWT tokens for room access
- **Project-Specific**: Create separate keys for different projects/environments
- **Managed in Dashboard**: Available for paid subscribers to monitor usage

**Access Key Tiers:**
- **Free/Development Keys**: Valid for up to 25 concurrent connected users (CCU)
- **Paid Tier Keys**: Support unlimited users with budget management controls

**Security Best Practices:**
- Store Access Keys in environment variables or secret managers
- Never expose keys in client-side code, repositories, or public APIs
- Rotate keys periodically for enhanced security
- Use separate keys for development, staging, and production environments

### Access Tokens

Access Tokens are signed JSON Web Tokens (JWT) used by clients to join ODIN rooms.

**Token Structure:**
- **Room ID (`rid`)**: Identifies which room the user can join
- **User ID (`uid`)**: Unique identifier for the user
- **Lifetime**: Optional expiration time (in seconds)
- **Custom Claims**: Additional metadata as needed

**Token Generation Workflow:**

1. **Backend Token Generation** (Node.js example):
```javascript
const { TokenGenerator } = require('@4players/odin-tokens');

const accessKey = process.env.ODIN_ACCESS_KEY; // From environment
const generator = new TokenGenerator(accessKey);

const token = generator.createToken(
    'my-room-id',        // Room identifier
    'user-123',          // User identifier
    { lifetime: 300 }    // Optional: 5 minutes TTL
);
```

2. **Client Token Usage**:
```javascript
// Fetch token from your backend
const response = await fetch('/api/odin-token');
const { token } = await response.json();

// Join room with token
await room.join(token, {
    gateway: 'https://gateway.odin.4players.io'
});
```

**Important Security Notes:**
- Tokens MUST be generated server-side to protect Access Keys
- Validate user authentication before issuing tokens
- Set appropriate token lifetimes for your use case
- Use HTTPS for all token exchanges
- Tokens expire and cannot be refreshed - generate new ones as needed

### Gateway Options

**European Gateway (Default):**
```javascript
await room.join(token);
// Uses: https://gateway.odin.4players.io
```

**US Gateway:**
```javascript
await room.join(token, {
    gateway: 'https://gateway-us.odin.4players.io'
});
```

## Pricing Model

### Trial Tier (Free Forever)

**Cost:** €0/month (never billed)

**Resources:**
- 1,440 CPU-hours per month
- 2,880 GB-RAM-hours per month
- Up to 25 concurrent voice users (CCU)

**Limitations:**
- Intended for evaluation only
- Not suitable for live production use
- Servers auto-shutdown when resources exhaust
- No resource rollover to next month

**Use Cases:**
- Development and testing
- Small-scale prototypes
- Educational projects

### Indie Starter Plan

**Cost:** €49/month (never exceeds this amount)

**Resources:**
- 14,400 CPU-hours per month
- 28,800 GB-RAM-hours per month
- Up to 500 concurrent voice users (CCU)
- 4 hours one-on-one support with engineer

**Grace Period:**
- If usage temporarily exceeds pool, servers scale up to 200% for 48 hours at no extra cost
- After grace period, servers stop (no surprise charges)

**Eligibility Requirements:**
- Studios with <€250,000 annual turnover
- ≤10 employees

**Billing:**
- Pro-rata billing for mid-month starts/switches
- Resources refilled monthly
- Never charges beyond €49/month - servers stop instead

### Pay-as-you-Go (Pro)

**Voice Pricing** (per Peak Concurrent User per month):
- 1–19,999 PCUs: €0.29/PCU
- 20,000–39,999 PCUs: €0.25/PCU
- 40,000–59,999 PCUs: €0.23/PCU
- 60,000–79,999 PCUs: €0.20/PCU
- ≥80,000 PCUs: €0.19/PCU

**Fleet Pricing Examples** (daily rates):
- **S-2** (1 vCPU / 2GB RAM): €0.60/day (~€18/month)
- **M-4** (2 vCPU / 4GB RAM): €0.93/day (~€27.90/month)
- **L-8** (4 vCPU / 8GB RAM): €2.00/day (~€60/month)

**Billing Details:**
- Monthly cycles run from first to last day of month
- Usage tracked daily
- Highest daily concurrent instance count determines monthly cost
- No hidden traffic fees
- No minimum commitment

**Volume Discounts:**
- Automatic tier-based pricing for voice users
- Significant savings at scale (€0.19 vs €0.29 per PCU)

## Core Concepts

### Rooms

Virtual spaces where users communicate.

**Room Properties:**
- **Room ID**: Unique identifier for the room
- **Peers**: Collection of connected users
- **Media Streams**: Audio/video streams from peers
- **User Data**: Custom metadata attached to room
- **Position**: Optional 3D coordinates for spatial audio
- **Connection Status**: idle, connecting, connected, disconnected

**Room Operations:**
- Join/leave rooms with token authentication
- Send messages to all peers or specific targets
- Add/remove audio and video inputs
- Update position for 3D spatial audio culling
- Monitor connection statistics (RTT, packet loss)

### Peers

Individual users within a room.

**Peer Types:**
- **LocalPeer**: Your own user instance
- **RemotePeer**: Other users in the room

**Peer Properties:**
- **Peer ID**: Unique numeric identifier
- **User ID**: String identifier from token
- **User Data**: Custom metadata (synced across peers)
- **Media Streams**: Active audio/video streams
- **Volume**: Individual volume control

**Peer Operations:**
- Set volume for specific peers
- Send direct messages to peers
- Update user data (synced automatically)
- Monitor audio activity and power levels

### Media Streams

Audio and video streams transmitted within rooms.

**Audio Inputs:**
- Microphone capture with processing
- Volume control (0=mute, 1=normal, 2=boost)
- Voice Activity Detection
- Power level monitoring (dBFS)
- Loopback for local monitoring

**Audio Outputs:**
- Remote peer audio playback
- Per-stream volume control
- Pause/resume capabilities
- Activity monitoring

**Video Inputs:**
- Camera capture from MediaStream
- Custom video sources
- Custom type identifiers

**Media Management:**
- Explicit signaling for start/stop/pause/resume
- RPC commands to server for media operations
- Event callbacks for media status changes

### Data Channels

Optional binary data transmission alongside voice/video.

**Use Cases:**
- Game state synchronization
- Player position updates
- Custom events and notifications
- Chat messages
- Metadata sharing

**Message Operations:**
- Broadcast to all peers in room
- Send to specific peer IDs
- Binary data (Uint8Array) transmission
- Automatic serialization helpers

## Integration Workflow

### Basic Integration Steps

1. **Obtain Access Key:**
   - Sign up at https://www.4players.io/odin/
   - Create app in dashboard
   - Copy Access Key (store securely)

2. **Set Up Backend Token Generation:**
   - Install `@4players/odin-tokens` (Node.js) or equivalent
   - Create API endpoint to generate tokens
   - Validate user authentication before issuing tokens

3. **Install SDK in Your Application:**
   - Use platform-specific package manager
   - Web: `npm install @4players/odin @4players/odin-plugin-web`
   - Unity: Import package from Unity Asset Store or GitHub
   - Unreal: Install plugin from marketplace or source

4. **Initialize ODIN:**
   - Create room instance
   - Register event handlers (before joining)
   - Fetch token from backend
   - Join room with token

5. **Configure Audio:**
   - Set output device (speakers/headphones)
   - Create audio input (microphone)
   - Add audio input to room
   - Configure APM settings (echo cancellation, noise suppression)

6. **Handle Events:**
   - Peer joined/left
   - Audio started/stopped
   - Voice activity detection
   - Messages received
   - Connection status changes

7. **Cleanup on Exit:**
   - Remove audio inputs
   - Close audio inputs
   - Leave room
   - Release resources

### Minimal Implementation Example

```javascript
// Backend: Generate token
app.get('/api/odin-token', (req, res) => {
    const generator = new TokenGenerator(process.env.ODIN_ACCESS_KEY);
    const token = generator.createToken('room-id', req.user.id);
    res.json({ token });
});

// Client: Join room
const response = await fetch('/api/odin-token');
const { token } = await response.json();

const room = new ODIN.Room();
room.onPeerJoined = (payload) => console.log('Peer joined:', payload.peer.userId);
room.onPeerLeft = (payload) => console.log('Peer left:', payload.peer.userId);

await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
```

## Best Practices

### Security

- **Never expose Access Keys** in client code, repositories, or public APIs
- **Always generate tokens server-side** with proper user authentication
- **Use environment variables** for Access Key storage
- **Set appropriate token lifetimes** (5-15 minutes for most use cases)
- **Implement token refresh** mechanisms for long sessions
- **Use HTTPS** for all token exchanges

### Performance

- **Register events before joining** to avoid missing early events
- **Use connection pooling** for multi-room applications
- **Monitor connection stats** (RTT, packet loss) for quality insights
- **Implement reconnection logic** with exponential backoff
- **Release resources** properly (close inputs, leave rooms)

### User Experience

- **Allow device selection** for microphone and speakers
- **Show connection status** to users (connecting, connected, disconnected)
- **Display peer list** with speaking indicators
- **Implement mute/unmute** controls
- **Provide volume controls** (master and per-peer)
- **Show voice activity** indicators for active speakers
- **Handle errors gracefully** with user-friendly messages

### Development

- **Use trial tier** for development and testing
- **Separate keys** for development, staging, production
- **Monitor usage** in ODIN dashboard (paid tiers)
- **Test reconnection** scenarios and edge cases
- **Implement logging** for debugging issues
- **Follow platform-specific** integration guides

## Platform Comparison

### ODIN Voice vs Competitors

**Advantages:**
- Built-in 3D spatial audio for game engines
- Cross-platform SDK with consistent API
- Low latency optimized for gaming
- Minimal CPU overhead
- No infrastructure management required
- Automatic scaling and routing
- Simple integration (few lines of code)

**Use Cases:**
- Multiplayer game voice chat
- Social platforms and virtual events
- Collaboration tools and meetings
- Educational applications
- Customer support with voice
- Metaverse and VR experiences

### ODIN Fleet vs Traditional Hosting

**Advantages:**
- Automatic scaling based on demand
- Global distribution for low latency
- Engine-agnostic (any game backend)
- API and CLI for automation
- Built-in monitoring and logging
- Managed backups and recovery
- Pay-per-use pricing model

**Use Cases:**
- Multiplayer game servers
- Match-based game sessions
- Tournament hosting
- Beta testing and load testing
- Regional server deployment

## Supported Tools & Resources

### Developer Tools

- **ODIN CLI**: Command-line interface for Fleet management
- **Developer Console**: https://console.4players.io
- **Token Generator Libraries**: Available for multiple languages
- **Dashboard**: Usage monitoring and key management (paid tiers)

### Documentation & Support

- **Main Documentation**: https://docs.4players.io/
- **Voice SDK Docs**: https://docs.4players.io/voice/
- **Fleet Docs**: https://docs.4players.io/fleet/
- **Rooms Docs**: https://docs.4players.io/rooms/
- **GitHub**: https://github.com/4players (examples and SDKs)
- **Discord Community**: https://4np.de/discord
- **Email Support**: support@4players.io

### Getting Started Resources

1. **Sign Up**: https://www.4players.io/odin/
2. **Create App**: Generate Access Key in dashboard
3. **Choose SDK**: Select platform-specific SDK
4. **Follow Guides**: Platform-specific integration guides
5. **Join Community**: Discord for questions and support

## FAQ

**Q: What's the difference between Access Keys and Access Tokens?**
A: Access Keys are server-side credentials used to generate Access Tokens. Access Tokens are client-side JWT tokens that allow users to join specific rooms. Always keep Access Keys secret on your backend.

**Q: Can I use ODIN for free?**
A: Yes, the Trial tier supports up to 25 concurrent users forever at no cost. Perfect for development and small projects.

**Q: Does ODIN work across different platforms?**
A: Yes, ODIN is fully cross-platform. A web user can talk to a Unity user in the same room seamlessly.

**Q: How do I scale from Trial to Production?**
A: Upgrade to Indie Starter (€49/month, 500 users) or Pay-as-you-Go (scales infinitely) based on your needs and budget.

**Q: Where is my data hosted?**
A: 4Players hosts ODIN infrastructure globally. You can choose EU or US gateways. Self-hosted on-premise deployment is also available.

**Q: Do I need to manage servers?**
A: No, 4Players handles all server infrastructure, maintenance, and scaling automatically.

**Q: What audio codecs does ODIN use?**
A: ODIN uses Opus codec for high-quality, low-latency audio transmission.

**Q: Can I send custom data alongside voice?**
A: Yes, ODIN supports binary data channels for game state, chat messages, and custom events.

**Q: Is there a bandwidth limit?**
A: No hidden traffic fees. Voice pricing is based on concurrent users, not bandwidth consumption.

**Q: How do I implement Push-to-Talk?**
A: Set audio input volume to 0 (muted), then set to 1 on key press. See platform-specific examples.

## Common Integration Patterns

### Multi-Room Applications

```javascript
const connectionPool = new ODIN.ConnectionPool();
const room1 = connectionPool.createRoom();
const room2 = connectionPool.createRoom();

await room1.join(token1, { gateway: 'https://gateway.odin.4players.io' });
await room2.join(token2, { gateway: 'https://gateway.odin.4players.io' });
```

### Push-to-Talk

```javascript
let audioInput = await ODIN.DeviceManager.createAudioInput(null, { /* APM settings */ });
audioInput.setVolume(0); // Start muted
await room.addAudioInput(audioInput);

// On key press
document.addEventListener('keydown', (e) => {
    if (e.code === 'Space') audioInput.setVolume(1); // Unmute
});

// On key release
document.addEventListener('keyup', (e) => {
    if (e.code === 'Space') audioInput.setVolume(0); // Mute
});
```

### Reconnection Logic

```javascript
let reconnectAttempts = 0;
const MAX_ATTEMPTS = 5;

room.onLeft = async () => {
    if (reconnectAttempts < MAX_ATTEMPTS) {
        reconnectAttempts++;
        const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000);
        await new Promise(resolve => setTimeout(resolve, delay));

        try {
            await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
            reconnectAttempts = 0; // Reset on success
        } catch (error) {
            console.error('Reconnection failed:', error);
        }
    }
};
```

### User Data Synchronization

```javascript
// Set user data
const userData = ODIN.valueToUint8Array({
    nickname: 'Player1',
    level: 42,
    status: 'online'
});
room.userData = userData;
room.flushUserData();

// Receive peer data updates
room.onUserDataChanged = (payload) => {
    const data = ODIN.uint8ArrayToValue(payload.peer.data);
    console.log(`${payload.peer.userId} updated:`, data);
};
```

## Summary

ODIN provides a complete platform for integrating real-time voice communication and game server hosting with:

- **Simple Integration**: Few lines of code to get started
- **Flexible Pricing**: Free tier for development, scalable paid tiers
- **Global Infrastructure**: Hosted and maintained by 4Players
- **Cross-Platform**: Unified API across Unity, Unreal, Web, Mobile
- **Enterprise Security**: JWT-based authentication with server-side key management
- **Production-Ready**: Used by thousands of games and applications worldwide

For implementation details specific to your platform, refer to the corresponding SDK skill (odin-voice-web, odin-voice-unity, odin-voice-unreal, etc.).
