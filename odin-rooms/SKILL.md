---
name: odin-rooms
description: |
  ODIN Rooms - decentralized virtual meeting rooms platform built on ODIN Voice and Fleet.
  Use when: setting up ODIN Rooms instances, configuring room settings, understanding Rooms
  architecture, extending Rooms with custom features/bots, or integrating Rooms into custom apps.
  For voice SDK integration in custom apps, use the odin-voice-* skills instead.
---

# ODIN Rooms

Decentralized, always-open virtual meeting rooms platform by 4Players. Browser-based with no accounts required, end-to-end encryption, and full GDPR compliance.

## Overview

ODIN Rooms provides browser-based virtual meeting spaces with:
- **No accounts required** for guests - just share a link
- **End-to-end encryption** with OdinCrypto cipher
- **GDPR compliant** - no metadata storage, complete privacy
- **Built on ODIN Voice + ODIN Fleet** - low latency, crystal-clear audio
- **Self-hosting option** - full control over your data
- **Free for small businesses & NGOs** - up to 20 participants free

Create a Rooms instance: https://www.rooms.chat

## Core Concept

Unlike traditional meeting-based conferencing (Teams, Zoom), Rooms uses an **always-open room concept**:
- Rooms are always available, not scheduled for specific times
- Join whenever you want, leave when you're done
- Auto-created on first join, auto-deleted when empty
- Perfect for team spaces, virtual offices, spontaneous collaboration
- Example room names: `coffee-break`, `devs`, `marketing`, `standup`

## Key Features

| Feature | Description |
|---------|-------------|
| Audio/Video | Browser-based WebRTC with granular permissions |
| Text Chat | Real-time messaging with markdown, emojis, image upload (10MB max) |
| Screen Sharing | Share your screen with participants |
| Whiteboard | Collaborative drawing with shapes, colors, export as PNG/SVG |
| Audio Processing | VAD, echo cancellation, noise suppression, automatic gain control |
| Permissions | Owner controls: mute participants, enable/disable features |
| Custom Branding | Logos, backgrounds, colors for light/dark mode |

## Getting Started

### 1. Sign Up & Configure Instance
1. Visit https://www.rooms.chat/pricing/ to order instance (free tier available)
2. Choose a unique **subdomain** (e.g., `talk.rooms.chat`)
3. Configure **slots** (max concurrent participants - up to 20 free)
4. Select **rental duration** (1 month default, save 10% with 12 months)
5. Set **server region** for optimal performance
6. Configure **app title** (displayed in browser tab)

### 2. Customize Branding
- **Primary/Secondary Colors**: Customize UI theme
- **Logos**: Light and dark mode logos (HTTPS URLs required)
- **Backgrounds**: Custom background images (HTTPS URLs required)
- **Default Mode**: Set light or dark mode as default

### 3. Set Permissions
Configure guest permissions:
- **Modify App Settings**: Allow guests to change branding
- **Mute Participants**: Allow guests to mute others
- **Poke Participants**: Send attention notifications
- **Enable Audio/Video**: Control microphone/camera access
- **Clear Drawings/Chat**: Manage whiteboard and chat clearing

### 4. Owner Password
- Set an **owner password** to access admin settings
- Required for GDPR compliance (no user accounts stored)
- Contact support if password is lost

### 5. Join Your First Room
```
URL Format: https://{subdomain}.rooms.chat/{room-name}
Example: https://talk.rooms.chat/standup
```

- Room names are **case-sensitive**
- No `/` character allowed in room names
- Rooms auto-created on first join
- Simply share the URL with participants

## Using Rooms

### Audio/Video Settings
Configure before joining:
- **Audio Input/Output Device**: Select microphone and speakers/headphones
- **Input/Output Volume**: Adjust volume levels
- **Volume Gate**: Threshold for microphone activation
- **Automatic Gain Control**: Auto-adjust microphone volume
- **Voice Activity Detection (VAD)**: AI-powered silence detection
- **Echo Cancellation**: Prevent audio feedback
- **Noise Suppression**: Reduce background noise

### Room Controls (Dock)
- **Mute**: Toggle microphone (red cross = muted)
- **Output**: Mute speakers (can't hear others)
- **Video**: Toggle camera (red cross = disabled)
- **Screen**: Share your screen (green checkmark = sharing)
- **Notifications**: View chat messages, pokes, system alerts
- **Settings**: Open audio/video settings

### Participant Management
- **Volume**: Adjust individual participant volume
- **Poke**: Send attention notification to participant
- **Add to Favorites**: Pin participant to top of list
- **Mute Microphone**: Mute participant (requires permissions)

### Text Chat
- Real-time messaging (not stored on server)
- **Markdown support**: Bold `**text**`, italic `*text*`, strikethrough `~~text~~`, code `` `code` ``
- **Emojis**: Full emoji support
- **Image upload**: Max 10MB per image
- **Multiline**: Press `Shift + Enter` for new line
- **Not persistent**: Chat history cleared on page reload (GDPR compliance)

### Whiteboard
- **Drawing Tools**: Freehand, lines, rectangles, circles
- **Color Picker**: Multiple color options
- **Eraser**: Erase parts or clear entire whiteboard
- **Undo/Redo**: Revert changes
- **Download**: Export as PNG or SVG
- Collaborative in real-time

## Development & Integration

### Prerequisites for Development
- A Rooms instance: https://www.rooms.chat/pricing/ (free option available)
- **Project API Key** (`apiKey`): Get from https://app.netplay-config.4players.de/rooms
- **Project Secret** (`secret`): Get from settings page (keep private, server-side only)

### 1. Create ODIN Token (Server-Side)

Rooms uses ODIN access tokens. Create tokens via hosted service:

**Endpoint**: `POST https://secure.4players.de/public/api/v1/projects/${apiKey}/create-token`

**Request Payload**:
```json
{
  "roomId": "my-room",
  "userId": "my-user-id",
  "secret": "my-secret"
}
```

**Response**:
```json
{
  "data": {
    "token": "eyJhbGciOiJFZERTQSIsIm...",
    "domain": "example.rooms.chat"
  },
  "error": {
    "code": 0,
    "message": ""
  }
}
```

**Node.js Example**:
```javascript
import express from "express";

const app = express();
app.use(express.json());

app.post("/api/rooms/create-token", async (req, res) => {
  const {roomId, userId, secret} = req.body;

  // Keep this on server only (env var / secret manager)
  const apiKey = process.env.ROOMS_PROJECT_API_KEY;

  const r = await fetch(
    `https://secure.4players.de/public/api/v1/projects/${apiKey}/create-token`,
    {
      method: "POST",
      headers: {"content-type": "application/json"},
      body: JSON.stringify({roomId, userId, secret}),
    }
  );

  if (!r.ok) {
    return res.status(r.status).json({error: "Token request failed"});
  }

  const json = await r.json();
  res.json(json.data);
});

app.listen(3000);
```

**WARNING**: Never call token endpoint from browser in production. API keys must stay server-side. Rate limit: 100 requests/minute.

### 2. Join Room with ODIN Web SDK (Client-Side)

**Install Dependencies**:
```bash
npm install @4players/odin @4players/odin-plugin-web
```

**Minimal Example**:
```typescript
import {init, Room, DeviceManager} from "@4players/odin";
import {createPlugin} from "@4players/odin-plugin-web";

let audioPlugin;

async function initOdinPlugin() {
  if (audioPlugin) return audioPlugin;

  audioPlugin = createPlugin(async (sampleRate) => {
    const audioContext = new AudioContext({sampleRate});
    await audioContext.resume();
    return audioContext;
  });

  init(audioPlugin);

  // Required to hear other peers (audio output is plugin-global)
  await audioPlugin.setOutputDevice({});

  return audioPlugin;
}

async function joinRooms({roomId, displayName}) {
  await initOdinPlugin();

  // 1) Create token (via your backend)
  const userId =
    displayName.replace(/[^a-zA-Z0-9]/g, "_").toLowerCase() +
    "_" +
    Math.floor(Math.random() * 10000);

  const tokenResp = await fetch("/api/rooms/create-token", {
    method: "POST",
    headers: {"content-type": "application/json"},
    body: JSON.stringify({roomId, userId}),
  });

  const {token} = await tokenResp.json();

  // Use Europe gateway (see https://docs.4players.io/voice/server/cloud/#gateway)
  const gateway = "https://gateway.odin.4players.io";

  // 2) Prepare Rooms peer data (for UI display)
  const roomsPeerData = {
    userId,
    name: displayName,
    avatar: `https://app-server.odin.4players.io/v1/avatar/${userId}/initials`,
    inputMuted: false,
    outputMuted: false,
    role: "Guest",
  };

  const userData = new TextEncoder().encode(JSON.stringify(roomsPeerData));

  // 3) Join ODIN room
  const room = new Room();

  await room.join(token, {
    gateway,
    // Rooms is E2E encrypted by default
    // Default password is the roomId
    cipher: {type: "OdinCrypto", password: roomId},
    userData,
  });

  // 4) Start microphone
  const audioInput = await DeviceManager.createAudioInput();
  await room.addAudioInput(audioInput);

  return {room, audioInput};
}
```

### Rooms Peer Data Structure

Rooms requires specific peer data format to display participants correctly:

```typescript
{
  userId: string,           // Unique identifier
  name: string,            // Display name (default: 'Unknown')
  status: string,          // 'online', etc. (default: 'online')
  avatar: string,          // Image URL (default: generated)
  outputMuted: boolean,    // Speaker muted (default: false)
  inputMuted: boolean,     // Microphone muted (default: false)
  renderer?: string,       // Optional: renderer info
  platform?: string,       // Optional: platform info
  revision?: string,       // Optional: revision info
  version?: string,        // Optional: version info
  buildNo?: number,        // Optional: build number
  role?: string,           // Optional: role (default: 'Guest')
  emoji?: string,          // Optional: emoji status
  customStatus?: string,   // Optional: custom status text
  presentationStartedAt: number  // Timestamp (default: 0)
}
```

### Avatar Service

Generate avatars for testing:
```
https://app-server.odin.4players.io/v1/avatar/{seed}/initials
```
The `seed` is initials followed by `-` and a number encoding a color.

### Important: Matching Parameters

For participants to connect to the same room, these must match:
1. **Rooms project API key**
2. **Room ID** (case-sensitive)
3. **Gateway** (same gateway for all participants)

### Use Cases for Development
- **Bots**: Recording, transcription, AI features
- **Custom UIs**: Build your own interface
- **Integrations**: External automations, webhooks
- **Communicator Widget**: Drop-in voice chat widget (see `odin-voice-web` skill)

## Technical Details

### Pricing Tiers

**Free Tier**:
- Up to 20 participants
- Unlimited rooms
- All core features
- Perfect for small businesses, NGOs, personal use

**Paid Tiers**:
- Higher participant limits (custom slots)
- Custom branding
- Priority support
- Extended features
- Save 10% with 12-month plan

### Architecture

```
ODIN Rooms = ODIN Voice SDK + ODIN Fleet + Web UI
             (Real-time audio)  (Hosting)     (Interface)
```

### Infrastructure
- **Powered by ODIN Fleet**: Global deployment, automatic scaling
- **Low-latency WebRTC**: Crystal-clear audio/video
- **Decentralized**: Secure and private communications
- **No metadata storage**: Full GDPR compliance

### Browser Permissions
- First time joining or opening settings: browser requests microphone/camera access
- Must allow permissions to use audio/video features
- Change in browser settings if accidentally denied

## Documentation & Support

- **Getting Started**: https://docs.4players.io/rooms/getting-started/
- **User Manual**: https://docs.4players.io/rooms/manual/
- **Development Guide**: https://docs.4players.io/rooms/development/
- **ODIN Voice Web SDK**: https://docs.4players.io/voice/web/
- **Discord Support**: https://4np.de/discord
- **Email Support**: support@4players.io

For custom integrations, bots, or advanced features, refer to the ODIN Voice SDK documentation and related skills:
- Web apps: `odin-voice-web` skill
- Unity games: `odin-voice-unity` skill
- Unreal games: `odin-voice-unreal` skill
