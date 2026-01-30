---
name: odin-voice-web
description: |
  ODIN Voice Web SDK - Browser SDK for real-time voice chat with WebTransport/HTTP3.
  Use when: implementing voice chat in web apps, browsers, or web-based applications,
  working with Room/Peer/AudioInput/AudioOutput classes, handling audio events,
  configuring audio input/output devices, or integrating with React/Angular/Vue frameworks.
  Supports NPM and CDN. Enables seamless voice communication in multiplayer games,
  social platforms, collaboration tools, and interactive web experiences.
license: MIT
---

# ODIN Voice Web SDK

Browser SDK for real-time voice chat with WebTransport/HTTP3 and WebRTC fallback, fully compatible with native ODIN SDKs across Unity, Unreal, NodeJS, and Swift platforms.

## Installation

### NPM (Recommended)

Install both core packages:

```bash
npm install @4players/odin @4players/odin-plugin-web
```

**For TypeScript projects**: Type definitions are included automatically.

### CDN (Browser-only)

**IIFE (Global Variables)**:
```html
<script src="https://cdn.odin.4players.io/client/js/sdk/latest/odin-sdk.js"></script>
<script src="https://cdn.odin.4players.io/client/js/sdk/latest/odin-plugin.js"></script>
<script>
  // Access via global ODIN and ODIN_PLUGIN objects
  console.log(ODIN.Room);
</script>
```

**ESM (ES Modules)**:
```html
<script type="module">
  import * as ODIN from 'https://cdn.odin.4players.io/client/js/sdk/latest/odin-sdk.esm.js';
  import * as ODIN_PLUGIN from 'https://cdn.odin.4players.io/client/js/sdk/latest/odin-plugin.esm.js';
</script>
```

## Quick Start

```javascript
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

let audioPlugin;
let room;
let audioInput;

// Step 1: Initialize plugin (call once per application, requires user interaction)
async function initPlugin() {
    if (audioPlugin) return audioPlugin;

    audioPlugin = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
        const audioContext = new AudioContext({ sampleRate });
        await audioContext.resume();
        return audioContext;
    });

    ODIN.init(audioPlugin);
    return audioPlugin;
}

// Step 2: Join voice room (call from button click or user interaction)
async function joinRoom(token) {
    await initPlugin();

    room = new ODIN.Room();
    setupEventHandlers(room);

    await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
    await audioPlugin.setOutputDevice({});

    audioInput = await ODIN.DeviceManager.createAudioInput(null, {
        echoCanceller: true,
        noiseSuppression: true,
        gainController: true,
    });
    await room.addAudioInput(audioInput);
}

// Step 3: Setup event handlers
function setupEventHandlers(room) {
    room.onJoined = (payload) => console.log(`Joined! Own peer ID: ${payload.ownPeer.id}`);
    room.onLeft = () => console.log('Left room');
    room.onPeerJoined = (payload) => console.log(`Peer joined: ${payload.peer.userId}`);
    room.onPeerLeft = (payload) => console.log(`Peer left: ${payload.peer.userId}`);
    room.onAudioOutputStarted = (payload) => console.log(`Audio from: ${payload.peer.userId}`);
}

// Step 4: Leave room when done
async function leaveRoom() {
    if (audioInput) {
        await room.removeAudioInput(audioInput);
        audioInput.close();
    }
    if (room) room.leave();
}
```

### Critical Requirements

- **User Interaction**: Modern browsers require direct user interaction (click/tap) to start AudioContext
- **Event Registration**: Register all event handlers BEFORE calling `room.join()`
- **Audio Activation**: Audio is only transmitted after calling `room.addAudioInput(audioInput)`

## Core Classes

### Room

Central hub for voice communication.

```javascript
// Properties
room.id                 // String: Room identifier
room.ownPeer            // LocalPeer: Your own peer instance
room.remotePeers        // Map<number, RemotePeer>: Connected peers
room.status             // 'idle' | 'connecting' | 'connected' | 'disconnected'
room.connectionStats    // Object: Network statistics (RTT, packet loss)

// Methods
await room.join(token, { gateway?: string })
room.leave()
await room.addAudioInput(audioInput)
await room.removeAudioInput(audioInput)
room.setVolume(value)              // 0-2 or "muted"
await room.setPosition(x, y, z)    // 3D positional audio
await room.sendMessage(message, targetPeerIds?)
```

### LocalPeer / RemotePeer

```javascript
// Properties
peer.id         // Number: Peer ID
peer.userId     // String: User identifier from token
peer.data       // Uint8Array: Custom user data

// Methods
peer.setVolume(value)           // 0-2 or "muted"
await peer.sendMessage(message) // RemotePeer only
```

### AudioInput

```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput(device?, settings?);

audioInput.volume       // Number: Capture volume (0-2)
audioInput.isActive     // Boolean: Currently capturing
audioInput.powerLevel   // Number: RMS in dBFS

audioInput.setVolume(value)
await audioInput.setDevice(device)
await audioInput.setInputSettings(settings)
audioInput.close()
```

### DeviceManager

```javascript
await ODIN.DeviceManager.listAudioInputs()   // Available microphones
await ODIN.DeviceManager.listAudioOutputs()  // Available speakers
await ODIN.DeviceManager.createAudioInput(device?, settings?)
await ODIN.DeviceManager.createVideoInput(mediaStream, options?)
```

## Event System

Register handlers BEFORE calling `room.join()`:

```javascript
// Pattern 1: Property assignment
room.onPeerJoined = (payload) => { /* ... */ };

// Pattern 2: addEventListener (multiple handlers)
room.addEventListener('PeerJoined', (event) => { /* ... */ });
```

### Key Events

```javascript
// Lifecycle
room.onJoined = ({ ownPeer, room, mediaIds }) => { };
room.onLeft = ({ room }) => { };
room.onStatusChanged = ({ status }) => { };

// Peers
room.onPeerJoined = ({ peer }) => { };
room.onPeerLeft = ({ peer }) => { };
room.onUserDataChanged = ({ peer }) => { };

// Audio
room.onAudioOutputStarted = ({ peer, media }) => { };
room.onAudioOutputStopped = ({ peer, media }) => { };
room.onAudioActivity = ({ peer, media, active }) => { };
room.onAudioPowerLevel = ({ peer, media, powerLevel }) => { };

// Messaging
room.onMessageReceived = ({ peer, message }) => { };
```

## Authentication

Tokens must be generated server-side to protect your access key.

### Backend (Node.js)

```javascript
const { TokenGenerator } = require('@4players/odin-tokens');
const generator = new TokenGenerator(process.env.ODIN_ACCESS_KEY);

app.get('/api/odin-token', (req, res) => {
    const token = generator.createToken('room-id', req.user.id, { lifetime: 300 });
    res.json({ token });
});
```

### Frontend

```javascript
const response = await fetch('/api/odin-token');
const { token } = await response.json();
await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
```

**Security**: Never embed access keys in client-side code.

## Common Patterns

### User Data & Messaging

```javascript
// Set user data
room.userData = ODIN.valueToUint8Array({ nickname: 'Alice' });
room.flushUserData();

// Send message
await room.sendMessage(ODIN.valueToUint8Array({ type: 'chat', text: 'Hello!' }));

// Receive message
room.onMessageReceived = (payload) => {
    const data = ODIN.uint8ArrayToValue(payload.message);
};
```

### Video Integration

```javascript
const stream = await navigator.mediaDevices.getUserMedia({ video: true });
const videoInput = await ODIN.DeviceManager.createVideoInput(stream);
await room.addVideoInput(videoInput);

room.onVideoOutputStarted = async (payload) => {
    await payload.media.start();
    const video = document.createElement('video');
    video.srcObject = payload.media.mediaStream;
    video.autoplay = true;
};
```

## Additional Documentation

- **Audio Processing (APM)**: See [references/audio-processing.md](references/audio-processing.md) for presets and dBFS tuning
- **Framework Integration**: See [references/framework-integration.md](references/framework-integration.md) for React, Vue, Angular examples
- **Advanced Patterns**: See [references/advanced-patterns.md](references/advanced-patterns.md) for push-to-talk, reconnection, device selection

## Troubleshooting

**"AudioContext not allowed to start"**: Call `initPlugin()` from a button click handler.

**No audio from peers**: Verify `await audioPlugin.setOutputDevice({})` was called.

**Microphone not transmitting**: Check `await room.addAudioInput(audioInput)` was called.

**Events not firing**: Register handlers BEFORE calling `room.join()`.

## Browser Compatibility

- Chrome/Edge: 94+
- Firefox: 93+
- Safari: 15.4+
- Mobile: iOS 15.4+, Android Chrome 94+

## Resources

- **API Reference**: https://docs.4players.io/voice/web/api/
- **NPM Package**: https://www.npmjs.com/package/@4players/odin
- **GitHub Examples**: https://github.com/4Players/odin-sdk-web-examples
- **Get Access Keys**: https://www.4players.io/odin/
