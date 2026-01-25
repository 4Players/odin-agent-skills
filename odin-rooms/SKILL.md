---
name: odin-rooms
description: |
  ODIN Rooms - decentralized virtual meeting rooms platform built on ODIN Voice and Fleet.
  Use when: setting up ODIN Rooms instances, configuring room settings, understanding Rooms
  architecture, or extending Rooms with custom features. For voice SDK integration in custom
  apps, use the odin-voice-* skills instead.
---

# ODIN Rooms

Decentralized, always-open virtual meeting rooms platform by 4Players.

## Overview

ODIN Rooms provides browser-based virtual meeting spaces with:
- No accounts required for guests
- End-to-end encryption
- GDPR compliant (no metadata storage)
- Built on ODIN Voice + ODIN Fleet

## Key Features

| Feature | Description |
|---------|-------------|
| Audio/Video | Browser-based with permissions |
| Text Chat | Real-time messaging |
| Screen Sharing | Share your screen |
| Whiteboard | Collaborative drawing |
| Audio Processing | VAD, echo cancellation, noise suppression, gain control |
| Permissions | Mute participants, enable/disable features |

## Architecture

```
ODIN Rooms = ODIN Voice SDK + ODIN Fleet + Web UI
             (Real-time audio)  (Hosting)   (Interface)
```

## Setup Process

### 1. Sign Up
- Create account at ODIN Rooms portal
- Choose subdomain (e.g., `company.rooms.chat`)

### 2. Configure Instance
- **Title**: Display name for your rooms
- **Server Region**: Geographic location
- **Tier**: Capacity and features
- **Slots**: Maximum concurrent participants
- **Rental Duration**: Subscription period

### 3. Customize Branding
- Light mode logo
- Dark mode logo
- Primary colors
- Theme customization

### 4. Set Permissions
- Guest permissions (audio, video, chat)
- Owner password for admin access
- Room modification rights
- Participant management (mute, kick)

## Usage

### Joining Rooms
```
URL: https://{subdomain}.rooms.chat/{room-name}

Example: https://company.rooms.chat/standup
```

- Rooms are case-sensitive
- No `/` character in room names
- Auto-created on first join
- Auto-deleted when empty

### Sharing Links
Simply share the room URL. No invites or accounts needed for guests.

## Room Controls

### Audio Settings
- Input/output volume adjustment
- Voice Activity Detection (VAD)
- Automatic Gain Control
- Echo Cancellation
- Noise Suppression
- Volume Gate

### Video Settings
- Camera selection
- Quality settings
- Screen sharing

### Participant Management (Owner)
- Mute participants
- Enable/disable audio
- Enable/disable video
- Enable/disable chat
- Poke (attention request)
- Kick from room

## Extensibility

ODIN Rooms can be extended with:
- Custom bots
- Recording functionality
- Transcription and AI features
- Custom UIs via SDK integration

For building custom applications, use the ODIN Voice SDKs directly:
- Web apps: `odin-voice-web` skill
- Unity games: `odin-voice-unity` skill
- Unreal games: `odin-voice-unreal` skill

## Technical Details

### Free Tier
- Up to 20 participants
- Basic features

### Paid Tiers
- Higher participant limits
- Custom branding
- Priority support
- Extended features

### Infrastructure
- Powered by ODIN Fleet for global deployment
- Automatic scaling
- Low-latency audio/video

## Documentation

Getting started: https://docs.4players.io/rooms/getting-started/
Development: https://docs.4players.io/rooms/development/
Manual: https://docs.4players.io/rooms/manual/
