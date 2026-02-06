
# ODIN Voice NodeJS SDK

Server-side voice chat SDK with native C++ bindings for building recording bots, AI assistants, and audio processing applications.

## Overview

- **Native C++ bindings** for high-performance server-side applications
- **Raw PCM audio access** for recording, processing, and AI integration
- **End-to-End Encryption** with OdinCipher
- **3D positional audio** support for spatial voice
- **Built on ODIN Core SDK** (same foundation as Unity, Unreal, Web, Swift)

## Installation

```bash
npm install @4players/odin-nodejs
```

**Build Requirements:**
- **macOS**: `xcode-select --install`
- **Ubuntu/Debian**: `sudo apt-get install build-essential python3`
- **Windows**: `npm install --global windows-build-tools`

**Note**: Cannot combine Web SDK (`@4players/odin`) and NodeJS SDK in the same project.

## Quick Start

```javascript
const { OdinClient, OdinCipher } = require('@4players/odin-nodejs');

const client = new OdinClient();
const token = OdinClient.generateToken(accessKey, "MyRoom", "User1");
const room = client.createRoom(token);

room.onJoined((event) => {
    console.log(`Joined room ${event.roomId}, peer ID: ${event.ownPeerId}`);
});

room.onPeerJoined((event) => {
    console.log(`Peer ${event.userId} joined`);
});

room.onAudioDataReceived((data) => {
    // data.samples32 = Float32Array for processing
    // data.samples16 = Int16Array for WAV recording
    processAudio(data.samples32);
});

await room.join("https://gateway.odin.4players.io");

// Send audio
const media = room.createAudioStream(48000, 1);
await media.sendMP3('./audio.mp3');
media.close();

// Cleanup
room.close();
```

## Core Classes

### OdinClient

```javascript
const client = new OdinClient();
const token = OdinClient.generateToken(accessKey, roomId, userId);
const room = client.createRoom(token);
```

### OdinRoom

```javascript
// Connection
await room.join(gateway, userData?);
room.close();
room.connected;  // boolean
room.ownPeerId;  // number

// Audio streaming
const media = room.createAudioStream(sampleRate, channels);
await media.sendMP3('./audio.mp3');
await media.sendWAV('./audio.wav');
media.sendAudioData(samples);  // Float32Array
media.close();

// Messaging
room.sendMessage(Buffer.from("Hello"));
room.sendMessage(Buffer.from("Hi"), [peerId1, peerId2]);

// 3D Audio
room.setPositionScale(1.0);  // 1 unit = 1 meter
room.updatePosition(x, y, z);

// Diagnostics
const stats = room.getConnectionStats();  // { rtt, udpTxLoss, udpRxLoss, ... }
const jitter = room.getJitterStats(mediaId);  // { packetsLost, packetsTotal, ... }
```

### OdinCipher (E2EE)

```javascript
const cipher = new OdinCipher();
cipher.setPassword(new TextEncoder().encode("shared-secret"));
room.setCipher(cipher);  // Apply BEFORE joining
await room.join(gateway);

// Verify peer encryption
const status = cipher.getPeerStatus(peerId);
// "encrypted", "mismatch", "unencrypted", "unknown"
```

## Event Handling

```javascript
// Room lifecycle
room.onJoined(({ roomId, ownPeerId, mediaIds }) => { });
room.onLeft(({ reason }) => { /* Centralized cleanup */ });

// Peers
room.onPeerJoined(({ peerId, userId, userData }) => { });
room.onPeerLeft(({ peerId }) => { });

// Media
room.onMediaStarted(({ peerId, media }) => { });
room.onMediaStopped(({ peerId, mediaId }) => { });
room.onMediaActivity(({ peerId, mediaId, active }) => { });

// Audio data
room.onAudioDataReceived(({ peerId, mediaId, samples16, samples32 }) => {
    // samples16: Int16Array (WAV)
    // samples32: Float32Array [-1, 1] (processing)
});

// Messages
room.onMessageReceived(({ senderPeerId, message }) => { });

// Connection
room.onConnectionStateChanged(({ state }) => { });
```

## Error Handling

```javascript
room.onConnectionStateChanged((event) => {
    if (event.state === 'disconnected') {
        // Implement reconnection
    }
});

room.onLeft((event) => {
    console.log(`Disconnected: ${event.reason}`);
    // Centralized cleanup
});

try {
    await room.join(gateway);
} catch (error) {
    console.error('Failed to join:', error);
}
```

## Best Practices

1. **Client Lifecycle**: Create one `OdinClient` instance per application (singleton)
2. **Token Security**: Generate tokens server-side with secure access key storage
3. **Encryption**: Enable OdinCipher before joining for E2EE
4. **Event Handling**: Use `onLeft` for centralized cleanup
5. **Audio Streams**: Always call `media.close()` after transmission
6. **Audio Timing**: Respect 20ms chunk requirement (`sampleRate / 50`)

## Use Cases

- **Recording Bots**: Capture and store voice conversations
- **AI Voice Assistants**: Real-time speech-to-text and response generation
- **Content Moderation**: Automated monitoring and filtering
- **Audio Processing**: Real-time effects, transcoding, analysis
- **Bridge Servers**: Connect different voice platforms

## Additional Documentation

- **Audio Patterns**: See [voice-nodejs-audio.md](voice-nodejs-audio.md) for recording bots, AI assistants, multi-room bots

## Resources

- **Docs**: https://docs.4players.io/voice/nodejs/
- **NPM**: https://www.npmjs.com/package/@4players/odin-nodejs
- **GitHub**: https://github.com/4Players/odin-nodejs
- **Access Keys**: https://www.4players.io/odin/
