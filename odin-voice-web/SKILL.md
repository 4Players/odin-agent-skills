---
name: odin-voice-web
description: |
  ODIN Voice Web SDK for real-time voice chat in browsers and web applications.
  Use when: implementing voice chat in web apps, working with Room/Peer classes, handling events,
  configuring audio input/output, or integrating with React/Angular/Vue. Supports NPM and CDN.
  For ODIN concepts, see odin-voice skill.
---

# ODIN Voice Web SDK

Browser SDK for real-time voice chat with WebTransport/HTTP3 and WebRTC fallback.

## Installation

### NPM
```bash
npm i @4players/odin
npm i @4players/odin-plugin-web
```

### CDN
```html
<!-- IIFE -->
<script src="https://cdn.odin.4players.io/client/js/sdk/1.0.1/odin-sdk.js"></script>
<script src="https://cdn.odin.4players.io/client/js/sdk/1.1.0/odin-plugin.js"></script>

<!-- ESM -->
<script type="module">
import * as ODIN from 'https://cdn.odin.4players.io/client/js/sdk/1.0.1/odin-sdk.esm.js';
import * as ODIN_PLUGIN from 'https://cdn.odin.4players.io/client/js/sdk/1.1.0/odin-plugin.esm.js';
</script>
```

## Quick Start

### Plugin Initialization (Required)
```javascript
let audioPlugin;

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
```

### Basic Connection
```javascript
// 1. Initialize plugin (once per app)
await initPlugin();

// 2. Create room
const room = new ODIN.Room();

// 3. Register events BEFORE joining
room.onJoined = (payload) => console.log('Joined!', payload);
room.onPeerJoined = (payload) => console.log('Peer:', payload.peer.userId);
room.onAudioOutputStarted = (payload) => console.log('Audio from:', payload.peer.userId);

// 4. Join with token (from backend)
await room.join(token, { gateway: 'https://gateway.odin.4players.io' });

// 5. Set output device
await audioPlugin.setOutputDevice({});

// 6. Add microphone
const audioInput = await ODIN.DeviceManager.createAudioInput();
await room.addAudioInput(audioInput);

// 7. Leave when done
room.leave();
```

## Key Classes

### Room
Central hub for voice communication.

```javascript
// Properties
room.id                 // Room identifier
room.ownPeerId          // Local peer ID
room.ownPeer            // LocalPeer instance
room.remotePeers        // Map of RemotePeer
room.status             // Connection status
room.connectionStats    // Network stats
room.position           // [x, y, z] for 3D

// Methods
room.join(token, options?)           // Connect
room.leave()                         // Disconnect
room.addAudioInput(input)            // Add microphone
room.removeAudioInput(input)         // Remove microphone
room.setVolume(value)                // Output volume (0-2 or "muted")
room.setPosition(x, y, z)            // Update 3D position
room.sendMessage(msg, targetIds?)    // Send data
room.setAudioOutputDevice(device?)   // Select speaker
```

### Peer (LocalPeer / RemotePeer)

```javascript
// Shared properties
peer.id                 // Unique peer ID
peer.userId             // User identifier from token
peer.isRemote           // true for RemotePeer
peer.data               // User data (Uint8Array)

// LocalPeer specific
localPeer.audioInputs   // Array of AudioInput
localPeer.setVolume(value)
localPeer.update()      // Flush user data to server

// RemotePeer specific
remotePeer.audioOutputs  // Array of AudioOutput
remotePeer.setVolume(value)
remotePeer.sendMessage(msg)
```

### AudioInput (Microphone)

```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput(device?, settings?);

// Properties
audioInput.volume       // Capture volume
audioInput.isActive     // Currently capturing
audioInput.powerLevel   // RMS in dBFS

// Methods
audioInput.setVolume(value)
audioInput.setDevice(device)
audioInput.setInputSettings(settings)
audioInput.close()
audioInput.startLoopback() / stopLoopback()
```

### AudioOutput (Remote Playback)

```javascript
// Properties
audioOutput.peer        // Associated RemotePeer
audioOutput.mediaId     // Media identifier
audioOutput.volume      // Playback volume
audioOutput.isActive    // Currently playing
audioOutput.isPaused    // Pause state

// Methods
audioOutput.setVolume(value?)
audioOutput.pause() / resume()
audioOutput.close()
```

### DeviceManager

```javascript
// List devices
await ODIN.DeviceManager.listAudioInputs()
await ODIN.DeviceManager.listAudioOutputs()
await ODIN.DeviceManager.listDevices(kind?)

// Create inputs
await ODIN.DeviceManager.createAudioInput(device?, settings?, customType?)
await ODIN.DeviceManager.createVideoInput(mediaStream, options?)

// Find specific device
await ODIN.DeviceManager.getInputDevice(id)
await ODIN.DeviceManager.getOutputDevice(id)
```

## Room Events

```javascript
// Lifecycle
room.onJoined           // Successfully joined
room.onLeft             // Disconnected
room.onStatusChanged    // Connection status

// Peers
room.onPeerJoined       // Peer joined (payload.peer)
room.onPeerLeft         // Peer left (payload.peer)
room.onUserDataChanged  // Peer data updated

// Media
room.onAudioOutputStarted  // Remote audio started (payload.peer, payload.media)
room.onAudioOutputStopped  // Remote audio stopped
room.onAudioInputStarted   // Local input activated
room.onAudioInputStopped   // Local input stopped

// Data
room.onMessageReceived     // Message from peer (payload.peer, payload.message)
room.onConnectionStats     // Network statistics

// Audio analysis
room.onAudioActivity       // VAD detection
room.onAudioPowerLevel     // Power levels
```

### Event Handling Patterns

```javascript
// Pattern 1: Property assignment (overwrites previous)
room.onPeerJoined = (payload) => { /* handler */ };

// Pattern 2: addEventListener (can register multiple)
room.addEventListener('PeerJoined', (event) => {
    console.log(event.detail.peer);
});
```

## Audio Input Settings

```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput(null, {
    volume: 1,
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.9,
        releaseThreshold: 0.8,
    },
    volumeGate: {
        attackThreshold: -30,  // dBFS
        releaseThreshold: -40,
    }
});

// Update settings later
await audioInput.setInputSettings({
    noiseSuppression: false,
    volumeGate: { attackThreshold: -25 }
});
```

## Common Patterns

### User Data & Messaging
```javascript
// Set user data
const userData = ODIN.valueToUint8Array({ nickname: 'Alice', status: 'online' });
room.userData = userData;
room.flushUserData();

// Send message
const message = ODIN.valueToUint8Array({ type: 'chat', text: 'Hello!' });
await room.sendMessage(message);  // Broadcast
await remotePeer.sendMessage(message);  // To specific peer

// Receive message
room.onMessageReceived = (payload) => {
    const data = ODIN.uint8ArrayToValue(payload.message);
    console.log(`${payload.peer.userId}: ${data.text}`);
};
```

### Audio Level Monitoring
```javascript
audioInput.onAudioActivity = (payload) => {
    updateMicIndicator(payload.media.isActive);
};

remotePeer.onPowerLevel = (payload) => {
    updateSpeakingIndicator(payload.media.powerLevel);
};
```

### 3D Positional Audio
```javascript
// Update position for proximity-based culling
await room.setPosition(x, y, z);
const [x, y, z] = room.position;
```

### Video Integration
```javascript
// Add camera
const stream = await navigator.mediaDevices.getUserMedia({ video: true });
const videoInput = await ODIN.DeviceManager.createVideoInput(stream, { customType: 'cam' });
await room.addVideoInput(videoInput);

// Handle remote video
room.onVideoOutputStarted = async (payload) => {
    await payload.media.start();
    const video = document.createElement('video');
    video.srcObject = payload.media.mediaStream;
    video.autoplay = true;
    document.body.appendChild(video);
};
```

## Browser Requirements

- User interaction required to start AudioContext (click/tap)
- Call `setOutputDevice()` and `addAudioInput()` within user interaction handlers
- Firefox: Must request MediaStream once before device enumeration

## Security

- **NEVER** embed access keys in client code
- Generate tokens on your backend using `@4players/odin-tokens`

## Documentation

API reference: https://docs.4players.io/voice/web/api/
Vanilla JS guide: https://docs.4players.io/voice/web/vanillajs
React/Angular: https://docs.4players.io/voice/web/react, https://docs.4players.io/voice/web/angular
