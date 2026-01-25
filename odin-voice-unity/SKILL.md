---
name: odin-voice-unity
description: |
  ODIN Voice Unity SDK 2.x for real-time voice chat in Unity games and XR experiences.
  Use when: implementing voice chat in Unity projects, working with OdinRoom or Room classes,
  handling peer/media events, configuring 2D/3D spatial audio, or integrating microphone input.
  Requires Unity 2021.4+. For ODIN concepts, see odin-voice skill.
---

# ODIN Voice Unity SDK 2.x

Unity SDK for real-time 3D positional voice chat integration.

## Quick Start

### Installation
1. Import package via Package Manager → Add package from disk
2. Select `package.json` from extracted ZIP
3. Add `OdinInstance` prefab from `Packages/io.fourplayers.odin/Runtime/` to scene

### Minimal Setup
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

## API Decision Tree

```
Need voice chat?
├─ Simple 2D (non-spatial) → Use OdinRoom component (High-Level)
├─ Simple 3D (spatial) → Use OdinRoom + custom event handling
└─ Complex multi-room/shared connections → Use Room class (Low-Level)
```

## High-Level API (OdinRoom)

### Key Components

| Component | Purpose |
|-----------|---------|
| `OdinRoom` | Main MonoBehaviour, handles server communication |
| `OdinPeer` | Represents connected peer (auto-created) |
| `OdinMedia` | Audio playback for peer's stream (auto-created) |
| `OdinMicrophoneReader` | Captures and sends microphone audio |

### OdinRoom Properties
```csharp
string Gateway;           // Server URL
string Token;             // Auth token (auto-joins when set)
bool IsJoined;            // Connection status
AudioMixerGroup AudioMixerGroup;  // For audio effects
```

### OdinRoom Events
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

### 2D Voice Chat (Non-Spatial)
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

### 3D Voice Chat (Spatial)
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

## Low-Level API (Room)

For full control over voice implementation.

### Creating Room
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

### Handling Events (Thread Safety)
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

### Room Key Methods
```csharp
Room.Create(endpoint, samplerate, stereo)  // Create instance
room.Join(token)                           // Connect to room
room.StartMedia(encoder)                   // Begin sending audio
room.StopMedia(encoder)                    // Stop sending
room.SetPosition(x, y, z)                  // Update 3D position
room.SendMessage(string/bytes)             // Send data to peers
room.UpdateUserData(bytes)                 // Update peer metadata
```

## Audio Pipeline

### Adding Effects to Media
```csharp
OdinMedia media = // get media
var pipeline = media.GetPipeline();

media.AddApm();          // Acoustic Processing Module
media.AddVad();          // Voice Activity Detection
media.AddVolumeBoost();  // Volume enhancement
media.AddMute();         // Mute control
```

## Microphone Input

### OdinMicrophoneReader Properties
```csharp
bool RedirectCapturedAudio = true;   // Auto-send to servers
bool SilenceCapturedAudio = false;   // Mute outgoing
bool AutostartListen = true;         // Start on component Start()
string InputDevice;                  // Specific device name
float MicVolumeScale;                // Volume multiplier
```

## Multi-Room Setup
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

## Common Patterns

### User Data for Player Mapping
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

### Push-to-Talk
```csharp
void Update() {
    if (Input.GetKeyDown(KeyCode.V))
        microphoneReader.SilenceCapturedAudio = false;
    if (Input.GetKeyUp(KeyCode.V))
        microphoneReader.SilenceCapturedAudio = true;
}
```

## Samples

Import from Package Manager:
- **InstanceSample2D** - Conferencing-style non-spatial voice
- **InstanceSample3D** - Spatial voice with positioning
- **InstanceSampleBasic** - Minimal low-level API example

## Documentation

Full API reference: https://docs.4players.io/voice/unity/api/
Manual: https://docs.4players.io/voice/unity/manual/
