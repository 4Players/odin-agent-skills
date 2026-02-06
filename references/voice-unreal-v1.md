# ODIN Voice Unreal SDK 1.x

Unreal Engine plugin for real-time voice chat with full Blueprint and C++ support.

> **Version Compatibility**: ODIN 1.x and 2.x are **not interoperable**. Clients using SDK 1.x cannot communicate with clients using SDK 2.x.

## Overview

The v1.x SDK uses:
- **Media objects** (`UOdinCaptureMedia`, `UOdinPlaybackMedia`) for audio streams
- **Room-level APM settings** configured during construction
- **`OnMediaAdded`/`OnMediaRemoved` events** for audio stream lifecycle
- **Latent `JoinRoom` action** with error/success callbacks

## Quick Start

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

### Security Note

> **CRITICAL**: Only embed access keys in client code for testing purposes, __NEVER__ in production code. Generate room tokens on a server for production deployments.

## Key Blueprint Nodes

### Setup Nodes

| Node | Purpose |
|------|---------|
| `Construct a Token Generator` | Create token generator with access key |
| `Generate Room Token` | Generate auth token for room |
| `Construct Local Room Handle` | Create room instance with APM settings |
| `Make ODIN APM Settings` | Configure audio processing |

### Room Management

| Node | Purpose |
|------|---------|
| `Join Room` | Connect to ODIN room (latent action) |
| `Add Media To Room` | Add microphone stream |
| `Remove Media From Room` | Stop sending audio |
| `Update Peer Position` | Update 3D position |
| `Send Message` | Send data to peers |

### Audio Capture

| Node | Purpose |
|------|---------|
| `Create Audio Capture` | Create capture object from microphone |
| `Construct Local Media` | Create media from capture object |
| `Get Capture Devices Available` | List available microphones |
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
2. On Peer Joined (existing peers)  ← Events for already-connected peers
3. On Room Joined                   ← Room connection successful
4. On Peer Joined (new)             ← New peers joining after you
5. On Media Added                   ← CRITICAL: Create synth component here
6. On Media Removed                 ← Cleanup synth
7. On Peer Left                     ← Cleanup all components
```

### Blueprint Delegates

```cpp
Bind to On Room Joined              // Room connection successful
Bind to On Peer Joined              // Peer entered (peerId, userId, userData)
Bind to On Peer Left                // Peer exited
Bind to On Media Added              // Audio stream available (peerId, Media)
Bind to On Media Removed            // Audio stream stopped
Bind to On Message Received         // Data message received
Bind to On Connection State Changed // Status updates
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
UOdinRoom*              // Room handle
UOdinTokenGenerator*    // Token generation
UOdinPlaybackMedia*     // Incoming audio stream
UOdinCaptureMedia*      // Outgoing audio stream
UOdinAudioCapture*      // Microphone input
UOdinSynthComponent*    // Audio output component
```

### Room Construction with APM

```cpp
// Configure APM settings at room construction
FOdinApmSettings Settings;
Settings.bVoiceActivityDetection = true;
Settings.fVadAttackProbability = 0.7f;
Settings.fVadReleaseProbability = 0.6f;
Settings.bEnableVolumeGate = true;
Settings.fVolumeGateAttackLoudness = -30.0f;
Settings.fVolumeGateReleaseLoudness = -40.0f;
Settings.bHighPassFilter = true;
Settings.bEchoCanceller = true;
Settings.noise_suppression_level = EOdinNoiseSuppressionLevel::OdinNS_Moderate;

UOdinRoom* OdinRoomPtr = UOdinRoom::ConstructRoom(this, Settings);
```

### Token Generation

```cpp
UOdinTokenGenerator* TokenGenerator = UOdinTokenGenerator::ConstructTokenGenerator(
    this, "<ACCESS_KEY>");
FString RoomToken = TokenGenerator->GenerateRoomToken(
    "OdinRoomName", "UserId", EOdinTokenAudience::Default);
```

### Joining a Room

```cpp
FString GatewayUrl = "https://gateway.odin.4players.io";
TArray<uint8> UserDataByteArray;
FVector InitialPosition = FVector(0, 0, 0);

FOdinRoomJoinError JoinErrorCallback;
JoinErrorCallback.BindDynamic(this, &UMyClass::OnJoinRoomError);
FOdinRoomJoinSuccess JoinSuccessCallback;
JoinSuccessCallback.BindDynamic(this, &UMyClass::OnJoinRoomSuccess);

UOdinRoomJoin* OdinRoomJoin = UOdinRoomJoin::JoinRoom(
    this, OdinRoomPtr, GatewayUrl, RoomToken,
    UserDataByteArray, InitialPosition,
    JoinErrorCallback, JoinSuccessCallback);
OdinRoomJoin->Activate();
```

### Setup Microphone Input

```cpp
UOdinAudioCapture* Capture = UOdinFunctionLibrary::CreateOdinAudioCapture(this);
UAudioGenerator* CaptureAsGenerator = Cast<UAudioGenerator>(Capture);
CaptureMedia = UOdinFunctionLibrary::Odin_CreateMedia(CaptureAsGenerator);

FOdinRoomAddMediaError AddMediaError;
AddMediaError.BindDynamic(this, &UMyClass::OnAddMediaError);
FOdinRoomAddMediaSuccess AddMediaSuccess;
AddMediaSuccess.BindDynamic(this, &UMyClass::OnAddMediaSuccess);

UOdinRoomAddMedia* OdinRoomAddMedia = UOdinRoomAddMedia::AddMedia(
    this, OdinRoomPtr, CaptureMedia, AddMediaError, AddMediaSuccess);
OdinRoomAddMedia->Activate();
```

### Media Playback (OnMediaAdded Handler)

```cpp
void UMyClass::OnMediaAdded(int64 PeerId, UOdinPlaybackMedia* Media,
                            UOdinJsonObject* Properties, UOdinRoom* Room)
{
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

## APM Settings Reference

| Setting | Description | Recommended Value |
|---------|-------------|---------|
| `bVoiceActivityDetection` | Enable VAD | `true` |
| `fVadAttackProbability` | VAD activation threshold | `0.7` |
| `fVadReleaseProbability` | VAD deactivation threshold | `0.6` |
| `bEnableVolumeGate` | Enable volume gate | `true` |
| `fVolumeGateAttackLoudness` | Gate open threshold (dB) | `-30.0` |
| `fVolumeGateReleaseLoudness` | Gate close threshold (dB) | `-40.0` |
| `bHighPassFilter` | Remove low frequencies | `true` |
| `bEchoCanceller` | Enable echo cancellation | `true` |
| `noise_suppression_level` | Noise reduction level | `Moderate` |

### Echo Cancellation Best Practices

- Enable APM settings with Voice Activity Detection (VAD)
- Recommended VAD Attack/Release probability offset: 0.1
- Recommended Volume Gate Attack/Release offset: 10 dB
- Allow players to customize settings for individual hardware setups

## Common Patterns

### Push-to-Talk

```cpp
void OnPushToTalkPressed() {
    CaptureMedia->SetIsPaused(false);
}

void OnPushToTalkReleased() {
    CaptureMedia->SetIsPaused(true);
}
```

### Position Updates

```cpp
// IMPORTANT: Max 10 calls per second (100ms intervals)
void UpdatePosition() {
    FVector Pos = GetActorLocation();
    Room->UpdatePeerPosition(Pos);
}

// In BeginPlay:
GetWorld()->GetTimerManager().SetTimer(
    PositionTimer, this, &AMyActor::UpdatePosition, 0.1f, true);
```

## Platform Considerations

### Android/Meta Quest

**JoinRoom Callback Issue**: Using standard `FVector2D()` constructor prevents callbacks from triggering.

```cpp
// WRONG - callbacks won't trigger
Room->JoinRoom(URL, Token, UserData, FVector2D());

// CORRECT - explicitly initialize
Room->JoinRoom(URL, Token, UserData, FVector2D(0, 0));
```

### Platform Permissions

- **Android**: Configure microphone permissions in project settings
- **iOS/Apple**: Set up microphone permissions in Info.plist

## Audio Middleware Integration

### FMOD

FMOD Adapter Plugin available for routing ODIN voice chat through FMOD pipeline.

### Wwise

Wwise Adapter Plugin available for integrating ODIN with Wwise audio engine.

## Troubleshooting

See [voice-unreal-troubleshooting.md](voice-unreal-troubleshooting.md) for common issues.

### v1.x Specific Issues

**Android/Meta Quest JoinRoom Callback Issue**: Using standard `FVector2D()` constructor prevents callbacks from triggering.

```cpp
// WRONG - callbacks won't trigger
Room->JoinRoom(URL, Token, UserData, FVector2D());

// CORRECT - explicitly initialize
Room->JoinRoom(URL, Token, UserData, FVector2D(0, 0));
```

## Testing

Use ODIN Web Client (https://4players.app):
- Same access key as game
- Same gateway URL and room name
- Cross-platform testing without multiple game instances

## Learning Resources

### Sample Projects

- **Unreal Sample Project** - Basic integration (GitHub)
- **Unreal Minimal Samples** - Multiplayer proximity voice
- **Unreal Tech Demo** - Spatial audio, occlusion
- **Unreal C++ Sample** - Full C++ patterns

### Documentation

- Blueprint Reference: https://docs.4players.io/voice/unreal/blueprint-reference/
- Guides: https://docs.4players.io/voice/unreal/guides/
- FAQ: https://docs.4players.io/voice/unreal/faq/
