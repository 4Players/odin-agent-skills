# ODIN Voice Unreal SDK 2.x

Unreal Engine plugin for real-time voice chat with full Blueprint and C++ support.

> **Version Compatibility**: ODIN 1.x and 2.x are **not interoperable**. Clients using SDK 1.x cannot communicate with clients using SDK 2.x.

## Overview

The v2.x SDK introduces significant architectural improvements:

- **Encoder/Decoder pattern** replaces Media objects
- **Audio Pipeline** with extensible effect chains (VAD, APM, custom)
- **Implicit connection pooling** - automatic network management
- **64-channel spatial audio** with channel masks and 3D positioning
- **End-to-end encryption** support via `OdinCipher`

### Key Changes from v1.x

| v1.x | v2.x |
|------|------|
| `UOdinCaptureMedia` / `UOdinPlaybackMedia` | `UOdinEncoder` / `UOdinDecoder` |
| `ConstructRoom(APMSettings)` | `ConstructRoom()` - no APM parameter |
| `UOdinRoomJoin::JoinRoom()` latent | `UOdinRoom::ConnectRoom()` sync |
| `OnMediaAdded` + `Odin_AssignSynthToMedia()` | `OnPeerJoined` + `RegisterDecoder()` + `SetDecoder()` |
| Room-level APM config | Per-encoder `UOdinPipeline` effects |

## Quick Start

### Basic Blueprint Flow

```
BeginPlay:
1. Construct Token Generator (access key)
2. Generate Room Token (room name, user name) → returns AuthJson
3. Construct Room Handle (no APM settings)
4. Bind event delegates (OnRoomPeerJoined, OnRoomJoined)
5. Connect Room (gateway URL, AuthJson)
6. On Room Joined: Create Audio Capture → Create Encoder from Generator
7. On Peer Joined: Create Decoder → Register to Peer → Assign to Synth
```

### Security Note

> **CRITICAL**: Only embed access keys in client code for testing purposes, __NEVER__ in production code. Generate room tokens on a server for production deployments.

## Key Blueprint Nodes

### Setup Nodes

| Node | Purpose |
|------|---------|
| `Construct a Token Generator` | Create token generator with access key |
| `Generate Room Token` | Generate auth token + AuthJson object |
| `Construct Room` | Create room instance (no APM parameter) |

### Room Management

| Node | Purpose |
|------|---------|
| `Connect Room` | Connect to ODIN room (sync, returns success bool) |
| `Send Rpc` | Send JSON RPC commands |

### Audio Capture (Encoder)

| Node | Purpose |
|------|---------|
| `Create Odin Audio Capture` | Create microphone capture |
| `Create Odin Encoder from Generator` | Create encoder from audio capture |
| `Start Capturing Audio` | Begin microphone capture |
| `Get Or Create Pipeline` | Access encoder's audio pipeline |

### Audio Playback (Decoder)

| Node | Purpose |
|------|---------|
| `Construct Decoder` | Create decoder (sample rate, stereo) |
| `Register Decoder to Peer` | Link decoder to room and peer |
| `Add Odin Synth Component` | Create audio output |
| `Set Decoder` | Assign decoder to synth component |

### Audio Pipeline

| Node | Purpose |
|------|---------|
| `Insert Apm Effect` | Add APM effect to pipeline |
| `Set Apm Config` | Configure APM settings |
| `Insert Vad Effect` | Add VAD effect to pipeline |
| `Set Vad Config` | Configure VAD settings |
| `Insert Custom Effect` | Add custom audio effect |

## Event Sequence

**Critical**: Bind event handlers BEFORE connecting to room.

```
1. Connect Room Called
2. On Room Status Changed (joining)
3. On Room Peer Joined (existing peers)  ← Create decoder for each peer
4. On Room Joined                        ← Setup microphone encoder
5. On Room Peer Joined (new)             ← Create decoder for new peers
6. On Room Peer Left                     ← Cleanup decoder and synth
```

### Event Changes from v1.x

| v1.x Event | v2.x Replacement |
|------------|------------------|
| `On Media Added` | Removed - create decoder in `On Peer Joined` |
| `On Media Removed` | Removed |
| `On Media Active State Changed` | `On Audio Event` (Is Silent Changed) |
| `On Message Received` | `On Room Message Received` |
| `On Peer Joined` | `On Room Peer Joined` (params in struct) |
| `On Peer Left` | `On Room Peer Left` (params in struct) |
| `On Peer User Data Changed` | `On Room Peer Changed` |
| `On Room Connection State Changed` | `On Room Status Changed` |

### New Events

| Event | Description |
|-------|-------------|
| `On Room New Reconnect Token` | New reconnect token available |
| `On Rpc` | Control and event messages |
| `On Audio Event` | Encoder/decoder state changes |

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
UOdinEncoder*           // Outgoing audio encoder
UOdinDecoder*           // Incoming audio decoder
UOdinPipeline*          // Audio effect pipeline
UOdinAudioCapture*      // Microphone input
UOdinSynthComponent*    // Audio output component
UOdinJsonObject*        // JSON helper for auth data
```

### Complete Component Example

Header:
```cpp
#pragma once

#include "OdinTokenGenerator.h"
#include "OdinRoom.h"
#include "OdinAudio/OdinAudioCapture.h"
#include "OdinAudio/OdinEncoder.h"
#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "OdinClientComponent.generated.h"

UCLASS(ClassGroup=(Custom), meta=(BlueprintSpawnableComponent))
class UOdinClientComponent : public UActorComponent
{
    GENERATED_BODY()

public:
    UOdinClientComponent();
    void ConnectToOdin(FGuid PlayerId);

protected:
    UPROPERTY() UOdinTokenGenerator* TokenGenerator;
    UPROPERTY() FString RoomToken;
    UPROPERTY() UOdinRoom* Room;
    UPROPERTY() UOdinAudioCapture* Capture;
    UPROPERTY() UOdinEncoder* Encoder;

    UFUNCTION() void OnRoomJoinSuccessHandler(UOdinRoom* OdinRoom, FOdinJoined Data);
    UFUNCTION() void OnPeerJoinedHandler(UOdinRoom* OdinRoom, FOdinPeerJoined PeerData);
};
```

### Token Generation

```cpp
TokenGenerator = UOdinTokenGenerator::ConstructTokenGenerator(this, "<ACCESS_KEY>");
UOdinJsonObject* AuthJson;
TokenGenerator->GenerateRoomToken("TestRoom", "Player", AuthJson, RoomToken);
```

### Room Construction and Connection

```cpp
// Construct room (no APM settings in v2)
Room = UOdinRoom::ConstructRoom(this);

// Bind delegates BEFORE connecting
Room->OnRoomPeerJoinedBP.AddUniqueDynamic(this, &UMyClass::OnPeerJoinedHandler);
Room->OnRoomJoinedBP.AddUniqueDynamic(this, &UMyClass::OnRoomJoinSuccessHandler);

// Add PlayerId to user data
UOdinJsonObject* UserDataObject = UOdinJsonObject::ConstructJsonObject(this);
UserDataObject->SetStringField("PlayerId", *PlayerId.ToString());
AuthJson->SetObjectField("user_data", UserDataObject);

// Connect with authentication JSON
bool bSuccess = false;
Room->ConnectRoom("https://gateway.odin.4players.io", AuthJson->EncodeJson(), bSuccess);
```

### Microphone Setup (OnRoomJoined Handler)

```cpp
void UMyClass::OnRoomJoinSuccessHandler(UOdinRoom* OdinRoom, FOdinJoined Data)
{
    // Create capture
    Capture = UOdinFunctionLibrary::CreateOdinAudioCapture(this);

    // Create encoder and link to room
    UAudioGenerator* CaptureAsGenerator = Cast<UAudioGenerator>(Capture);
    Encoder = UOdinFunctionLibrary::CreateOdinEncoderFromGenerator(this, Room, CaptureAsGenerator);

    // Configure audio pipeline (see APM section below)
    ConfigureAudioPipeline(Encoder);

    // Start capturing
    Capture->StartCapturingAudio();
}
```

### Audio Playback (OnPeerJoined Handler)

```cpp
void UMyClass::OnPeerJoinedHandler(UOdinRoom* OdinRoom, FOdinPeerJoined PeerData)
{
    // Parse PlayerId from user data
    auto JSON = UOdinJsonObject::ConstructJsonObjectFromString(this, PeerData.user_data);
    FString GUIDString = JSON->GetStringField(TEXT("PlayerId"));
    
    FGuid GUID;
    if (FGuid::Parse(GUIDString, GUID))
    {
        ACharacter* Character = FindCharacterByGUID(GUID);
        if (Character)
        {
            // 1. Create decoder (48000Hz, stereo)
            UOdinDecoder* Decoder = UOdinDecoder::ConstructDecoder(this, 48000, true);

            // 2. Register decoder to room and peer
            UOdinFunctionLibrary::RegisterDecoder(Decoder, OdinRoom, PeerData.peer_id);

            // 3. Create synth component on character
            UActorComponent* Comp = Character->AddComponentByClass(
                UOdinSynthComponent::StaticClass(), false, FTransform::Identity, false);
            UOdinSynthComponent* Synth = Cast<UOdinSynthComponent>(Comp);

            // 4. Assign decoder to synth
            Synth->SetDecoder(Decoder);

            // 5. Configure 3D audio
            FSoundAttenuationSettings AttenuationSettings;
            AttenuationSettings.bSpatialize = true;
            AttenuationSettings.bAttenuate = true;
            Synth->AdjustAttenuation(AttenuationSettings);

            // 6. Activate
            Synth->Activate();
        }
    }
}
```

## Audio Pipeline Configuration

APM and VAD are now configured via the encoder's audio pipeline:

```cpp
void ConfigureAudioPipeline(UOdinEncoder* Encoder)
{
    if (UOdinPipeline* Pipeline = Encoder->GetOrCreatePipeline())
    {
        // Insert APM effect at index 0
        const int32 ApmId = Pipeline->InsertApmEffect(0, 48000, true);

        FOdinApmConfig ApmConfig;
        ApmConfig.echo_canceller = false;
        ApmConfig.noise_suppression = EOdinNoiseSuppression::ODIN_NOISE_SUPPRESSION_MODERATE;
        ApmConfig.high_pass_filter = true;
        ApmConfig.gain_controller = EOdinGainControllerVersion::ODIN_GAIN_CONTROLLER_V2;
        ApmConfig.transient_suppressor = true;
        Pipeline->SetApmConfig(ApmId, ApmConfig);

        // Insert VAD effect at index 1
        const int32 VadId = Pipeline->InsertVadEffect(1);

        FOdinVadConfig VadConfig;
        VadConfig.VoiceActivity = {
            .Enabled = true,
            .AttackThreshold = 0.7f,
            .ReleaseThreshold = 0.6f
        };
        VadConfig.VolumeGate = {
            .Enabled = true,
            .AttackThreshold = -30.0f,
            .ReleaseThreshold = -40.0f
        };
        Pipeline->SetVadConfig(VadId, VadConfig);
    }
}
```

### APM Config Options

| Setting | Type | Description |
|---------|------|-------------|
| `echo_canceller` | bool | Enable echo cancellation |
| `high_pass_filter` | bool | Remove low frequencies |
| `transient_suppressor` | bool | Keyboard click detection |
| `noise_suppression` | enum | None/Low/Moderate/High/VeryHigh |
| `gain_controller` | enum | None/V1/V2 |

### VAD Config Options

| Setting | Description |
|---------|-------------|
| `VoiceActivity.Enabled` | Enable voice activity detection |
| `VoiceActivity.AttackThreshold` | Activation threshold (0.0-1.0) |
| `VoiceActivity.ReleaseThreshold` | Deactivation threshold (0.0-1.0) |
| `VolumeGate.Enabled` | Enable volume gate |
| `VolumeGate.AttackThreshold` | Gate open threshold (dBFS) |
| `VolumeGate.ReleaseThreshold` | Gate close threshold (dBFS) |

## New Features in v2.x

### 64-Channel Spatial Audio

V2 supports up to 64 audio channels per room with 3D positioning:

```cpp
#include "OdinAudio/OdinEncoder.h"
#include "OdinAudio/OdinDecoder.h"
#include "OdinAudio/OdinChannelMask.h"

// Set encoder position on channel 1
FOdinChannelMask ChannelMask(0x01);  // Channel 1
FOdinPosition Position;
Position.X = 10.0f;
Position.Y = 0.0f;
Position.Z = 5.0f;
Encoder->SetPosition(ChannelMask, Position);

// Clear position for a channel
Encoder->ClearPosition(ChannelMask);

// Get active channels from decoder
int64 ActiveChannels = Decoder->GetActiveChannelMask();

// Get positions for specific channels
TArray<FOdinPosition> Positions = Decoder->GetPositions(ChannelMask);
```

### Channel Masks

Filter which peers you hear using channel masks. Use `UOdinRoom::SetChannelMasks()`:

```cpp
#include "OdinRoom.h"

// Create channel mask request
FOdinSetChannelMasks Request;
Request.reset = true;  // Clear existing masks first

// Set peer 1 to only hear channels 1, 3, 5 (binary 10101 = 21)
Request.masks.Add(1, FOdinChannelMask(21));

// Send RPC
Room->SetChannelMasks(Request);

// Alternative: use TMap directly
TMap<int64, uint64> Masks;
Masks.Add(1, 21);  // Peer 1 hears channels 1, 3, 5
Room->SetChannelMasks(Masks, true);
```

**Channel Mask Helper Functions** (from `UOdinFunctionLibrary`):

```cpp
// Create mask with specific channels enabled
TArray<int32> EnabledChannels = {1, 3, 5};
FOdinChannelMask Mask = UOdinFunctionLibrary::CreateChannelMaskFromEnabled(EnabledChannels);

// Check if a channel is enabled
bool bIsChannelEnabled = UOdinFunctionLibrary::IsChannelEnabledInMask(Mask, 3);

// Create full mask (all 64 channels)
FOdinChannelMask FullMask = UOdinFunctionLibrary::CreateFullMask();

// Create empty mask (no channels)
FOdinChannelMask EmptyMask = UOdinFunctionLibrary::CreateEmptyMask();
```

Use cases:
- Team-only voice chat
- Proximity channels
- Spectator modes

### End-to-End Encryption

Secure voice data with `UOdinCrypto`:

```cpp
#include "OdinCryptoExtension.h"
#include "OdinRoom.h"

// Create room with encryption
UOdinRoom* Room = UOdinRoom::ConstructRoom(this);

// Create crypto extension with secret (as byte array)
TArray<uint8> Secret;
// ... fill secret bytes ...
UOdinCrypto* Crypto = UOdinCrypto::ConstructCrypto(this, Secret);

// Connect with encryption enabled
bool bSuccess;
Room->ConnectRoom(GatewayUrl, AuthJson, bSuccess, Crypto);

// Alternative: set password after room creation
Room->SetPassword(TEXT("MySecretPassword"));

// Check peer encryption status
EOdinCryptoPeerStatus Status = Crypto->GetPeerStatus(PeerId);
// ENCRYPTED, UNENCRYPTED, UNKNOWN, or INVALID_PASSWORD
```

> **Note**: Requires `ODIN_USE_CRYPTO` define and CryptoExtension library. `ODIN_USE_CRYPTO` is defined by default.

## 3D Proximity Voice Pattern

Same pattern as v1.x, but decoder-based:

```
1. Each character has replicated PlayerId (GUID)
2. Store mapping: PlayerId → Character* in GameInstance
3. On peer join: Extract PlayerId from user_data, create decoder
4. Register decoder to peer
5. Create synth on character, assign decoder
```

## Platform Considerations

### Android/Meta Quest

Microphone permissions must be configured in project settings.

### iOS/Apple

Set up microphone permissions in Info.plist.

## Migration from v1.x

For detailed migration steps, see the [Migration Guide](https://docs.4players.io/voice/unreal/guides/migration-v1-to-v2/).

Key changes:
1. Replace `ConstructRoom(APMSettings)` with `ConstructRoom()`
2. Replace `JoinRoom` latent action with `ConnectRoom`
3. Replace `OnMediaAdded` with decoder creation in `OnPeerJoined`
4. Replace `Odin_AssignSynthToMedia` with `SetDecoder`
5. Configure APM via encoder's audio pipeline

## Troubleshooting

See [voice-unreal-troubleshooting.md](voice-unreal-troubleshooting.md) for common issues.

### v2.x Specific Issues

**Decoder Not Receiving Audio**

- Ensure `RegisterDecoder` called with correct room and peer_id
- Verify decoder sample rate matches playback requirements (48000Hz recommended)
- Check that synth component is activated after `SetDecoder`

## Documentation

- Online Docs: https://docs.4players.io/voice/unreal/
- Migration Guide: https://docs.4players.io/voice/unreal/guides/migration-v1-to-v2/
- Blueprint Reference: https://docs.4players.io/voice/unreal/blueprint-reference/
- C++ Sample: https://github.com/4Players/odin-unreal-cpp-sample
