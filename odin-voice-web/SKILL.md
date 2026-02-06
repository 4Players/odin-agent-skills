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

> **Warning**: Using `latest` in the CDN URL (e.g. `.../latest/odin-sdk.js`) means your app automatically picks up new releases, including breaking changes. We recommend pinning a specific version to avoid unexpected issues:
> ```
> https://cdn.odin.4players.io/client/js/sdk/1.3.12/odin-sdk.js
> https://cdn.odin.4players.io/client/js/sdk/1.3.12/odin-plugin.js
> ```
>
> **Note**: Check npm for the latest available versions:
> - SDK: https://www.npmjs.com/package/@4players/odin-sdk
> - Plugin: https://www.npmjs.com/package/@4players/odin-plugin

**IIFE (Global Variables)**:
```html
<script src="https://cdn.odin.4players.io/client/js/sdk/1.3.12/odin-sdk.js"></script>
<script src="https://cdn.odin.4players.io/client/js/sdk/1.3.12/odin-plugin.js"></script>
<script>
  // Access via global ODIN and ODIN_PLUGIN objects
  console.log(ODIN.Room);
</script>
```

**ESM (ES Modules)**:
```html
<script type="module">
  import * as ODIN from 'https://cdn.odin.4players.io/client/js/sdk/1.3.12/odin-sdk.esm.js';
  import * as ODIN_PLUGIN from 'https://cdn.odin.4players.io/client/js/sdk/1.3.12/odin-plugin.esm.js';
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

    await audioPlugin.setOutputDevice({});
    await room.join(token, { gateway: 'https://gateway.odin.4players.io' });

    audioInput = await ODIN.DeviceManager.createAudioInput();
    await room.addAudioInput(audioInput);
}

// Step 3: Setup event handlers
function setupEventHandlers(room) {
    room.onJoined = (payload) => console.log(`Joined! Own peer ID: ${payload.peer.id}`);
    room.onLeft = () => console.log('Left room');
    room.onPeerJoined = (payload) => console.log(`Peer joined: ${payload.peer.userId}`);
    room.onPeerLeft = (payload) => console.log(`Peer left: ${payload.peer.userId}`);
    room.onAudioOutputStarted = (payload) => console.log(`Audio from: ${payload.peer.userId}`);
}

// Step 4: Leave room when done
async function leaveRoom() {
    if (audioInput) {
        room.removeAudioInput(audioInput);
        audioInput.close();
    }
    if (room) room.leave();
}
```

### Critical Requirements

- **User Interaction**: Modern browsers require direct user interaction (click/tap) to start AudioContext
- **Event Registration**: Register all event handlers BEFORE calling `room.join()`
- **Output Device**: `await audioPlugin.setOutputDevice({})` MUST be called BEFORE `room.join()` to hear other peers. After the initial call it can be used at any time to switch the output device.
- **Audio Activation**: Audio is only transmitted after calling `room.addAudioInput(audioInput)`

## Core Classes

### Room

Central hub for voice communication.

```javascript
// Properties
room.id                 // String: Room identifier
room.ownPeer            // LocalPeer: Your own peer instance
room.remotePeers        // Map<number, RemotePeer>: Connected peers
room.status             // { status: 'disconnected' | 'joining' | 'joined' | 'reconnecting', reason?: string }
room.connectionStats    // Object: Network statistics (RTT, packet loss)

// Methods
await room.join(token, { gateway?: string })
room.leave()
await room.addAudioInput(audioInput)
room.removeAudioInput(audioInput)
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

audioInput.volume       // Number | 'muted': Capture volume (0-2 or 'muted')
audioInput.isActive     // Boolean: Currently capturing
audioInput.powerLevel   // Number: RMS in dBFS

await audioInput.setVolume(value)   // 0-2 or 'muted'
await audioInput.setDevice(device)
await audioInput.setInputSettings(settings)
audioInput.close()
```

> **Note**: In most cases, calling `createAudioInput()` without arguments is sufficient. The defaults use the system's default microphone with echo cancellation, noise suppression, and gain control already enabled. Only pass `device` or `settings` when you need to select a specific microphone or customize audio processing.

#### Muting

To mute, use `'muted'` instead of `0` -- this stops the underlying MediaStream so the browser's recording indicator disappears:

```javascript
// Mute: stops MediaStream, removes browser recording indicator
await audioInput.setVolume('muted');

// Unmute: restores capture at full volume
await audioInput.setVolume(1);
```

For a full mute that also stops the encoder (saving resources), remove the AudioInput from the room:

```javascript
// Full mute: stop encoder + stop MediaStream
room.removeAudioInput(audioInput);
await audioInput.setVolume('muted');

// Unmute: restore volume and re-add to room to resume encoding
await audioInput.setVolume(1);
await room.addAudioInput(audioInput);
```

### DeviceManager

```javascript
await ODIN.DeviceManager.listAudioInputs()   // Available microphones
await ODIN.DeviceManager.listAudioOutputs()  // Available speakers
await ODIN.DeviceManager.createAudioInput(device?, settings?)
await ODIN.DeviceManager.createVideoInput(mediaStream, options?)
```

## Event System

All events support two patterns. Register handlers BEFORE calling `room.join()`:

```javascript
// Pattern 1: Property assignment (single handler)
room.onPeerJoined = (payload) => { /* ... */ };

// Pattern 2: addEventListener (multiple handlers)
room.addEventListener('PeerJoined', (event) => {
    const payload = event.payload;
});
```

Both patterns work on `room` and on `peer` objects (e.g. `peer.onAudioActivity`, `peer.addEventListener('AudioActivity', ...)`).

### Room Events

| `on*` Callback | `addEventListener` Name | Payload |
|---|---|---|
| `onJoined` | `'Joined'` | `{ room, peer }` |
| `onLeft` | `'Left'` | `{ room, reason }` |
| `onStatusChanged` | `'RoomStatusChanged'` | `{ oldState, newState }` |
| `onDataChanged` | `'RoomDataChanged'` | `{ room }` |
| `onPeerJoined` | `'PeerJoined'` | `{ room, peer }` |
| `onPeerLeft` | `'PeerLeft'` | `{ room, peer }` |
| `onUserDataChanged` | `'UserDataChanged'` | `{ room, peer }` |
| `onAudioOutputStarted` | `'AudioOutputStarted'` | `{ room, peer, media }` |
| `onAudioOutputStopped` | `'AudioOutputStopped'` | `{ room, peer, media }` |
| `onAudioActivity` | `'AudioActivity'` | `{ media }` |
| `onAudioPowerLevel` | `'AudioPowerLevel'` | `{ media }` |
| `onMessageReceived` | `'MessageReceived'` | `{ room, peer, message }` |
| `onConnectionStats` | `'ConnectionStats'` | `{ rtt, bytesReceivedLastSecond, bytesSentLastSecond, ... }` |
| `onMediaStarted` | `'MediaStarted'` | `{ room, peer, media }` |
| `onMediaStopped` | `'MediaStopped'` | `{ room, peer, media }` |

> **Note**: The `addEventListener` names for `onStatusChanged` and `onDataChanged` are prefixed with `Room` (`'RoomStatusChanged'`, `'RoomDataChanged'`).

### Peer Events

Peer objects also emit events. Use these on individual `RemotePeer` or `LocalPeer` instances:

| `on*` Callback | `addEventListener` Name | Payload |
|---|---|---|
| `onAudioActivity` | `'AudioActivity'` | `{ media }` |
| `onPowerLevel` | `'AudioPowerLevel'` | `{ media }` |
| `onUserDataChanged` | `'UserDataChanged'` | `{ room, peer }` |
| `onAudioOutputStarted` | `'AudioOutputStarted'` | `{ room, peer, media }` |
| `onAudioOutputStopped` | `'AudioOutputStopped'` | `{ room, peer, media }` |
| `onMediaStarted` | `'MediaStarted'` | `{ room, peer, media }` |
| `onMediaStopped` | `'MediaStopped'` | `{ room, peer, media }` |

```javascript
// Example: listen for audio activity on a specific peer
room.onPeerJoined = ({ peer }) => {
    peer.onAudioActivity = ({ media }) => {
        console.log(`Peer ${peer.userId} speaking: ${media.isActive}`);
    };
};
```

### Media Events (AudioInput / AudioOutput)

Individual `AudioInput` and `AudioOutput` instances also emit events. Use these for per-media monitoring (e.g. visualizing the local microphone level independently from room-level events).

> **Note**: The `addEventListener` names on media objects are `'Activity'` and `'PowerLevel'` (not `'AudioActivity'` / `'AudioPowerLevel'` as on Room/Peer).

**AudioInput events:**

| `on*` Callback | `addEventListener` Name | Payload |
|---|---|---|
| `onAudioActivity` | `'Activity'` | `{ media }` |
| `onPowerLevel` | `'PowerLevel'` | `{ media }` |

**AudioOutput events:**

| `on*` Callback | `addEventListener` Name | Payload |
|---|---|---|
| `onAudioActivity` | `'Activity'` | `boolean` (isActive) |
| `onPowerLevel` | `'PowerLevel'` | `number` (powerLevel in dBFS) |
| `onJitterStats` | `'JitterStats'` | `JitterStats` |

> **Note**: `AudioOutput` callbacks receive simplified values (`boolean` / `number`), while `AudioInput` callbacks receive `{ media }` payload objects. The `addEventListener` pattern always wraps the payload in `event.payload`.

```javascript
// Example: monitor local microphone activity and power level
const audioInput = await ODIN.DeviceManager.createAudioInput({});

audioInput.onAudioActivity = ({ media }) => {
    console.log('Mic active:', media.isActive);
};

audioInput.onPowerLevel = ({ media }) => {
    console.log('Mic power level:', media.powerLevel, 'dBFS');
};

// Or via addEventListener
audioInput.addEventListener('Activity', (event) => {
    console.log('Activity:', event.payload.media.isActive);
});
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

**No audio from peers**: Verify `await audioPlugin.setOutputDevice({})` was called BEFORE `room.join()`.

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
