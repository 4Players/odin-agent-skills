---
name: odin-voice-unreal
description: |
  ODIN Voice Unreal Engine SDK for real-time voice chat in UE4/UE5 games.
  Use when: implementing voice chat in Unreal projects, working with Blueprint nodes or C++ classes,
  handling room/peer/media events, configuring 3D spatial audio, or integrating audio capture.
  Requires UE 4.26+. For ODIN concepts, see odin-voice skill.
---

# ODIN Voice Unreal SDK

Unreal Engine plugin for real-time voice chat with full Blueprint and C++ support.

## Quick Start

### Installation
1. **Marketplace**: Install from Unreal Marketplace (recommended)
2. **Manual**: Clone from GitHub, use git-lfs, place in `/Plugins/Odin/`

### Basic Blueprint Flow
```
BeginPlay:
1. Construct Token Generator (access key)
2. Generate Room Token (room name, user name)
3. Construct Room Handle (with APM settings)
4. Bind event delegates (OnPeerJoined, OnMediaAdded)
5. Join Room
6. Create Audio Capture → Construct Local Media
7. Add Media To Room
```

## Key Blueprint Nodes

### Setup Nodes
| Node | Purpose |
|------|---------|
| `Construct a Token Generator` | Create token generator with access key |
| `Generate Room Token` | Generate auth token for room |
| `Construct Local Room Handle` | Create room instance |
| `Make ODIN APM Settings` | Configure audio processing |

### Room Management
| Node | Purpose |
|------|---------|
| `Join Room` | Connect to ODIN room |
| `Add Media To Room` | Add microphone stream |
| `Remove Media From Room` | Stop sending audio |
| `Update Peer Position` | Update 3D position |
| `Send Message` | Send data to peers |

### Audio Capture
| Node | Purpose |
|------|---------|
| `Create Audio Capture` | Create capture from mic |
| `Construct Local Media` | Create media from capture |
| `Get Capture Devices Available` | List microphones |
| `Get/Set Is Paused` | Mute/unmute control |

### Synth Component (Playback)
| Node | Purpose |
|------|---------|
| `Add Odin Synth Component` | Create audio output |
| `Odin Assign Synth to Media` | Connect to playback media |
| `Adjust Attenuation` | Set 3D audio settings |
| `Get/Set Volume Multiplier` | Volume control |

## Event Sequence

**Critical**: Bind event handlers BEFORE joining room.

```
1. Join Room Called
2. On Peer Joined (existing peers) ← Events for already-connected peers
3. On Room Joined ← Room connection successful
4. On Peer Joined (new) ← New peers joining after you
5. On Media Added ← CRITICAL: Create synth component here
6. On Media Removed ← Cleanup synth
7. On Peer Left ← Cleanup all components
```

### Blueprint Delegates
```cpp
Bind to On Room Joined         // Room connection successful
Bind to On Peer Joined         // Peer entered (peerId, userId, userData)
Bind to On Peer Left           // Peer exited
Bind to On Media Added         // Audio stream available (peerId, Media)
Bind to On Media Removed       // Audio stream stopped
Bind to On Message Received    // Data message received
Bind to On Connection State Changed  // Status updates
```

## C++ Integration

### Module Dependencies
```cpp
PublicDependencyModuleNames.AddRange(new string[] {
    "Core", "CoreUObject", "Engine", "InputCore",
    "Odin", "OdinLibrary",
    "AudioCapture", "AudioCaptureCore"
});
```

### Key Classes
```cpp
UOdinRoom*           // Room handle
UOdinTokenGenerator* // Token generation
UOdinPlaybackMedia*  // Incoming audio stream
UOdinAudioCapture*   // Microphone input
UOdinSynthComponent* // Audio output component
```

### C++ Setup Pattern
```cpp
// Header
UPROPERTY() UOdinRoom* Room;
UPROPERTY() UOdinAudioCapture* Capture;

UFUNCTION() void OnRoomJoinedHandler(...);
UFUNCTION() void OnMediaAddedHandler(int64 PeerId, UOdinPlaybackMedia* Media);

// BeginPlay - Bind BEFORE joining
Room->onPeerJoined.AddUniqueDynamic(this, &UMyComponent::OnPeerJoinedHandler);
Room->onMediaAdded.AddUniqueDynamic(this, &UMyComponent::OnMediaAddedHandler);

// Async join
UOdinRoomJoin* Action = UOdinRoomJoin::JoinRoom(
    this, Room, URL, Token, UserData, Position, OnError, OnSuccess);
Action->Activate();
```

### Media Playback (OnMediaAdded)
```cpp
void OnMediaAddedHandler(int64 PeerId, UOdinPlaybackMedia* Media) {
    // Find player character
    APlayerCharacter* Player = FindPlayerByPeerId(PeerId);

    // Create synth component
    UOdinSynthComponent* Synth = Cast<UOdinSynthComponent>(
        Player->AddComponentByClass(UOdinSynthComponent::StaticClass(),
            false, FTransform::Identity, false));

    // Assign media for playback
    Synth->Odin_AssignSynthToMedia(Media);

    // Setup 3D audio
    FSoundAttenuationSettings Attenuation;
    Attenuation.bSpatialize = true;
    Attenuation.bAttenuate = true;
    Synth->AdjustAttenuation(Attenuation);

    Synth->Activate();
}
```

## 3D Proximity Voice Pattern

### Player ID Synchronization
```
1. Each character has replicated PlayerId (GUID)
2. Store mapping: PlayerId → Character* in GameInstance
3. On peer join: Extract PlayerId from UserData
4. Create mapping: PeerId → Character*
5. On media added: Look up character, attach synth component
```

### Blueprint Implementation
```
On Peer Joined:
  → Get User Data from Peer
  → Parse PlayerId from bytes
  → Find Character by PlayerId
  → Store PeerId → Character mapping

On Media Added:
  → Look up Character from PeerId
  → Add Odin Synth Component to Character
  → Assign Synth to Media
  → Set Attenuation Settings (bSpatialize = true)
```

## APM Settings

Audio Processing Module configuration:
```cpp
FOdinApmSettings Settings;
Settings.bVoiceActivityDetection = true;
Settings.fVadAttackProbability = 0.9f;
Settings.fVadReleaseProbability = 0.8f;
Settings.bEnableVolumeGate = true;
Settings.bHighPassFilter = true;
Settings.bEchoCanceller = true;
Settings.noise_suppression_level = EOdinNoiseSuppressionLevel::Moderate;
```

## Common Patterns

### Push-to-Talk
```cpp
// Blueprint or C++
void OnPushToTalkPressed() {
    AudioCapture->SetIsPaused(false);
}

void OnPushToTalkReleased() {
    AudioCapture->SetIsPaused(true);
}
```

### Position Updates
```cpp
void Tick(float DeltaTime) {
    FVector Pos = GetActorLocation();
    Room->UpdatePeerPosition(Pos);
}
```

## Testing

Use ODIN Web Client (https://4players.app):
- Same access key as game
- Same gateway URL and room name
- Cross-platform testing without multiple game instances

## Sample Projects

- **Unreal Sample Project** - Basic integration
- **Unreal Minimal Samples** - Multiplayer proximity voice
- **Unreal Tech Demo** - Spatial audio, occlusion, effects
- **Unreal C++ Sample** - Full C++ patterns

## Documentation

Blueprint reference: https://docs.4players.io/voice/unreal/blueprint-reference/
Guides: https://docs.4players.io/voice/unreal/guides/
FAQ: https://docs.4players.io/voice/unreal/faq/
