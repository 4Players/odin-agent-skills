---
name: odin-voice-swift
description: |
  ODIN Voice Swift SDK (OdinKit) for real-time voice chat in iOS and macOS apps.
  Use when: implementing voice chat in Swift/SwiftUI projects, working with OdinRoom/OdinPeer classes,
  handling delegates or Combine publishers, configuring audio processing.
  Requires iOS 9+/macOS 10.15+, Swift 5.0+. For ODIN concepts, see odin-fundamentals skill.
license: MIT
---

# ODIN Voice Swift SDK (OdinKit)

> **⚠️ DEPRECATION NOTICE:** This documentation refers to an **outdated version** of the ODIN core SDK. OdinKit is subject to be updated with breaking API changes. Please check the official repository for the latest API.

Swift wrapper for real-time VoIP chat on iOS and macOS.

## Requirements

- iOS 9.0+ / macOS 10.15+
- Xcode 10.2+
- Swift 5.0+

## Quick Start

```swift
// 1. Create access key (do this on backend in production)
let accessKey = try OdinAccessKey("YOUR_ACCESS_KEY")
// Or create a new access key for testing:
// let accessKey = OdinAccessKey()

// 2. Generate token
let token = try accessKey.generateToken(roomId: "MyRoom", userId: "user123")

// 3. Create room and set delegate
let room = try OdinRoom(gateway: "https://gateway.odin.4players.io")
room.delegate = self

// 4. Join room
try room.join(token: token)

// 5. Add microphone
try room.addMedia(type: OdinMediaStreamType_Audio)
```

## Key Classes

### OdinAccessKey
Manages authentication credentials.

```swift
// Create from string
let accessKey = try OdinAccessKey("YOUR_44_CHAR_ACCESS_KEY")

// Properties
accessKey.rawValue      // Original string
accessKey.id            // Key identifier
accessKey.publicKey     // Public key bytes
accessKey.secretKey     // Secret key bytes

// Generate token
let token = try accessKey.generateToken(roomId: "RoomName", userId: "UserId")
```

### OdinRoom
Virtual communication space.

```swift
// Properties
room.id                 // Room identifier
room.customer           // Customer identifier
room.userData           // Room user data ([UInt8])
room.connectionStatus   // Current status tuple (state, reason)
room.peers              // Connected peers dictionary [UInt64: OdinPeer]
room.medias             // Audio streams dictionary
room.ownPeer            // Self reference

// Observable (Combine)
room.$connectionStatus  // Published status
room.$userData          // Published user data
room.$peers             // Published peers
room.$medias            // Published medias

// Methods
try room.join(token: token)
room.leave()
try room.addMedia(type: OdinMediaStreamType_Audio)
try room.addMedia(audioConfig: OdinAudioStreamConfig(sample_rate: 48000, channel_count: 1))
try room.updatePeerUserData(userData: bytes)
try room.updatePosition(x: 100, y: 50)
room.setPositionScale(scale: 1.0)
try room.sendMessage(data: bytes)                      // Broadcast to all
try room.sendMessage(data: bytes, targetIds: [1, 2])   // Send to specific peers
```

### OdinPeer
Represents a connected participant.

```swift
// Properties
peer.id                 // Unique peer ID
peer.userId             // User identifier
peer.userData           // Peer metadata
peer.medias             // Associated media streams
peer.activeMedias       // Currently active streams

// Self reference
let myPeer = room.ownPeer
```

### OdinMedia
Audio stream (local or remote).

```swift
// Properties
media.streamHandle      // Handle for operations
media.id                // Media identifier
media.peerId            // Owner peer ID
media.remote            // Is from remote peer
media.type              // Media type (.Audio)
media.activityStatus    // Currently active/speaking
media.audioNode         // For audio mixing
```

### OdinManager
Manages multiple rooms.

```swift
// Singleton
let manager = OdinManager.sharedInstance()

// Properties
manager.rooms           // [String: OdinRoom] dictionary
```

## Event Handling

### Option A: Delegate Pattern

Implement `OdinRoomDelegate`:

```swift
class VoiceManager: OdinRoomDelegate {
    func onRoomConnectionStateChanged(room: OdinRoom, oldState: OdinRoomConnectionState,
                                       newState: OdinRoomConnectionState, reason: OdinRoomConnectionStateChangeReason) {
        print("Connection: \(oldState.rawValue) → \(newState.rawValue)")
    }

    func onRoomJoined(room: OdinRoom) {
        print("Joined as peer \(room.ownPeer.id)")
    }

    func onPeerJoined(room: OdinRoom, peer: OdinPeer) {
        print("Peer \(peer.id) joined with userId '\(peer.userId)'")
    }

    func onPeerLeft(room: OdinRoom, peer: OdinPeer) {
        print("Peer \(peer.id) left")
    }

    func onMediaAdded(room: OdinRoom, peer: OdinPeer, media: OdinMedia) {
        print("Peer \(peer.id) added media \(media.id)")
    }

    func onMediaRemoved(room: OdinRoom, peer: OdinPeer, media: OdinMedia) {
        print("Peer \(peer.id) removed media \(media.id)")
    }

    func onMediaActiveStateChanged(room: OdinRoom, peer: OdinPeer, media: OdinMedia) {
        print("Peer \(peer.id) \(media.activityStatus ? "started" : "stopped") talking on media \(media.id)")
    }

    func onMessageReceived(room: OdinRoom, senderId: UInt64, data: [UInt8]) {
        print("Peer \(senderId) sent message: \(data)")
    }

    func onRoomUserDataChanged(room: OdinRoom) {
        print("Room data updated: \(room.userData)")
    }

    func onPeerUserDataChanged(room: OdinRoom, peer: OdinPeer) {
        print("Peer \(peer.id) data updated: \(peer.userData)")
    }
}
```

### Option B: Combine Publishers

```swift
import Combine

// Monitor the room connection status
room.$connectionStatus.sink {
    print("New Connection Status: \($0.state.rawValue)")
}

// Monitor the room user data
room.$userData.sink {
    print("New User Data: \($0)")
}

// Monitor the list of peers in the room
room.$peers.sink {
    print("New Peers: \($0.keys)")
}

// Monitor the list of media streams in the room
room.$medias.sink {
    print("New Medias: \($0.keys)")
}
```

## Audio Processing (APM)

```swift
// Create APM config with all settings
let apmConfig = OdinApmConfig(
    voice_activity_detection: true,
    voice_activity_detection_attack_probability: 0.9,
    voice_activity_detection_release_probability: 0.8,
    volume_gate: true,
    volume_gate_attack_loudness: -30,   // dBFS
    volume_gate_release_loudness: -40,
    echo_canceller: true,
    high_pass_filter: true,
    pre_amplifier: false,
    noise_suppression_level: OdinNoiseSuppressionLevel_Moderate,  // or _Low, _High, _VeryHigh
    transient_suppressor: true,
    gain_controller: true
)

// Apply config
try room.updateAudioConfig(apmConfig)
```

## User Data & Messaging

### Setting User Data
```swift
// Using OdinCustomData helper with a String
let stringData = OdinCustomData.encode("Hello World!")
try room.updatePeerUserData(userData: stringData)

// Using OdinCustomData helper with a Codable type
struct PlayerData: Codable {
    var name: String
    var level: Int
}
let playerData = OdinCustomData.encode(PlayerData(name: "Alice", level: 10))
try room.updatePeerUserData(userData: playerData)

// Reading user data (peer.userData is [UInt8])
let decoded: PlayerData? = OdinCustomData.decode(peer.userData)
```

### Sending Messages
```swift
// Broadcast to all peers in the room
let message = OdinCustomData.encode("Hello everyone!")
try room.sendMessage(data: message)

// Send to specific peers by ID
try room.sendMessage(data: message, targetIds: [peer1.id, peer2.id])
```

## Proximity Audio

For large multiplayer spaces with local voice clustering:

```swift
// Update position (2D coordinates)
try room.updatePosition(x: 100.0, y: 50.0)

// Set scale (affects culling radius)
room.setPositionScale(scale: 1.0)

// Culling: Peers within unit circle (1.0) can hear each other
```

## Audio Integration

OdinKit integrates with AVAudioEngine:

```swift
// Custom sample rate and channel configuration
try room.addMedia(audioConfig: OdinAudioStreamConfig(
    sample_rate: 48000,
    channel_count: 1
))
```

## SwiftUI Example

```swift
struct VoiceView: View {
    @StateObject var voiceManager = VoiceManager()

    var body: some View {
        VStack {
            ForEach(voiceManager.peers, id: \.id) { peer in
                HStack {
                    Text(peer.userId)
                    if peer.activeMedias.count > 0 {
                        Image(systemName: "mic.fill")
                    }
                }
            }

            Button("Join Room") {
                voiceManager.joinRoom()
            }
        }
    }
}
```

## Documentation

API reference: https://docs.4players.io/voice/swift/api/
SwiftUI sample: https://docs.4players.io/voice/swift/samples/swiftui-sample
