---
name: odin-voice-core
description: |
  ODIN Voice Core SDK (C/C++) - the low-level foundation for all ODIN SDKs.
  Use when: building custom platform integrations, working with C/C++ directly,
  understanding the underlying ODIN architecture, or creating bindings for other languages.
  Enables seamless integration of real-time voice chat into multiplayer games, applications, and websites.
  This is the base SDK that Unity, Unreal, Web, and Swift SDKs are built upon.
---

# ODIN Voice Core SDK (C/C++)

Low-level C API foundation for all ODIN Voice SDKs, enabling real-time voice chat integration.

## Overview

The Core SDK is:
- **Built with Rust**, exposing a standard C API for cross-platform compatibility
- **Foundation for all platform SDKs** (Unity, Unreal, Web, Swift, NodeJS)
- **Event-driven architecture** based on RPC (Remote Procedure Calls) with MessagePack encoding
- **Connection Pool pattern** for efficient connection management
- **JWT-based authentication** using Ed25519 key pairs
- **Advanced audio processing** with Opus codec, VAD, echo cancellation, and noise suppression
- **Flexible deployment** - use 4Players cloud or self-hosted infrastructure

## Supported Platforms

ODIN provides native binaries for:

| Platform | x86_64 | aarch64 | x86 | armv7a |
|----------|--------|---------|-----|--------|
| Windows  | ✅     | ✅      | ✅  | —      |
| Linux    | ✅     | ✅      | ✅  | —      |
| macOS    | ✅     | ✅      | —   | —      |
| Android  | ✅     | ✅      | ✅  | ✅     |
| iOS      | ✅     | ✅      | —   | —      |

Console platforms (Xbox, PlayStation, Nintendo Switch) available upon request.

## Installation

**Repository**: https://github.com/4Players/odin-sdk

**Prerequisites**:
- CMake 3.10+
- C++17 compatible compiler
- Git with LFS support

**Steps**:
```bash
git clone https://github.com/4Players/odin-sdk.git
cd odin-sdk
git lfs fetch
git lfs checkout
```

**Integration**:
1. Include `odin.h` from the `include/` directory
2. Link against the ODIN library for your platform from `bin/`
3. Initialize with `odin_initialize(ODIN_VERSION)`

## Quick Start

```c
#include "odin.h"

int main() {
    // 1. Initialize SDK
    odin_initialize(ODIN_VERSION);

    // 2. Generate access key (for testing only - use server-side in production!)
    char access_key[128];
    odin_access_key_generate(access_key, sizeof(access_key));

    // 3. Create token generator
    OdinTokenGenerator* generator = odin_token_generator_create(access_key);

    // 4. Generate room token with room ID and user ID
    char room_token[512];
    odin_token_generator_create_token(generator, "MyRoom", "User1",
                                       room_token, sizeof(room_token));
    odin_token_generator_destroy(generator);

    // 5. Create connection pool with callbacks
    OdinConnectionPool* pool = odin_connection_pool_create();
    odin_connection_pool_set_event_callback(pool, on_rpc_event, NULL);
    odin_connection_pool_set_datagram_callback(pool, on_audio_datagram, NULL);

    // 6. Create and join room
    OdinRoomHandle room = odin_room_create();
    odin_room_set_event_callback(room, handle_odin_event, NULL);
    odin_room_join(room, "https://gateway.odin.4players.io", room_token);

    // 7. Create and add audio stream
    OdinAudioStreamConfig config = {
        .sample_rate = 48000,
        .channel_count = 1
    };
    OdinMediaStreamHandle media = odin_audio_stream_create(config);
    odin_room_add_media(room, media);

    // ... main loop with audio processing ...

    // 8. Cleanup
    odin_room_remove_media(room, media);
    odin_room_close(room);
    odin_room_destroy(room);
    odin_connection_pool_destroy(pool);
    return 0;
}
```

## Architecture & Key Concepts

### Connection Pool
ODIN uses a Connection Pool pattern for efficient connection management:
- Handles both RPC signaling events and audio datagrams
- Separate callbacks for control messages (`on_rpc`) and voice data (`on_datagram`)
- Manages multiple room connections efficiently

### RPC Event System
- **Event-driven architecture** using Remote Procedure Calls
- **MessagePack encoding** for efficient binary serialization
- Events are delivered as structured data that can be deserialized to JSON
- **Visitor pattern** recommended for routing events to appropriate handlers

### Authentication & Security
- **JWT-based authentication** using signed JSON Web Tokens
- **Ed25519 key pairs** for cryptographic signing
- Tokens contain `"rid"` (room ID) and `"uid"` (user ID) claims
- **CRITICAL**: Access keys are sensitive credentials - never embed in client-side code!
- **Production deployment**: Generate tokens server-side
- **Testing only**: Use `OdinTokenGenerator` for quick prototyping

### Media Signaling
- **Explicit signaling** for media stream control
- Operations: Start, Stop, Pause, Resume
- When joining, send `StartMedia` RPC to announce your stream
- Handle `PeerUpdateMediaStarted` events to configure decoders for remote peers
- Direct callbacks on encoders/decoders for performance-critical updates

## Key Functions

### SDK Initialization
```c
// Initialize ODIN SDK (call once at startup)
int odin_initialize(const char* version);  // Use ODIN_VERSION constant
```

### Access Key Management
```c
// Generate new access key
int odin_access_key_generate(char* buffer, size_t buffer_size);

// Get key ID from access key
int odin_access_key_id(const char* access_key, char* key_id, size_t key_id_size);
```

### Token Generation
```c
// Create token generator
OdinTokenGenerator* odin_token_generator_create(const char* access_key);

// Generate room token
int odin_token_generator_create_token(
    OdinTokenGenerator* generator,
    const char* room_id,
    const char* user_id,
    char* token_buffer,
    size_t buffer_size
);

// Cleanup
void odin_token_generator_destroy(OdinTokenGenerator* generator);
```

### Connection Pool Management
```c
// Create connection pool
OdinConnectionPool* odin_connection_pool_create();

// Set RPC event callback (handles signaling/events)
int odin_connection_pool_set_event_callback(
    OdinConnectionPool* pool,
    OdinRpcCallback callback,
    void* user_data
);

// Set datagram callback (handles audio packets)
int odin_connection_pool_set_datagram_callback(
    OdinConnectionPool* pool,
    OdinDatagramCallback callback,
    void* user_data
);

// Cleanup
void odin_connection_pool_destroy(OdinConnectionPool* pool);
```

### Room Management
```c
// Create room handle
OdinRoomHandle odin_room_create();

// Set event callback
int odin_room_set_event_callback(
    OdinRoomHandle room,
    OdinEventCallback callback,
    void* user_data
);

// Join room
int odin_room_join(
    OdinRoomHandle room,
    const char* gateway_url,
    const char* room_token
);

// Leave room
int odin_room_close(OdinRoomHandle room);

// Destroy room
void odin_room_destroy(OdinRoomHandle room);
```

### Media Handling
```c
// Add audio input (microphone)
OdinMediaStreamHandle odin_audio_stream_create(OdinAudioStreamConfig config);

// Add media to room
int odin_room_add_media(OdinRoomHandle room, OdinMediaStreamHandle media);

// Remove media from room
int odin_room_remove_media(OdinRoomHandle room, OdinMediaStreamHandle media);

// Push audio samples to media stream
int odin_audio_push_data(
    OdinMediaStreamHandle media,
    const float* samples,
    size_t sample_count
);

// Read audio samples from media stream
int odin_audio_read_data(
    OdinMediaStreamHandle media,
    float* buffer,
    size_t buffer_size,
    OdinChannelLayout layout
);
```

### Position & Messaging
```c
// Update 3D position
int odin_room_update_position(OdinRoomHandle room, float x, float y, float z);

// Update user data
int odin_room_update_peer_user_data(
    OdinRoomHandle room,
    const uint8_t* data,
    size_t data_size
);

// Send message
int odin_room_send_message(
    OdinRoomHandle room,
    const uint64_t* target_ids,
    size_t target_count,
    const uint8_t* data,
    size_t data_size
);
```

## Event Callback

```c
typedef void (*OdinEventCallback)(
    OdinRoomHandle room,
    const OdinEvent* event,
    void* user_data
);

void handle_odin_event(OdinRoomHandle room, const OdinEvent* event, void* user_data) {
    switch (event->tag) {
        case OdinEvent_Joined:
            printf("Joined room, own peer ID: %llu\n", event->joined.own_peer_id);
            break;

        case OdinEvent_PeerJoined:
            printf("Peer %llu joined\n", event->peer_joined.peer_id);
            break;

        case OdinEvent_PeerLeft:
            printf("Peer %llu left\n", event->peer_left.peer_id);
            break;

        case OdinEvent_MediaAdded:
            printf("Media %u added from peer %llu\n",
                   event->media_added.media_handle,
                   event->media_added.peer_id);
            break;

        case OdinEvent_MediaRemoved:
            printf("Media %u removed\n", event->media_removed.media_handle);
            break;

        case OdinEvent_MediaActiveStateChanged:
            printf("Media %u active: %s\n",
                   event->media_active_state_changed.media_handle,
                   event->media_active_state_changed.active ? "yes" : "no");
            break;

        case OdinEvent_MessageReceived:
            printf("Message from peer %llu: %.*s\n",
                   event->message_received.peer_id,
                   (int)event->message_received.data_len,
                   event->message_received.data);
            break;

        case OdinEvent_ConnectionStateChanged:
            printf("Connection state: %d, reason: %d\n",
                   event->connection_state_changed.state,
                   event->connection_state_changed.reason);
            break;
    }
}
```

## Event Types

| Event | Description |
|-------|-------------|
| `OdinEvent_Joined` | Successfully joined room |
| `OdinEvent_PeerJoined` | Remote peer joined |
| `OdinEvent_PeerLeft` | Remote peer left |
| `OdinEvent_PeerUserDataChanged` | Peer data updated |
| `OdinEvent_MediaAdded` | Media stream started |
| `OdinEvent_MediaRemoved` | Media stream stopped |
| `OdinEvent_MediaActiveStateChanged` | Speaking state changed |
| `OdinEvent_MessageReceived` | Data message received |
| `OdinEvent_RoomUserDataChanged` | Room data updated |
| `OdinEvent_ConnectionStateChanged` | Connection status changed |

## Audio Configuration

```c
OdinAudioStreamConfig config = {
    .sample_rate = 48000,
    .channel_count = 1
};

OdinMediaStreamHandle media = odin_audio_stream_create(config);
```

## APM (Audio Processing Module)

```c
OdinApmConfig apm = {
    .voice_activity_detection = true,
    .voice_activity_detection_attack_probability = 0.9f,
    .voice_activity_detection_release_probability = 0.8f,
    .volume_gate = true,
    .volume_gate_attack_loudness = -30.0f,
    .volume_gate_release_loudness = -40.0f,
    .echo_canceller = true,
    .high_pass_filter = true,
    .noise_suppression_level = OdinNoiseSuppressionLevel_Moderate,
    .transient_suppressor = true,
    .gain_controller = OdinGainControllerVersion_V2
};

odin_room_configure_apm(room, apm);
```

## Error Handling

```c
int result = odin_room_join(room, gateway, token);

if (odin_is_error(result)) {
    char error_buffer[256];
    odin_error_format(result, error_buffer, sizeof(error_buffer));
    printf("Error: %s\n", error_buffer);
}
```

## Common Patterns

### Audio Loop
```c
// Input thread - capture microphone
void audio_input_thread(OdinMediaStreamHandle media) {
    float samples[480];  // 10ms at 48kHz
    while (running) {
        capture_microphone(samples, 480);
        odin_audio_push_data(media, samples, 480);
    }
}

// Output thread - playback remote audio
void audio_output_thread(OdinMediaStreamHandle media) {
    float buffer[480];
    while (running) {
        int samples_read = odin_audio_read_data(
            media, buffer, 480, OdinChannelLayout_Mono);
        if (samples_read > 0) {
            playback_audio(buffer, samples_read);
        }
    }
}
```

### Multi-Room
```c
OdinRoomHandle room1 = odin_room_create();
OdinRoomHandle room2 = odin_room_create();

// Join both rooms
odin_room_join(room1, gateway, token1);
odin_room_join(room2, gateway, token2);

// Each room needs its own media stream
OdinMediaStreamHandle media1 = odin_audio_stream_create(config);
OdinMediaStreamHandle media2 = odin_audio_stream_create(config);

odin_room_add_media(room1, media1);
odin_room_add_media(room2, media2);
```

## Architecture Notes

- **Connection pool pattern** for efficient connection management
- **Event-driven via RPC** (Remote Procedure Calls) with MessagePack encoding
- **Explicit media signaling** (Start/Stop/Pause/Resume)
- **JWT authentication** with Ed25519 signing, `"rid"` (room ID) and `"uid"` (user ID) claims
- **Opus codec compression** with automatic resampling
- **Multi-channel audio** support within rooms
- **No server-side user data storage** required

## Code Samples

### Minimal Command Line Client
Complete example demonstrating:
- Connection Pool setup with callbacks
- RPC event handling with MessagePack deserialization
- Media signaling flow (StartMedia RPC)
- Bidirectional audio with miniaudio integration
- Encoder/decoder configuration

**Location**: `test/` directory in the SDK repository
**Build**: Requires CMake 3.10+, C++17 compiler
**Run**: `./odin_minimal -r <room_id> -s <server_url>`

## Best Practices

1. **Authentication**: Always generate JWT tokens server-side in production
2. **Error Handling**: Check all return values with `odin_is_error()` and format errors
3. **Audio Threading**: Separate threads for capture (push) and playback (read)
4. **Resource Cleanup**: Always destroy rooms, media streams, and pools in reverse creation order
5. **APM Configuration**: Enable audio processing (VAD, echo cancellation) before joining rooms
6. **Multi-Room**: Each room requires its own media stream instance
7. **Event Processing**: Use visitor pattern or switch statements for efficient event routing
8. **Production Deployment**: Choose between 4Players cloud or self-hosted infrastructure

## Documentation & Resources

- **Full API Reference**: https://docs.4players.io/voice/core/
- **GitHub Repository**: https://github.com/4Players/odin-sdk
- **Minimal Client Sample**: https://docs.4players.io/voice/core/samples/minimal-client/
- **Event Structures**: Complete RPC event schemas in documentation
- **Platform SDKs**: Unity, Unreal, Web, Swift, NodeJS built on this Core SDK
