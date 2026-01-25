---
name: odin-voice-unity
description: |
  ODIN Voice Unity SDK for real-time voice chat in Unity games and XR experiences.
  Supports both 1.x (stable, Unity 2019.4+) and 2.x (beta, Unity 2021.4+) versions.
  Use when: implementing voice chat in Unity projects, handling peer/media events,
  configuring 2D/3D spatial audio, or integrating microphone input.
  CRITICAL: Always determine installed version first - APIs differ significantly.
  For ODIN concepts, see odin-voice skill.
---

# ODIN Voice Unity SDK

Unity SDK for real-time 3D positional voice chat integration.

## ⚠️ VERSION DETECTION - READ THIS FIRST

**CRITICAL**: ODIN Unity has two major versions with completely different APIs.
**You MUST determine which version is installed before providing guidance.**

### How to Detect Version

1. **Check Package Manager**:
   - Open Unity Package Manager (Window > Package Manager)
   - Find "4Players ODIN" package
   - Version number indicates: 1.x.x or 2.x.x

2. **Check for Key Components**:
   - **Version 1.x**: Has `OdinHandler` class (singleton pattern)
   - **Version 2.x**: Has `OdinRoom` class and `OdinInstance` prefab

3. **Check Unity Version**:
   - Unity 2019.4 - 2020.x → Most likely 1.x
   - Unity 2021.4+ → Could be either, but 2.x requires this minimum

4. **Check Installation Method**:
   - Installed via Git URL → Version 1.x
   - Installed via "Add from disk" with ZIP → Likely version 2.x

### Which Version to Use

| Version | Status | Unity Support | Best For |
|---------|--------|---------------|----------|
| **1.x** | Stable (current release) | 2019.4+ | Production projects, older Unity versions |
| **2.x** | Pre-release beta | 2021.4+ | New projects, testing, requires explicit access |

**Default recommendation**: Use version 1.x unless the user specifically requested 2.x or is in the beta program.

---

# Version 1.x Documentation

**Status**: Current stable release
**Unity**: 2019.4 or later
**Installation**: Package Manager with Git URL or Unity Package from GitHub

## Installation (1.x)

### Method 1: Package Manager (Recommended)
```
Window > Package Manager > + > Add package from git URL
URL: https://github.com/4Players/odin-sdk-unity.git
```

### Method 2: Unity Package
1. Download `.unitypackage` from GitHub releases
2. Double-click to import into project

**Important**: Remove previous versions before upgrading.

## Core Components (1.x)

### OdinHandler (Singleton)
Main management component - handles all ODIN functionality:
```csharp
// Access the singleton instance
OdinHandler.Instance.JoinRoom("MyRoomName");
```

**Properties**:
- Singleton pattern with `DontDestroyOnLoad`
- Manages lifetime across scene changes
- Handles token generation, events, and audio pipeline

**Placement**: Add to starting scenes like "Loading" or "Main Menu"

### OdinEditorConfig
Configuration component for:
- Sample rate settings
- Microphone selection preferences
- Basic ODIN setup parameters

### MicrophoneReader
Captures and sends audio input:
```csharp
// Control microphone transmission
AudioSender.RedirectCapturedAudio = Input.GetKey(KeyCode.V); // Push-to-talk
```

### PlaybackComponent
Manages audio playback for remote peers:
- Wraps peer audio streams as Unity AudioSources
- Enables spatial audio configuration

## Quick Start (1.x)

### Minimal Setup
```csharp
using OdinNative.Unity;

void Start() {
    // Join a room
    OdinHandler.Instance.JoinRoom("MyRoomName");

    // Listen to events
    OdinHandler.Instance.OnCreatedMediaObject.AddListener(OnMediaCreated);
    OdinHandler.Instance.OnDeleteMediaObject.AddListener(OnMediaDeleted);
}

void OnMediaCreated(string roomName, ulong peerId, int mediaId) {
    // Create playback component for peer audio
    PlaybackComponent playback = OdinHandler.Instance.AddPlaybackComponent(
        peerContainer, roomName, peerId, mediaId);

    // Configure for 3D spatial audio
    playback.PlaybackSource.spatialBlend = 1.0f;
    playback.PlaybackSource.maxDistance = 10f;
}
```

### Access Key Configuration
Set access key in Unity Inspector under "Client Authorization" in ODIN Manager component.
Free tier supports up to 25 concurrent users.

## Key Events (1.x)

```csharp
// Peer starts sending audio
OdinHandler.Instance.OnCreatedMediaObject.AddListener(
    (string roomName, ulong peerId, int mediaId) => {
        // Create playback for this peer
    });

// Peer stops sending audio
OdinHandler.Instance.OnDeleteMediaObject.AddListener(
    (string roomName, ulong peerId, int mediaId) => {
        // Cleanup playback
    });

// Peer leaves room
OdinHandler.Instance.OnRoomLeft.AddListener(
    (string roomName, ulong peerId) => {
        // Handle peer departure
    });
```

## Spatial Audio (1.x)

### 2D Voice Chat (Non-Spatial)
```csharp
void OnMediaCreated(string roomName, ulong peerId, int mediaId) {
    PlaybackComponent playback = OdinHandler.Instance.AddPlaybackComponent(
        peerContainer, roomName, peerId, mediaId);

    // 2D audio - same volume for everyone
    playback.PlaybackSource.spatialBlend = 0.0f;
}
```

### 3D Voice Chat (Spatial)
```csharp
void OnMediaCreated(string roomName, ulong peerId, int mediaId) {
    // Find player GameObject for this peer
    GameObject playerObject = FindPlayerByPeerId(peerId);

    // Add playback component to player
    PlaybackComponent playback = OdinHandler.Instance.AddPlaybackComponent(
        playerObject, roomName, peerId, mediaId);

    // Configure 3D spatial audio
    playback.PlaybackSource.spatialBlend = 1.0f;
    playback.PlaybackSource.rolloffMode = AudioRolloffMode.Inverse;
    playback.PlaybackSource.maxDistance = 10f;
    playback.PlaybackSource.minDistance = 1f;
}
```

## Common Patterns (1.x)

### Push-to-Talk (1.x)
```csharp
void Update() {
    // V key for push-to-talk
    AudioSender.RedirectCapturedAudio = Input.GetKey(KeyCode.V);
}
```

### AudioListener Positioning
```csharp
// Position AudioListener on player for first-person perspective
AudioListener listener = Camera.main.GetComponent<AudioListener>();
```

## Available Samples (1.x)
Import from Package Manager (4Players ODIN > Samples):
- **Sample3dScene** - 3D positional audio example
- **Integration samples** for Photon PUN, Mirror Networking

## Documentation Links (1.x)
- Main docs: https://docs.4players.io/voice/unity/
- Guides: https://docs.4players.io/voice/unity/guides/
- API Reference: https://docs.4players.io/voice/unity/api/

---
---

# Version 2.x Documentation

**Status**: Pre-release beta (requires explicit access)
**Unity**: 2021.4 or later
**Installation**: Package Manager via "Add from disk" with provided ZIP file

## Installation (2.x)

1. Receive ZIP file from 4Players (beta access required)
2. Extract ZIP to local folder
3. Open Unity Package Manager (Window > Package Manager)
4. Click + button → "Add package from disk"
5. Select `package.json` from extracted folder
6. Add `OdinInstance` prefab from `Packages/io.fourplayers.odin/Runtime/` to scene

## Core Components (2.x)

**Major Architecture Change**: Version 2.x eliminates the singleton pattern and allows multiple concurrent room instances.

### OdinInstance Prefab
Pre-configured prefab containing essential components. Drag into your scene to get started.

### OdinRoom Component
Modern replacement for OdinHandler - **not a singleton**:
```csharp
// You can create multiple rooms
OdinRoom teamChat;
OdinRoom proximityVoice;
```

**Key difference from 1.x**: Flexible, non-singleton design enables multi-room scenarios.

### OdinPeer Component
Represents a connected peer (auto-created by OdinRoom).

### OdinMedia Component
Handles audio playback for peer streams (auto-created by OdinRoom).

### OdinMicrophoneReader Component
Captures and sends microphone audio.

## Quick Start (2.x)

### API Decision Tree

```
Need voice chat?
├─ Simple 2D (non-spatial) → Use OdinRoom component (High-Level)
├─ Simple 3D (spatial) → Use OdinRoom + custom event handling
└─ Complex multi-room/shared connections → Use Room class (Low-Level)
```

### Minimal Setup (2.x)
```csharp
OdinRoom _room;

void Start() {
    _room = GetComponent<OdinRoom>();
    _room.Gateway = "https://gateway.odin.4players.io";
    _room.OnRoomJoined.AddListener(OnJoined);
    _room.OnPeerJoined.AddListener(OnPeerJoined);
    _room.OnMediaAdded.AddListener(OnMediaAdded);

    // For testing only - generate tokens on backend in production
    _room.Token = OdinRoom.GenerateTestToken("MyRoom", "User1", 5, "YOUR_ACCESS_KEY");
}
```

## High-Level API - OdinRoom (2.x)

### Component Reference

| Component | Purpose |
|-----------|---------|
| `OdinRoom` | Main MonoBehaviour, handles server communication |
| `OdinPeer` | Represents connected peer (auto-created) |
| `OdinMedia` | Audio playback for peer's stream (auto-created) |
| `OdinMicrophoneReader` | Captures and sends microphone audio |

### OdinRoom Properties (2.x)
```csharp
string Gateway;           // Server URL
string Token;             // Auth token (auto-joins when set)
bool IsJoined;            // Connection status
AudioMixerGroup AudioMixerGroup;  // For audio effects
```

### OdinRoom Events (2.x)
```csharp
// Room lifecycle
OnRoomJoined      // Successfully joined - args: ownPeerId, roomName, peers
OnConnectionStateChanged  // Status changes

// Peer management
OnPeerJoined      // New peer - args: peerId, userId, userData
OnPeerLeft        // Peer left - args: peerId

// Media streams
OnMediaAdded      // Peer started audio - args: peerId, mediaId
OnMediaRemoved    // Peer stopped audio - args: peerId, mediaId

// Messaging
OnMessageReceived // Data received - args: peerId, bytes
```

### 2D Voice Chat - Non-Spatial (2.x)
```csharp
void OnPeerJoined(PeerJoinedEventArgs args) {
    Debug.Log($"Peer {args.UserId} joined");
    // OdinPeer GameObject created automatically
    // Audio plays at same volume for everyone
}

void OnMediaAdded(MediaAddedEventArgs args) {
    // OdinMedia component created automatically with spatialBlend = 0
}
```

### 3D Voice Chat - Spatial (2.x)
```csharp
void OnPeerJoined(PeerJoinedEventArgs args) {
    // Parse user data to find matching player
    int connectionId = BitConverter.ToInt32(args.UserData, 0);
    PlayerController player = FindPlayerByConnectionId(connectionId);

    // Parent OdinPeer to player transform for 3D positioning
    OdinPeer peer = // get from OdinRoom
    peer.transform.SetParent(player.transform);
}

void OnMediaAdded(MediaAddedEventArgs args) {
    OdinMedia media = // get media component
    media.SpatialBlend = 1.0f;  // Enable 3D
    media.RolloffMode = AudioRolloffMode.Inverse;
    media.MinDistance = 1f;
    media.MaxDistance = 100f;
}

void Update() {
    // Update position for server-side culling
    _room.SetPosition(transform.position.x, transform.position.y, transform.position.z);
}
```

## Low-Level API - Room Class (2.x)

For full control over voice implementation.

### Creating Room (2.x)
```csharp
Room _room;
ConcurrentQueue<Action> UnityQueue = new ConcurrentQueue<Action>();

void Start() {
    _room = Room.Create("https://gateway.odin.4players.io", 48000, false);
    _room.OnRoomJoined += OnRoomJoined;
    _room.OnMediaStarted += OnMediaStarted;
    _room.OnMediaStopped += OnMediaStopped;
}
```

### Handling Events - Thread Safety (2.x)
```csharp
// Events called in different thread - must dispatch to Unity main thread
void OnRoomJoined(object sender, ulong ownPeerId, string name, ...) {
    if (_room.AvailableEncoderIds.TryDequeue(out ushort mediaId)) {
        if (_room.GetOrCreateEncoder(mediaId, out MediaEncoder encoder)) {
            _room.StartMedia(encoder);
            // Link to microphone
        }
    }
}

void OnMediaStarted(object sender, ulong peerId, MediaRpc media) {
    if (_room.RemotePeers.TryGetValue(peerId, out PeerEntity peer)) {
        if (peer.Medias.TryGetValue(media.Id, out MediaDecoder decoder)) {
            UnityQueue.Enqueue(() => CreatePlayback(decoder, peer));
        }
    }
}

void Update() {
    while (UnityQueue.TryDequeue(out var action)) {
        action?.Invoke();
    }
}
```

### Room Key Methods (2.x)
```csharp
Room.Create(endpoint, samplerate, stereo)  // Create instance
room.Join(token)                           // Connect to room
room.StartMedia(encoder)                   // Begin sending audio
room.StopMedia(encoder)                    // Stop sending
room.SetPosition(x, y, z)                  // Update 3D position
room.SendMessage(string/bytes)             // Send data to peers
room.UpdateUserData(bytes)                 // Update peer metadata
```

## Audio Pipeline (2.x)

### Adding Effects to Media (2.x)
```csharp
OdinMedia media = // get media
var pipeline = media.GetPipeline();

media.AddApm();          // Acoustic Processing Module
media.AddVad();          // Voice Activity Detection
media.AddVolumeBoost();  // Volume enhancement
media.AddMute();         // Mute control
```

## Microphone Input (2.x)

### OdinMicrophoneReader Properties (2.x)
```csharp
bool RedirectCapturedAudio = true;   // Auto-send to servers
bool SilenceCapturedAudio = false;   // Mute outgoing
bool AutostartListen = true;         // Start on component Start()
string InputDevice;                  // Specific device name
float MicVolumeScale;                // Volume multiplier
```

## Multi-Room Setup (2.x)
```csharp
// Create separate OdinRoom instances
OdinRoom teamChat;   // 2D team communication
OdinRoom worldVoice; // 3D proximity voice

void Start() {
    teamChat.OnPeerJoined += Handle2DPeer;
    worldVoice.OnPeerJoined += Handle3DPeer;

    teamChat.Token = GetTeamToken();
    worldVoice.Token = GetWorldToken();
}
```

## Common Patterns (2.x)

### User Data for Player Mapping (2.x)
```csharp
// When joining - include player ID in user data
byte[] userData = BitConverter.GetBytes(myPlayerId);
_room.UpdateUserData(userData);

// When peer joins - extract their player ID
void OnPeerJoined(args) {
    int theirPlayerId = BitConverter.ToInt32(args.UserData, 0);
    // Map to correct player GameObject
}
```

### Push-to-Talk (2.x)
```csharp
void Update() {
    if (Input.GetKeyDown(KeyCode.V))
        microphoneReader.SilenceCapturedAudio = false;
    if (Input.GetKeyUp(KeyCode.V))
        microphoneReader.SilenceCapturedAudio = true;
}
```

## Available Samples (2.x)

Import from Package Manager:
- **InstanceSample2D** - Conferencing-style non-spatial voice
- **InstanceSample3D** - Spatial voice with positioning
- **InstanceSampleBasic** - Minimal low-level API example

## Documentation Links (2.x)

- Main docs: https://docs.4players.io/voice/unity/next/
- Manual: https://docs.4players.io/voice/unity/next/manual/
- API Reference: https://docs.4players.io/voice/unity/next/api/

---

## Quick Reference: Version Differences

| Feature | Version 1.x | Version 2.x |
|---------|-------------|-------------|
| **Main Component** | `OdinHandler` (singleton) | `OdinRoom` (non-singleton) |
| **Architecture** | Singleton pattern | Component-based, multi-room |
| **Room Join** | `OdinHandler.Instance.JoinRoom("room")` | `odinRoom.Token = token;` |
| **Playback** | Manual `AddPlaybackComponent()` | Auto-created `OdinPeer`/`OdinMedia` |
| **Events** | `OnCreatedMediaObject`, `OnDeleteMediaObject` | `OnPeerJoined`, `OnMediaAdded` |
| **Unity Version** | 2019.4+ | 2021.4+ |
| **Status** | Stable release | Pre-release beta |
| **Installation** | Git URL or Unity Package | ZIP file with "Add from disk" |
| **Prefab** | Manual setup or samples | `OdinInstance` prefab |

## Migration from 1.x to 2.x

**Key Changes**:
1. Replace `OdinHandler.Instance` calls with `OdinRoom` component references
2. Update event listeners: `OnCreatedMediaObject` → `OnMediaAdded`
3. Remove manual `AddPlaybackComponent()` calls - handled automatically
4. Set `Token` property instead of calling `JoinRoom()`
5. Update Unity version to 2021.4+ if needed

**When to migrate**: Only migrate if you need multi-room support or new 2.x features. Version 1.x remains stable and supported.
