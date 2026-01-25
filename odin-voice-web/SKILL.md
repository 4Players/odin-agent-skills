---
name: odin-voice-web
description: |
  ODIN Voice Web SDK - Browser SDK for real-time voice chat with WebTransport/HTTP3.
  Use when: implementing voice chat in web apps, browsers, or web-based applications,
  working with Room/Peer/AudioInput/AudioOutput classes, handling audio events,
  configuring audio input/output devices, or integrating with React/Angular/Vue frameworks.
  Supports NPM and CDN. Enables seamless voice communication in multiplayer games,
  social platforms, collaboration tools, and interactive web experiences.
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

**Pinned Versions** (for production stability):
```html
<!-- Replace 'latest' with specific version numbers -->
<script src="https://cdn.odin.4players.io/client/js/sdk/1.0.1/odin-sdk.js"></script>
<script src="https://cdn.odin.4players.io/client/js/sdk/1.1.0/odin-plugin.js"></script>
```

## Quick Start

### Complete Implementation

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
    // Initialize plugin first
    await initPlugin();

    // Create room instance
    room = new ODIN.Room();

    // Register event handlers BEFORE joining
    setupEventHandlers(room);

    // Join with token from your backend
    await room.join(token, { gateway: 'https://gateway.odin.4players.io' });

    // Configure output device (speakers/headphones)
    await audioPlugin.setOutputDevice({});

    // Add microphone input
    audioInput = await ODIN.DeviceManager.createAudioInput(null, {
        echoCanceller: true,
        noiseSuppression: true,
        gainController: true,
    });
    await room.addAudioInput(audioInput);

    console.log('Successfully joined voice room!');
}

// Step 3: Setup event handlers
function setupEventHandlers(room) {
    // Room lifecycle
    room.onJoined = (payload) => {
        console.log(`Joined room! Own peer ID: ${payload.ownPeer.id}`);
    };

    room.onLeft = (payload) => {
        console.log('Left room');
    };

    // Peer management
    room.onPeerJoined = (payload) => {
        console.log(`Peer joined: ${payload.peer.userId}`);
    };

    room.onPeerLeft = (payload) => {
        console.log(`Peer left: ${payload.peer.userId}`);
    };

    // Audio events
    room.onAudioOutputStarted = (payload) => {
        console.log(`Audio started from: ${payload.peer.userId}`);
    };

    room.onAudioOutputStopped = (payload) => {
        console.log(`Audio stopped from: ${payload.peer.userId}`);
    };
}

// Step 4: Leave room when done
async function leaveRoom() {
    if (audioInput) {
        await room.removeAudioInput(audioInput);
        audioInput.close();
        audioInput = null;
    }

    if (room) {
        room.leave();
        room = null;
    }
}

// Usage: Call from button click
document.getElementById('joinBtn').addEventListener('click', async () => {
    // Fetch token from your backend
    const response = await fetch('/api/odin-token');
    const { token } = await response.json();

    await joinRoom(token);
});

document.getElementById('leaveBtn').addEventListener('click', leaveRoom);
```

### Critical Requirements

**User Interaction**: Modern browsers require direct user interaction (click/tap) to start AudioContext. Always call `initPlugin()` and audio setup from user action handlers.

**Event Registration**: Register all event handlers BEFORE calling `room.join()` to avoid missing early events.

**Audio Activation**: Audio is only transmitted after calling `room.addAudioInput(audioInput)`. Simply creating an AudioInput is insufficient.

## Core Classes & API Reference

### Room

Central hub for voice communication and primary interface for managing connections.

**Properties**:
```javascript
room.id                 // String: Room identifier
room.ownPeerId          // Number: Local peer ID
room.ownPeer            // LocalPeer: Your own peer instance
room.remotePeers        // Map<number, RemotePeer>: Connected peers
room.status             // String: 'idle' | 'connecting' | 'connected' | 'disconnected'
room.connectionStats    // Object: Network statistics (RTT, packet loss, etc.)
room.position           // Array: [x, y, z] for 3D positional audio
room.userData           // Uint8Array: Room-level custom data
```

**Methods**:
```javascript
// Connection management
await room.join(token, options?)
// token: JWT token from your backend
// options: { gateway?: string }

room.leave()
// Disconnects from room

// Audio management
await room.addAudioInput(audioInput)
// Attaches microphone to room (enables transmission)

await room.removeAudioInput(audioInput)
// Removes microphone from room

room.setVolume(value)
// value: 0-2 (0=mute, 1=normal, 2=boost) or "muted"
// Sets master output volume

await room.setAudioOutputDevice(device?)
// device: MediaDeviceInfo or undefined for default
// Selects speaker/headphones

// 3D Positional audio
await room.setPosition(x, y, z)
// Updates position for proximity-based culling

// Messaging
await room.sendMessage(message, targetPeerIds?)
// message: Uint8Array
// targetPeerIds: Array<number> (omit to broadcast to all)

// User data
room.flushUserData()
// Sends updated userData to all peers
```

### LocalPeer

Represents your own user in the room.

**Properties**:
```javascript
localPeer.id            // Number: Your peer ID
localPeer.userId        // String: User identifier from token
localPeer.isRemote      // Boolean: Always false
localPeer.data          // Uint8Array: Custom user data
localPeer.audioInputs   // Array<AudioInput>: Active microphones
```

**Methods**:
```javascript
localPeer.setVolume(value)
// value: 0-2 or "muted"
// Sets volume for local peer

localPeer.update()
// Flush user data changes to server
```

### RemotePeer

Represents other users in the room.

**Properties**:
```javascript
remotePeer.id           // Number: Peer ID
remotePeer.userId       // String: User identifier from token
remotePeer.isRemote     // Boolean: Always true
remotePeer.data         // Uint8Array: Custom user data
remotePeer.audioOutputs // Array<AudioOutput>: Active audio streams
```

**Methods**:
```javascript
remotePeer.setVolume(value)
// value: 0-2 or "muted"
// Sets playback volume for this peer

await remotePeer.sendMessage(message)
// message: Uint8Array
// Send direct message to this peer
```

### AudioInput

Manages microphone capture and transmission.

**Creation**:
```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput(device?, settings?, customType?);
// device: MediaDeviceInfo or null for default microphone
// settings: Audio processing configuration (see Audio Input Settings below)
// customType: Optional custom type identifier
```

**Properties**:
```javascript
audioInput.volume       // Number: Capture volume (0-2)
audioInput.isActive     // Boolean: Currently capturing audio
audioInput.powerLevel   // Number: RMS power level in dBFS
```

**Methods**:
```javascript
audioInput.setVolume(value)
// value: 0-2 (0=mute, 1=normal, 2=boost)

await audioInput.setDevice(device)
// device: MediaDeviceInfo
// Switch microphone device

await audioInput.setInputSettings(settings)
// settings: Partial audio processing configuration
// Update APM settings dynamically

audioInput.close()
// Release microphone resources

audioInput.startLoopback() / audioInput.stopLoopback()
// Enable/disable local audio monitoring
```

**Events**:
```javascript
audioInput.onAudioActivity = (payload) => {
    console.log(payload.media.isActive); // Voice activity detected
};
```

### AudioOutput

Handles playback of remote peer audio.

**Properties**:
```javascript
audioOutput.peer        // RemotePeer: Associated remote peer
audioOutput.mediaId     // Number: Media stream identifier
audioOutput.volume      // Number: Playback volume (0-2)
audioOutput.isActive    // Boolean: Currently playing
audioOutput.isPaused    // Boolean: Pause state
```

**Methods**:
```javascript
audioOutput.setVolume(value?)
// value: 0-2 or undefined for default
// Sets playback volume for this stream

audioOutput.pause()
// Pause playback

audioOutput.resume()
// Resume playback

audioOutput.close()
// Stop and release audio stream
```

### DeviceManager

Static utility class for managing audio/video devices.

**Device Enumeration**:
```javascript
await ODIN.DeviceManager.listAudioInputs()
// Returns: Promise<MediaDeviceInfo[]> - Available microphones

await ODIN.DeviceManager.listAudioOutputs()
// Returns: Promise<MediaDeviceInfo[]> - Available speakers/headphones

await ODIN.DeviceManager.listDevices(kind?)
// kind: 'audioinput' | 'audiooutput' | 'videoinput' | undefined
// Returns: Promise<MediaDeviceInfo[]> - Filtered or all devices
```

**Create Inputs**:
```javascript
await ODIN.DeviceManager.createAudioInput(device?, settings?, customType?)
// device: MediaDeviceInfo or null for default
// settings: AudioInputSettings configuration
// customType: Optional custom identifier
// Returns: Promise<AudioInput>

await ODIN.DeviceManager.createVideoInput(mediaStream, options?)
// mediaStream: MediaStream from getUserMedia
// options: { customType?: string }
// Returns: Promise<VideoInput>
```

**Find Specific Device**:
```javascript
await ODIN.DeviceManager.getInputDevice(deviceId)
// deviceId: string
// Returns: Promise<MediaDeviceInfo>

await ODIN.DeviceManager.getOutputDevice(deviceId)
// deviceId: string
// Returns: Promise<MediaDeviceInfo>
```

**Important Notes**:
- Firefox requires requesting MediaStream once before device enumeration works
- Always request microphone permission before listing devices
- Device IDs may change between sessions (use labels for persistence)

## Event System

ODIN uses an event-driven architecture. Events can be handled in two ways:

### Event Handling Patterns

**Pattern 1: Property Assignment** (overwrites previous handler):
```javascript
room.onPeerJoined = (payload) => {
    console.log('Peer joined:', payload.peer.userId);
};
```

**Pattern 2: addEventListener** (can register multiple handlers):
```javascript
room.addEventListener('PeerJoined', (event) => {
    console.log('Peer joined:', event.detail.peer.userId);
});

// Remove listener when needed
const handler = (event) => { /* ... */ };
room.addEventListener('PeerJoined', handler);
room.removeEventListener('PeerJoined', handler);
```

**IMPORTANT**: Register event handlers BEFORE calling `room.join()` to avoid missing early events.

### Room Events Reference

**Lifecycle Events**:
```javascript
room.onJoined = (payload) => {
    // payload: { ownPeer, room, mediaIds }
    console.log('Successfully joined room');
    console.log('Own peer ID:', payload.ownPeer.id);
    console.log('Existing media streams:', payload.mediaIds);
};

room.onLeft = (payload) => {
    // payload: { room }
    console.log('Left room');
    // Cleanup resources here
};

room.onStatusChanged = (payload) => {
    // payload: { status }
    // status: 'idle' | 'connecting' | 'connected' | 'disconnected'
    console.log('Connection status:', payload.status);
};
```

**Peer Events**:
```javascript
room.onPeerJoined = (payload) => {
    // payload: { peer }
    console.log(`Peer ${payload.peer.userId} joined (ID: ${payload.peer.id})`);
    console.log('User data:', ODIN.uint8ArrayToValue(payload.peer.data));
};

room.onPeerLeft = (payload) => {
    // payload: { peer }
    console.log(`Peer ${payload.peer.userId} left`);
};

room.onUserDataChanged = (payload) => {
    // payload: { peer }
    console.log('Peer data updated:', payload.peer.userId);
    const userData = ODIN.uint8ArrayToValue(payload.peer.data);
    console.log('New data:', userData);
};
```

**Audio Events**:
```javascript
room.onAudioOutputStarted = (payload) => {
    // payload: { peer, media }
    console.log(`Audio started from ${payload.peer.userId}`);
    console.log('Media ID:', payload.media.mediaId);
};

room.onAudioOutputStopped = (payload) => {
    // payload: { peer, media }
    console.log(`Audio stopped from ${payload.peer.userId}`);
};

room.onAudioInputStarted = (payload) => {
    // payload: { media }
    console.log('Local microphone activated');
};

room.onAudioInputStopped = (payload) => {
    // payload: { media }
    console.log('Local microphone deactivated');
};
```

**Voice Activity & Power Level**:
```javascript
room.onAudioActivity = (payload) => {
    // payload: { peer, media, active }
    const status = payload.active ? 'speaking' : 'silent';
    console.log(`${payload.peer.userId} is ${status}`);
};

room.onAudioPowerLevel = (payload) => {
    // payload: { peer, media, powerLevel }
    // powerLevel: RMS in dBFS (negative values)
    updateVoiceIndicator(payload.peer.id, payload.powerLevel);
};
```

**Data Messages**:
```javascript
room.onMessageReceived = (payload) => {
    // payload: { peer, message }
    const data = ODIN.uint8ArrayToValue(payload.message);
    console.log(`Message from ${payload.peer.userId}:`, data);
};
```

**Network Statistics**:
```javascript
room.onConnectionStats = (payload) => {
    // payload: { stats }
    console.log('RTT:', payload.stats.rtt);
    console.log('Packet loss:', payload.stats.packetLoss);
};
```

## Authentication & Token Generation

ODIN uses JWT-based authentication. Tokens must be generated server-side to protect your access key.

### Backend Token Generation

**Install the token generator** (Node.js backend):
```bash
npm install @4players/odin-tokens
```

**Generate tokens** (Express.js example):
```javascript
const express = require('express');
const { TokenGenerator } = require('@4players/odin-tokens');

const app = express();
const accessKey = process.env.ODIN_ACCESS_KEY; // Store securely!

app.get('/api/odin-token', (req, res) => {
    const generator = new TokenGenerator(accessKey);

    const token = generator.createToken(
        'my-room-id',        // Room identifier
        req.user.id,         // User identifier
        { lifetime: 300 }    // Optional: 5 minutes TTL
    );

    res.json({ token });
});

app.listen(3000);
```

**Token Options**:
```javascript
const token = generator.createToken(roomId, userId, {
    lifetime: 300,        // Seconds until expiration (optional)
    customData: {}        // Additional claims (optional)
});
```

### Frontend Token Usage

**Fetch and use token**:
```javascript
async function joinVoiceRoom(roomId) {
    // Fetch token from your backend
    const response = await fetch(`/api/odin-token?room=${roomId}`);
    const { token } = await response.json();

    // Join room with token
    await room.join(token, {
        gateway: 'https://gateway.odin.4players.io'
    });
}
```

### Security Best Practices

- **NEVER** embed your ODIN access key in client-side code
- **ALWAYS** generate tokens on your backend server
- **Store** access keys in environment variables or secret managers
- **Validate** user authentication before issuing ODIN tokens
- **Set** appropriate token lifetimes for your use case
- **Use** HTTPS for all token exchanges

### Gateway Options

**Default (EU)**:
```javascript
await room.join(token); // Uses EU gateway by default
```

**Specify Region**:
```javascript
await room.join(token, {
    gateway: 'https://gateway.odin.4players.io'      // EU
    // gateway: 'https://gateway-us.odin.4players.io' // US
});
```

## Audio Processing Module (APM) Settings

Configure advanced audio processing for optimal voice quality.

### Complete Configuration

```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput(null, {
    // Volume control
    volume: 1,                  // 0-2 (0=mute, 1=normal, 2=boost)

    // Acoustic Echo Cancellation (AEC)
    echoCanceller: true,        // Remove speaker echo from microphone

    // Noise Suppression
    noiseSuppression: true,     // Remove background noise

    // Automatic Gain Control (AGC)
    gainController: true,       // Normalize volume levels

    // Voice Activity Detection (VAD)
    voiceActivity: {
        attackThreshold: 0.9,   // 0-1: Probability to start speaking (0.9 = 90%)
        releaseThreshold: 0.8,  // 0-1: Probability to stop speaking (0.8 = 80%)
    },

    // Volume Gate (noise gate)
    volumeGate: {
        attackThreshold: -30,   // dBFS: Level to open gate (start transmitting)
        releaseThreshold: -40,  // dBFS: Level to close gate (stop transmitting)
    }
});
```

### Update Settings Dynamically

```javascript
// Update individual settings
await audioInput.setInputSettings({
    noiseSuppression: false,
    volumeGate: { attackThreshold: -25 }
});

// Toggle echo cancellation
await audioInput.setInputSettings({
    echoCanceller: !currentSettings.echoCanceller
});
```

### Recommended Presets

**Gaming (Low Latency)**:
```javascript
{
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.85,
        releaseThreshold: 0.75,
    },
    volumeGate: {
        attackThreshold: -35,
        releaseThreshold: -45,
    }
}
```

**Podcasting (High Quality)**:
```javascript
{
    echoCanceller: false,       // Use hardware echo cancellation
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.95,
        releaseThreshold: 0.90,
    },
    volumeGate: {
        attackThreshold: -25,
        releaseThreshold: -35,
    }
}
```

**Noisy Environment**:
```javascript
{
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.95,  // Higher threshold = less false positives
        releaseThreshold: 0.90,
    },
    volumeGate: {
        attackThreshold: -20,   // Higher gate = blocks more noise
        releaseThreshold: -30,
    }
}
```

### Understanding dBFS Values

- **0 dBFS**: Maximum possible level (digital clipping)
- **-6 dBFS**: Loud speaking
- **-12 dBFS**: Normal speaking
- **-20 dBFS**: Quiet speaking
- **-30 dBFS**: Whisper
- **-40 dBFS**: Very quiet background noise
- **-50+ dBFS**: Silence/noise floor

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

## Framework Integration

### React

```typescript
import React, { useState, useEffect, useRef } from 'react';
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

function VoiceChat() {
    const [isConnected, setIsConnected] = useState(false);
    const [peers, setPeers] = useState<Map<number, any>>(new Map());
    const [isMuted, setIsMuted] = useState(false);
    const roomRef = useRef<ODIN.Room | null>(null);
    const audioInputRef = useRef<any>(null);
    const pluginRef = useRef<any>(null);

    // Initialize plugin once
    useEffect(() => {
        const initPlugin = async () => {
            if (pluginRef.current) return;

            pluginRef.current = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
                const audioContext = new AudioContext({ sampleRate });
                await audioContext.resume();
                return audioContext;
            });

            ODIN.init(pluginRef.current);
        };

        initPlugin();
    }, []);

    // Cleanup on unmount
    useEffect(() => {
        return () => {
            if (roomRef.current) {
                roomRef.current.leave();
            }
            if (audioInputRef.current) {
                audioInputRef.current.close();
            }
        };
    }, []);

    const joinRoom = async (token: string) => {
        // Create room
        const room = new ODIN.Room();
        roomRef.current = room;

        // Setup events
        room.onJoined = () => setIsConnected(true);
        room.onLeft = () => setIsConnected(false);

        room.onPeerJoined = (payload) => {
            setPeers(prev => new Map(prev).set(payload.peer.id, payload.peer));
        };

        room.onPeerLeft = (payload) => {
            setPeers(prev => {
                const next = new Map(prev);
                next.delete(payload.peer.id);
                return next;
            });
        };

        // Join room
        await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
        await pluginRef.current.setOutputDevice({});

        // Add microphone
        const audioInput = await ODIN.DeviceManager.createAudioInput(null, {
            echoCanceller: true,
            noiseSuppression: true,
            gainController: true,
        });
        audioInputRef.current = audioInput;
        await room.addAudioInput(audioInput);
    };

    const leaveRoom = async () => {
        if (audioInputRef.current) {
            await roomRef.current?.removeAudioInput(audioInputRef.current);
            audioInputRef.current.close();
            audioInputRef.current = null;
        }

        if (roomRef.current) {
            roomRef.current.leave();
            roomRef.current = null;
        }
    };

    const toggleMute = async () => {
        if (audioInputRef.current) {
            const newVolume = isMuted ? 1 : 0;
            audioInputRef.current.setVolume(newVolume);
            setIsMuted(!isMuted);
        }
    };

    return (
        <div>
            {!isConnected ? (
                <button onClick={() => joinRoom(/* fetch token */)}>
                    Join Voice Chat
                </button>
            ) : (
                <>
                    <button onClick={leaveRoom}>Leave</button>
                    <button onClick={toggleMute}>
                        {isMuted ? 'Unmute' : 'Mute'}
                    </button>
                    <div>
                        <h3>Peers in room: {peers.size}</h3>
                        {Array.from(peers.values()).map(peer => (
                            <div key={peer.id}>{peer.userId}</div>
                        ))}
                    </div>
                </>
            )}
        </div>
    );
}

export default VoiceChat;
```

### Vue 3 (Composition API)

```typescript
<template>
  <div>
    <button v-if="!isConnected" @click="joinRoom">Join Voice Chat</button>
    <template v-else>
      <button @click="leaveRoom">Leave</button>
      <button @click="toggleMute">{{ isMuted ? 'Unmute' : 'Mute' }}</button>
      <div>
        <h3>Peers: {{ peers.size }}</h3>
        <div v-for="peer in Array.from(peers.values())" :key="peer.id">
          {{ peer.userId }}
        </div>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, onUnmounted } from 'vue';
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

const isConnected = ref(false);
const peers = ref(new Map());
const isMuted = ref(false);
const room = ref<ODIN.Room | null>(null);
const audioInput = ref<any>(null);
let audioPlugin: any;

// Initialize plugin
const initPlugin = async () => {
  if (audioPlugin) return audioPlugin;

  audioPlugin = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
    const audioContext = new AudioContext({ sampleRate });
    await audioContext.resume();
    return audioContext;
  });

  ODIN.init(audioPlugin);
  return audioPlugin;
};

const joinRoom = async () => {
  await initPlugin();

  // Fetch token from backend
  const response = await fetch('/api/odin-token');
  const { token } = await response.json();

  room.value = new ODIN.Room();

  room.value.onJoined = () => { isConnected.value = true; };
  room.value.onLeft = () => { isConnected.value = false; };
  room.value.onPeerJoined = (payload) => {
    peers.value.set(payload.peer.id, payload.peer);
  };
  room.value.onPeerLeft = (payload) => {
    peers.value.delete(payload.peer.id);
  };

  await room.value.join(token, { gateway: 'https://gateway.odin.4players.io' });
  await audioPlugin.setOutputDevice({});

  audioInput.value = await ODIN.DeviceManager.createAudioInput(null, {
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
  });
  await room.value.addAudioInput(audioInput.value);
};

const leaveRoom = async () => {
  if (audioInput.value) {
    await room.value?.removeAudioInput(audioInput.value);
    audioInput.value.close();
    audioInput.value = null;
  }

  if (room.value) {
    room.value.leave();
    room.value = null;
  }
};

const toggleMute = () => {
  if (audioInput.value) {
    const newVolume = isMuted.value ? 1 : 0;
    audioInput.value.setVolume(newVolume);
    isMuted.value = !isMuted.value;
  }
};

onUnmounted(() => {
  leaveRoom();
});
</script>
```

### Angular

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

@Component({
  selector: 'app-voice-chat',
  template: `
    <div>
      <button *ngIf="!isConnected" (click)="joinRoom()">Join Voice Chat</button>
      <ng-container *ngIf="isConnected">
        <button (click)="leaveRoom()">Leave</button>
        <button (click)="toggleMute()">{{ isMuted ? 'Unmute' : 'Mute' }}</button>
        <div>
          <h3>Peers: {{ peers.size }}</h3>
          <div *ngFor="let peer of getPeersArray()">
            {{ peer.userId }}
          </div>
        </div>
      </ng-container>
    </div>
  `
})
export class VoiceChatComponent implements OnInit, OnDestroy {
  isConnected = false;
  peers = new Map<number, any>();
  isMuted = false;

  private room: ODIN.Room | null = null;
  private audioInput: any = null;
  private audioPlugin: any;

  async ngOnInit() {
    await this.initPlugin();
  }

  ngOnDestroy() {
    this.leaveRoom();
  }

  private async initPlugin() {
    if (this.audioPlugin) return;

    this.audioPlugin = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
      const audioContext = new AudioContext({ sampleRate });
      await audioContext.resume();
      return audioContext;
    });

    ODIN.init(this.audioPlugin);
  }

  async joinRoom() {
    // Fetch token from backend
    const response = await fetch('/api/odin-token');
    const { token } = await response.json();

    this.room = new ODIN.Room();

    this.room.onJoined = () => { this.isConnected = true; };
    this.room.onLeft = () => { this.isConnected = false; };
    this.room.onPeerJoined = (payload) => {
      this.peers.set(payload.peer.id, payload.peer);
    };
    this.room.onPeerLeft = (payload) => {
      this.peers.delete(payload.peer.id);
    };

    await this.room.join(token, { gateway: 'https://gateway.odin.4players.io' });
    await this.audioPlugin.setOutputDevice({});

    this.audioInput = await ODIN.DeviceManager.createAudioInput(null, {
      echoCanceller: true,
      noiseSuppression: true,
      gainController: true,
    });
    await this.room.addAudioInput(this.audioInput);
  }

  async leaveRoom() {
    if (this.audioInput) {
      await this.room?.removeAudioInput(this.audioInput);
      this.audioInput.close();
      this.audioInput = null;
    }

    if (this.room) {
      this.room.leave();
      this.room = null;
    }
  }

  toggleMute() {
    if (this.audioInput) {
      const newVolume = this.isMuted ? 1 : 0;
      this.audioInput.setVolume(newVolume);
      this.isMuted = !this.isMuted;
    }
  }

  getPeersArray() {
    return Array.from(this.peers.values());
  }
}
```

## Advanced Patterns

### Device Selection

```javascript
// List available devices
const devices = await ODIN.DeviceManager.listAudioInputs();

// Show device picker
const deviceId = await showDevicePicker(devices);
const selectedDevice = devices.find(d => d.deviceId === deviceId);

// Create input with specific device
const audioInput = await ODIN.DeviceManager.createAudioInput(selectedDevice, {
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
});

// Switch device later
await audioInput.setDevice(newDevice);
```

### Connection Quality Monitoring

```javascript
setInterval(() => {
    const stats = room.connectionStats;

    if (stats.rtt > 200) {
        console.warn('High latency detected:', stats.rtt, 'ms');
    }

    if (stats.packetLoss > 5) {
        console.warn('High packet loss:', stats.packetLoss, '%');
    }
}, 5000);
```

### Push-to-Talk Implementation

```javascript
let audioInput;
let isPushToTalkActive = false;

// Create muted input
audioInput = await ODIN.DeviceManager.createAudioInput(null, {
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
});
audioInput.setVolume(0); // Start muted
await room.addAudioInput(audioInput);

// Key down: activate
document.addEventListener('keydown', (e) => {
    if (e.code === 'Space' && !isPushToTalkActive) {
        isPushToTalkActive = true;
        audioInput.setVolume(1); // Unmute
    }
});

// Key up: deactivate
document.addEventListener('keyup', (e) => {
    if (e.code === 'Space' && isPushToTalkActive) {
        isPushToTalkActive = false;
        audioInput.setVolume(0); // Mute
    }
});
```

### Room Reconnection

```javascript
let reconnectAttempts = 0;
const MAX_RECONNECT_ATTEMPTS = 5;

room.onLeft = async (payload) => {
    console.log('Disconnected from room');

    if (reconnectAttempts < MAX_RECONNECT_ATTEMPTS) {
        reconnectAttempts++;
        const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000);

        console.log(`Reconnecting in ${delay}ms (attempt ${reconnectAttempts})`);

        await new Promise(resolve => setTimeout(resolve, delay));

        try {
            await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
            reconnectAttempts = 0; // Reset on successful reconnection
        } catch (error) {
            console.error('Reconnection failed:', error);
        }
    }
};
```

## Troubleshooting & Best Practices

### Common Issues

**"AudioContext not allowed to start"**:
- **Cause**: Browser autoplay policy requires user interaction
- **Solution**: Call `initPlugin()` from a button click handler

**No audio received from peers**:
- Check `await audioPlugin.setOutputDevice({})` was called
- Verify browser has speaker permissions
- Check remotePeer.audioOutputs array is not empty

**Microphone not transmitting**:
- Verify `await room.addAudioInput(audioInput)` was called
- Check browser microphone permissions granted
- Ensure audioInput.volume is not 0

**Events not firing**:
- Register event handlers BEFORE calling `room.join()`
- Check for typos in event names (case-sensitive)

### Best Practices

1. **Event Registration**: Always register event handlers before joining room
2. **Resource Cleanup**: Call `audioInput.close()` and `room.leave()` on unmount
3. **Error Handling**: Wrap async operations in try-catch blocks
4. **Token Management**: Fetch fresh tokens before each join attempt
5. **Plugin Initialization**: Initialize plugin once per application lifecycle
6. **Device Selection**: Allow users to select input/output devices
7. **Connection Monitoring**: Track connection stats for quality insights
8. **User Feedback**: Show connection status and peer list to users
9. **Memory Management**: Remove event listeners when reusing Room instances
10. **Security**: Never commit access keys to version control

### Browser Compatibility

**Minimum Browser Versions**:
- Chrome/Edge: 94+
- Firefox: 93+
- Safari: 15.4+
- Opera: 80+

**Features**:
- WebTransport (HTTP/3): Chrome 97+, Edge 97+
- WebRTC fallback: All modern browsers
- Video support: All modern browsers

**Platform Notes**:
- **Firefox**: Must request MediaStream before device enumeration
- **Safari**: Requires explicit user interaction for AudioContext
- **Mobile**: iOS 15.4+, Android Chrome 94+

## Use Cases

- **Multiplayer Gaming**: Real-time voice chat for team coordination
- **Social Platforms**: Voice rooms and live audio conversations
- **Virtual Events**: Conferences, webinars, and networking
- **Education**: Virtual classrooms and study groups
- **Collaboration Tools**: Team communication and meetings
- **Live Streaming**: Interactive broadcaster-audience communication
- **Customer Support**: Voice-enabled support chat
- **Metaverse/VR**: Spatial voice for immersive experiences

## Documentation & Resources

- **API Reference**: https://docs.4players.io/voice/web/api/
- **Vanilla JS Guide**: https://docs.4players.io/voice/web/vanillajs
- **React Integration**: https://docs.4players.io/voice/web/react
- **Angular Integration**: https://docs.4players.io/voice/web/angular
- **NPM Package**: https://www.npmjs.com/package/@4players/odin
- **4Players ODIN Portal**: https://www.4players.io/odin/ (Get access keys)
- **GitHub Examples**: https://github.com/4Players/odin-sdk-web-examples
- **Support**: Discord, Email (support@4players.io), GitHub Issues

## Getting Started

1. Sign up at https://www.4players.io/odin/
2. Create a new app in the dashboard
3. Copy your access key
4. Set up backend token generation
5. Install SDK in your web application
6. Implement voice chat following the Quick Start guide

**Security Reminder**: Store access keys securely (environment variables, secret managers) - never commit to version control or expose in client-side code.
