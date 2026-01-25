---
name: odin-voice-nodejs
description: |
  ODIN Voice NodeJS SDK - Server-side voice chat SDK with native C++ bindings.
  Use when: building recording bots, AI voice assistants, server-side audio processing,
  content moderation systems, or any Node.js application requiring real-time voice chat.
  Enables high-quality real-time voice communication with end-to-end encryption,
  3D positional audio, and raw PCM audio access for processing.
---

# ODIN Voice NodeJS SDK

Server-side voice chat SDK with native C++ bindings for building recording bots, AI assistants, and audio processing applications.

## Overview

The NodeJS SDK provides:
- **Native C++ bindings** for high-performance server-side applications
- **Raw PCM audio access** for recording, processing, and AI integration
- **End-to-End Encryption** with OdinCipher
- **3D positional audio** support for spatial voice
- **Real-time diagnostics** for connection and audio quality monitoring
- **Built on ODIN Core SDK** (same foundation as Unity, Unreal, Web, Swift)
- **Flexible deployment** - use 4Players cloud or self-hosted infrastructure

## Supported Platforms

| Platform | x86_64 | ARM64 |
|----------|--------|-------|
| macOS    | ✅     | ✅    |
| Windows  | ✅     | —     |
| Linux    | ✅     | —     |

Other platforms require building from source with C/C++ compilers.

## Installation

**Prerequisites**:
- Node.js 14+ (LTS recommended)
- Build tools for native modules

**Install via npm**:
```bash
npm install @4players/odin-nodejs
```

**Build Requirements by Platform**:
- **macOS**: `xcode-select --install`
- **Ubuntu/Debian**: `sudo apt-get install build-essential python3`
- **Windows**: `npm install --global windows-build-tools`

**IMPORTANT**: Cannot combine Web SDK (`@4players/odin`) and NodeJS SDK (`@4players/odin-nodejs`) in the same project—choose one.

## Quick Start

```javascript
const { OdinClient, OdinCipher } = require('@4players/odin-nodejs');

// 1. Create client instance (singleton for application lifetime)
const client = new OdinClient();

// 2. Generate authentication token
const accessKey = "YOUR_ACCESS_KEY"; // Get from 4players.io
const token = OdinClient.generateToken(accessKey, "MyRoom", "User1");

// 3. Create and join room
const room = client.createRoom(token);

// 4. Set up event handlers
room.onJoined((event) => {
    console.log(`Joined room ${event.roomId}, peer ID: ${event.ownPeerId}`);

    // Optional: Enable 3D audio
    room.setPositionScale(1.0); // 1 unit = 1 meter
    room.updatePosition(0, 0, 0);
});

room.onPeerJoined((event) => {
    console.log(`Peer ${event.userId} joined (ID: ${event.peerId})`);
});

room.onMediaStarted((event) => {
    console.log(`Media started from peer ${event.peerId}`);
});

room.onAudioDataReceived((data) => {
    const { peerId, mediaId, samples16, samples32 } = data;
    // Process audio: samples16 (WAV recording) or samples32 (processing)
    console.log(`Received ${samples32.length} audio samples from peer ${peerId}`);
});

// 5. Join the room
await room.join("https://gateway.odin.4players.io");

// 6. Optional: Send audio
const media = room.createAudioStream(48000, 2); // 48kHz, stereo
await media.sendMP3('./audio.mp3');
media.close();

// 7. Cleanup
room.onLeft((event) => {
    console.log(`Left room: ${event.reason}`);
    // Centralized cleanup
});

// Later: disconnect
room.close();
```

## Architecture & Key Concepts

### Client Lifecycle (Singleton Pattern)
- Create **one** `OdinClient` instance for entire application lifetime
- Use it to create multiple room instances
- Each room represents a separate voice chat session

### Authentication & Security
- **JWT-based authentication** using signed tokens
- Tokens contain room ID (`rid`) and user ID (`uid`)
- **CRITICAL**: Access keys are sensitive credentials - never embed in client-side code
- **Production deployment**: Store access keys securely, generate tokens server-side
- **Local token generation**: Use `OdinClient.generateToken()` for server applications

### End-to-End Encryption
- Enable **OdinCipher** for E2EE before joining room
- All participants must use the same password
- Verify peer encryption status with `cipher.getPeerStatus(peerId)`

### Audio Handling
- **Receive audio**: 20ms chunks at configured sample rate
- **Two formats**: `samples16` (16-bit int for WAV) and `samples32` (32-bit float for processing)
- **Send audio**: High-level API (MP3/WAV) or low-level API (raw PCM)
- **20ms timing requirement**: `chunkLength = sampleRate / 50`

## Core Classes

### OdinClient

Factory for creating rooms and generating tokens.

```javascript
const client = new OdinClient();

// Generate authentication token
const token = OdinClient.generateToken(
    accessKey,  // Your 4Players access key
    roomId,     // Room identifier
    userId      // User identifier
);

// Create room instance
const room = client.createRoom(token);
```

### OdinRoom

Main interface for room management and communication.

**Connection Management**:
```javascript
// Join room
await room.join(gateway, userData?);

// Disconnect
room.close();

// Check connection status
const connected = room.connected;
const peerId = room.ownPeerId;
const connectionId = room.getConnectionId();
```

**Audio Streaming**:
```javascript
// Create audio output stream
const media = room.createAudioStream(sampleRate, channels);

// High-level: Send audio files
await media.sendMP3('./audio.mp3');
await media.sendWAV('./audio.wav');

// Low-level: Send raw PCM samples
media.setMediaId(mediaId);
media.sendAudioData(samples);

// Always close when done
media.close();
```

**Messaging**:
```javascript
// Send to all peers
room.sendMessage(Buffer.from("Hello"));

// Send to specific peers
room.sendMessage(Buffer.from("Hi"), [peerId1, peerId2]);
```

**3D Positional Audio**:
```javascript
// Set coordinate scale (1.0 = 1 meter per unit)
room.setPositionScale(1.0);

// Update position (x, y, z)
room.updatePosition(10.0, 0.0, 5.0);
```

**Diagnostics**:
```javascript
// Connection statistics
const stats = room.getConnectionStats();
// Returns: { rtt, udpTxLoss, udpRxLoss, udpTxBytes, udpRxBytes, congestionEvents }

// Audio quality metrics
const jitter = room.getJitterStats(mediaId);
// Returns: { packetsTotal, packetsLost, packetsArrivedTooLate }

// Available media streams
const mediaIds = room.availableMediaIds;
```

**Low-Level RPC**:
```javascript
// Send custom RPC commands (advanced use)
room.sendRpc(rpcData);
```

### OdinMedia

Audio stream interface for sending audio data.

**High-Level API (Recommended)**:
```javascript
const media = room.createAudioStream(44100, 2);
await media.sendMP3('./music.mp3');
await media.sendWAV('./audio.wav');
media.close(); // Automatically sends StopMedia RPC
```

**Low-Level API**:
```javascript
const media = room.createAudioStream(48000, 1);

// Assign media ID from room
media.setMediaId(mediaId);

// Stream audio in 20ms chunks
const chunkSize = 48000 / 50; // 960 samples for 48kHz
media.sendAudioData(audioSamples); // Float32Array

// Close when finished
media.close();
```

### OdinCipher

End-to-end encryption interface.

```javascript
const cipher = new OdinCipher();

// Set shared password (must match across all peers)
cipher.setPassword(new TextEncoder().encode("shared-secret"));

// Apply to room BEFORE joining
room.setCipher(cipher);

// Verify peer encryption status
const status = cipher.getPeerStatus(peerId);
// Returns: "encrypted", "mismatch", "unencrypted", "unknown"
```

## Event Handling

### Convenience Methods (Recommended)

```javascript
// Room lifecycle
room.onJoined((event) => {
    // { roomId, ownPeerId, room, mediaIds }
    console.log(`Joined room ${event.roomId}, own peer ID: ${event.ownPeerId}`);
});

room.onLeft((event) => {
    // { reason }
    console.log(`Left room: ${event.reason}`);
    // Centralized cleanup (handles both normal and error disconnects)
});

// Peer management
room.onPeerJoined((event) => {
    // { peerId, userId, userData, peer }
    console.log(`Peer ${event.userId} joined with ID ${event.peerId}`);
});

room.onPeerLeft((event) => {
    // { peerId }
    console.log(`Peer ${event.peerId} left`);
});

// Media streams
room.onMediaStarted((event) => {
    // { peerId, media }
    console.log(`Media stream started from peer ${event.peerId}`);
});

room.onMediaStopped((event) => {
    // { peerId, mediaId }
    console.log(`Media stream ${event.mediaId} stopped`);
});

room.onMediaActivity((event) => {
    // Voice Activity Detection
    // { peerId, mediaId, active }
    console.log(`Peer ${event.peerId} is ${event.active ? 'speaking' : 'silent'}`);
});

// Audio data
room.onAudioDataReceived((data) => {
    // { peerId, mediaId, samples16, samples32 }
    // samples16: Int16Array (for WAV recording)
    // samples32: Float32Array, range [-1, 1] (for processing)
    processAudio(data.samples32);
});

// Messages
room.onMessageReceived((event) => {
    // { senderPeerId, message }
    console.log(`Message from ${event.senderPeerId}: ${event.message}`);
});

// Connection
room.onConnectionStateChanged((event) => {
    console.log(`Connection state changed: ${event.state}`);
});
```

### Alternative addEventListener Pattern

```javascript
room.addEventListener('PeerJoined', (event) => {
    console.log(`Peer joined: ${event.peerId}`);
});

room.addEventListener('AudioDataReceived', (data) => {
    processAudio(data.samples32);
});
```

**Available Events**: Joined, Left, PeerJoined, PeerLeft, MessageReceived, MediaStarted, MediaStopped, MediaActivity, AudioDataReceived, PeerTagsChanged, ConnectionStateChanged

## Audio Processing Patterns

### Recording Bot

```javascript
const fs = require('fs');
const WavEncoder = require('wav').Writer;

room.onJoined((event) => {
    console.log(`Recording bot joined room ${event.roomId}`);
});

// Record all peers to separate WAV files
const recordings = new Map();

room.onMediaStarted((event) => {
    const filename = `peer_${event.peerId}_${Date.now()}.wav`;
    const writer = new WavEncoder({
        sampleRate: 48000,
        channels: 1,
        bitDepth: 16
    });
    writer.pipe(fs.createWriteStream(filename));
    recordings.set(event.media, writer);
    console.log(`Recording ${filename}`);
});

room.onAudioDataReceived((data) => {
    const writer = recordings.get(data.mediaId);
    if (writer) {
        // Convert Float32Array to Int16Array
        const buffer = Buffer.alloc(data.samples16.length * 2);
        for (let i = 0; i < data.samples16.length; i++) {
            buffer.writeInt16LE(data.samples16[i], i * 2);
        }
        writer.write(buffer);
    }
});

room.onMediaStopped((event) => {
    const writer = recordings.get(event.mediaId);
    if (writer) {
        writer.end();
        recordings.delete(event.mediaId);
        console.log(`Recording finished for media ${event.mediaId}`);
    }
});
```

### AI Voice Assistant

```javascript
const { OdinClient } = require('@4players/odin-nodejs');

// AI assistant that responds to voice
room.onAudioDataReceived(async (data) => {
    // 1. Transcribe audio
    const text = await speechToText(data.samples32);

    // 2. Generate AI response
    const response = await aiModel.generate(text);

    // 3. Synthesize and play response
    const audioResponse = await textToSpeech(response);
    const media = room.createAudioStream(48000, 1);
    media.sendAudioData(audioResponse);
    media.close();
});
```

### Audio Processing Pipeline

```javascript
// Apply real-time audio effects
room.onAudioDataReceived((data) => {
    // Access 32-bit float samples for processing
    let processed = data.samples32;

    // Apply effects
    processed = applyCompressor(processed);
    processed = applyEqualizer(processed);
    processed = applyReverb(processed);

    // Forward to output/storage
    outputDevice.write(processed);
});
```

### Multi-Room Bot

```javascript
// Bot that bridges multiple rooms
const client = new OdinClient();

const room1 = client.createRoom(token1);
const room2 = client.createRoom(token2);

await room1.join("https://gateway.odin.4players.io");
await room2.join("https://gateway.odin.4players.io");

// Forward audio from room1 to room2
room1.onAudioDataReceived((data) => {
    const media = room2.createAudioStream(48000, 1);
    media.sendAudioData(data.samples32);
    media.close();
});

// Forward audio from room2 to room1
room2.onAudioDataReceived((data) => {
    const media = room1.createAudioStream(48000, 1);
    media.sendAudioData(data.samples32);
    media.close();
});
```

## End-to-End Encryption

Enable E2EE before joining room:

```javascript
const { OdinCipher } = require('@4players/odin-nodejs');

// Create cipher
const cipher = new OdinCipher();
cipher.setPassword(new TextEncoder().encode("my-secret-password"));

// Apply to room
room.setCipher(cipher);

// Join room (encryption now active)
await room.join("https://gateway.odin.4players.io");

// Verify peer encryption
room.onPeerJoined((event) => {
    const status = cipher.getPeerStatus(event.peerId);
    console.log(`Peer ${event.peerId} encryption: ${status}`);
    // "encrypted" = secure communication
    // "mismatch" = wrong password
    // "unencrypted" = peer not using encryption
    // "unknown" = status not yet determined
});
```

**CRITICAL**: All room participants must use the **exact same password** for encryption to work.

## 3D Positional Audio (Proximity Chat)

```javascript
room.onJoined(() => {
    // Define scale: 1 unit = 1 meter
    room.setPositionScale(1.0);

    // Set initial position (x, y, z)
    room.updatePosition(10.0, 0.0, 5.0);
});

// Update position as entity moves
function onEntityMove(x, y, z) {
    room.updatePosition(x, y, z);
}

// Peers within unit circle radius 1.0 remain visible to each other
// Audio volume decreases with distance automatically
```

## Connection Monitoring

```javascript
// Monitor connection quality
setInterval(() => {
    const stats = room.getConnectionStats();
    console.log(`RTT: ${stats.rtt}ms`);
    console.log(`TX Loss: ${stats.udpTxLoss}%`);
    console.log(`RX Loss: ${stats.udpRxLoss}%`);
    console.log(`Congestion: ${stats.congestionEvents}`);

    // Check audio quality per media stream
    room.availableMediaIds.forEach(mediaId => {
        const jitter = room.getJitterStats(mediaId);
        console.log(`Media ${mediaId} - Lost: ${jitter.packetsLost}/${jitter.packetsTotal}`);
    });
}, 5000); // Every 5 seconds
```

## Error Handling

```javascript
room.onConnectionStateChanged((event) => {
    if (event.state === 'disconnected') {
        console.error('Disconnected from room');
        // Implement reconnection logic
    }
});

room.onLeft((event) => {
    console.log(`Room disconnected: ${event.reason}`);
    // Handles both normal disconnects and errors
    // Centralized cleanup location
});

// Catch join errors
try {
    await room.join("https://gateway.odin.4players.io");
} catch (error) {
    console.error('Failed to join room:', error);
}
```

## Common Patterns

### Audio Timing (20ms Chunks)

ODIN requires audio in 20ms intervals (50 times per second):

```javascript
const sampleRate = 48000;
const channels = 1;
const chunkSize = sampleRate / 50; // 960 samples = 20ms at 48kHz

const media = room.createAudioStream(sampleRate, channels);

// Stream audio in correct timing
setInterval(() => {
    const chunk = getNextAudioChunk(chunkSize); // Float32Array
    media.sendAudioData(chunk);
}, 20); // Every 20ms
```

### Graceful Shutdown

```javascript
process.on('SIGINT', async () => {
    console.log('Shutting down...');

    // Close all active media streams
    activeMediaStreams.forEach(media => media.close());

    // Disconnect from room
    room.close();

    // Wait for clean disconnect
    await new Promise(resolve => {
        room.onLeft(() => resolve());
        setTimeout(resolve, 1000); // Timeout after 1s
    });

    process.exit(0);
});
```

### Resource Management

```javascript
// Track active resources
const activeMedia = new Set();

room.onJoined((event) => {
    event.mediaIds.forEach(id => activeMedia.add(id));
});

room.onMediaStarted((event) => {
    activeMedia.add(event.media);
});

room.onMediaStopped((event) => {
    activeMedia.delete(event.mediaId);
});

room.onLeft(() => {
    // Cleanup all resources
    activeMedia.clear();
});
```

## Best Practices

1. **Client Lifecycle**: Create **one** `OdinClient` instance per application (singleton pattern)
2. **Token Security**: Generate tokens server-side with secure access key storage
3. **Encryption**: Enable OdinCipher before joining for E2EE
4. **Event Handling**: Use `onLeft` for centralized cleanup (handles both normal and error disconnects)
5. **Audio Streams**: Always call `media.close()` after transmission to release resources
6. **Audio Timing**: Respect 20ms chunk requirement (`sampleRate / 50`)
7. **API Choice**: Use high-level API (`sendMP3`/`sendWAV`) unless you need fine-grained control
8. **Error Handling**: Implement reconnection logic for production applications
9. **Diagnostics**: Monitor connection stats and jitter for quality assurance
10. **Resource Management**: Track and cleanup media streams properly

## Use Cases

- **Recording Bots**: Capture and store voice conversations
- **AI Voice Assistants**: Real-time speech-to-text and response generation
- **Content Moderation**: Automated monitoring and filtering
- **Audio Processing**: Real-time effects, transcoding, analysis
- **Bridge Servers**: Connect different voice platforms
- **Quality Monitoring**: Analyze call quality and metrics
- **Testing & Automation**: Automated voice chat testing

## Architecture Notes

- **Built on ODIN Core SDK** using native C++ bindings
- **Event-driven via RPC** with MessagePack encoding
- **JWT authentication** with Ed25519 signing
- **Opus codec compression** with automatic resampling
- **Connection pool pattern** for efficient connection management
- **No server-side user data storage** required

## Documentation & Resources

- **Full Documentation**: https://docs.4players.io/voice/nodejs/
- **NPM Package**: https://www.npmjs.com/package/@4players/odin-nodejs
- **GitHub Repository**: https://github.com/4Players/odin-nodejs
- **4Players ODIN Portal**: https://www.4players.io/odin/ (Access keys)
- **Core SDK**: Built on the same foundation as Unity, Unreal, Web, Swift SDKs
- **Support**: Discord, Email (support@4players.io), GitHub Issues

## Getting Access Keys

1. Sign up at https://www.4players.io/odin/
2. Create a new app in the dashboard
3. Copy your access key
4. Use for token generation in your Node.js application

**Security Note**: Store access keys securely (environment variables, secret managers) - never commit to version control.
