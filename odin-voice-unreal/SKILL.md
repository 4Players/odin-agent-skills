---
name: odin-voice-unreal
description: |
  ODIN Voice Unreal Engine SDK for real-time voice chat in UE4/UE5 games.
  Use when: implementing voice chat in Unreal projects, working with Blueprint nodes or C++ classes,
  handling room/peer/media events, configuring 3D spatial audio, or integrating audio capture.
  Requires UE 4.26+. For ODIN concepts, see odin-fundamentals skill.
license: MIT
---

# ODIN Voice Unreal SDK

Unreal Engine plugin for real-time voice chat with full Blueprint and C++ support.

## Quick Start

### Installation

**Marketplace (Recommended)**
- One-click installation through Unreal Marketplace
- Automatic updates
- Most reliable method for all project types

**Manual Installation**
1. Clone repository from GitHub
2. Run `git lfs fetch` to cache binary data locally
3. Run `git lfs checkout` to replace metadata with actual files
4. Place in `/MyProject/Plugins/Odin/`

**Blueprint-Only Projects**: May encounter packaging errors with manual installation. Solutions:
- Install plugin in Engine's Marketplace directory instead of project folder
- Convert to C++ project by adding a dummy C++ class

**Important**: When upgrading Unreal Engine versions, install ODIN plugin in new Engine version BEFORE opening existing projects

### System Requirements
- Unreal Engine 4.26 or later (including UE5.x)
- Audio Capture Plugin (Epic's official plugin, used by ODIN for microphone input)

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

### Echo Cancellation Best Practices
- Enable APM settings with Voice Activity Detection (VAD)
- Recommended VAD Attack/Release probability offset: 0.1
- Recommended Volume Gate Attack/Release offset: 10 dB
- Allow players to customize settings for their individual hardware setups
- Echo cancellation effectiveness varies by hardware configuration

### Volume Control Options
Multiple approaches for controlling audio levels:
1. **Gain Controller** in APM Settings
2. **Volume Multiplier** functions on Capture/Playback Media
3. **Unreal Sound Classes** applied to OdinSynthComponent objects

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
// IMPORTANT: Max 10 calls per second (100ms intervals)
// Use timer instead of Tick for reliable scheduling
void UpdatePosition() {
    FVector Pos = GetActorLocation();
    Room->UpdatePeerPosition(Pos);
}

// In BeginPlay or setup:
// GetWorld()->GetTimerManager().SetTimer(PositionTimer, this,
//     &AMyActor::UpdatePosition, 0.1f, true);
```

**Critical Requirements**:
- Maximum frequency: 10 calls per second (100ms minimum interval)
- Use "Set Timer by Event" node in Blueprints for reliable scheduling
- Ensure consistent position scale across all room peers

## Platform-Specific Considerations

### Android/Meta Quest
**JoinRoom Callback Issue**: Using standard `FVector2D()` constructor prevents callbacks from triggering.
```cpp
// WRONG - callbacks won't trigger
Room->JoinRoom(URL, Token, UserData, FVector2D());

// CORRECT - explicitly initialize
Room->JoinRoom(URL, Token, UserData, FVector2D(0, 0));
```

### Platform Permissions
- **Android**: Configure Android permissions for microphone access
- **Apple/iOS**: Set up Apple permissions in project settings

Refer to platform-specific permission guides in documentation.

## Audio Middleware Integration

ODIN supports integration with popular audio middleware solutions:

### FMOD
- FMOD Adapter Plugin available
- Enables ODIN voice chat through FMOD pipeline
- See FMOD Integration guide for setup

### Wwise
- Wwise Adapter Plugin available
- Integrates ODIN with Wwise audio engine
- See Wwise Integration guide for configuration

### Input Device Selection
- Guide available for custom microphone/input device configuration
- Allows players to select specific audio capture devices

## Testing

Use ODIN Web Client (https://4players.app):
- Same access key as game
- Same gateway URL and room name
- Cross-platform testing without multiple game instances
- Enables web-to-Unreal voice communication

## Troubleshooting

### Build and Packaging Issues

**Duplicate Plugin Installation**
- Never install ODIN in both Engine and Project directories simultaneously
- Causes UnrealBuildTool conflicts
- Solution: Keep plugin in one location only, clean temp folders:
  - Delete: Binaries, Build, Intermediate, DerivedDataCache
  - Rebuild project

**Asset Redirector Warnings**
- "LogUObjectGlobals: Warning" messages during packaging
- Solution: Right-click Content Browser root folder → "Fix up Redirectors"

**Blueprint-Only Project Crashes**
- Packaged builds may crash without C++ support
- Solutions:
  1. Install plugin in Engine Marketplace directory
  2. Convert to C++ project (add dummy C++ class)

**Engine Version Upgrades**
- CRITICAL: Install ODIN plugin in new engine BEFORE opening project
- Compiling blueprints without plugin causes irreversible damage
- Always prepare the engine first

### Spatial Audio Issues

**No 3D Positioning**
- Verify Odin Synth Component is attached to correct player character
- For non-Unreal clients: spawn placeholder actors "On Peer Joined"
- Attach synth components to these placeholders for proper positioning

**Sound Occlusion**
- Built-in support via Unreal's audio engine integration
- Works automatically when synth component is properly configured

### General Debugging Steps

For unlisted issues:
1. Delete intermediate folders: Binaries, Build, Intermediate, DerivedDataCache
2. Regenerate Visual Studio project files
3. Check ODIN version and Unreal Engine version compatibility
4. Post in `#odin-unreal` Discord with:
   - Plugin version
   - Engine version
   - Troubleshooting steps already attempted
   - Detailed error description

## Learning Resources

### Sample Projects
- **Unreal Sample Project** - Basic integration (GitHub)
- **Unreal Minimal Samples** - Multiplayer proximity voice
- **Unreal Tech Demo** - Spatial audio, occlusion, effects
- **Unreal C++ Sample** - Full C++ patterns

**Tip**: Select and copy blueprints from samples, paste directly into your project for rapid integration.

### Video Tutorials
- **YouTube Playlist**: Step-by-step video tutorial series demonstrating practical implementation
- Covers Blueprint-based setup and common patterns
- Visual guidance for complex integration scenarios

### Written Documentation
- **Blueprint Reference**: Complete node documentation
  - Audio Capture nodes
  - Delegates and Events
  - Functions and operations
  - Odin Synth Component
  - Room management
- **Implementation Guides**:
  - Platform permissions (Android, Apple)
  - Input device selection
  - Audio middleware (FMOD, Wwise)
  - Push-to-Talk implementation
  - C++ integration patterns
- **FAQ**: Common issues and solutions

## Documentation Links

Main docs: https://docs.4players.io/voice/unreal/
Blueprint reference: https://docs.4players.io/voice/unreal/blueprint-reference/
Guides: https://docs.4players.io/voice/unreal/guides/
FAQ: https://docs.4players.io/voice/unreal/faq/
GitHub repository: Source code and latest releases
Discord: `#odin-unreal` channel for support
