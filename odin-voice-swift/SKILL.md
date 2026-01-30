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

Swift wrapper for real-time VoIP chat on iOS and macOS.

## Requirements

- iOS 9.0+ / macOS 10.15+
- Xcode 10.2+
- Swift 5.0+

## Quick Start

```swift
// 1. Create access key (do this on backend in production)
let accessKey = try OdinAccessKey("YOUR_ACCESS_KEY")

// 2. Generate token
let token = try accessKey.generateToken(roomId: "MyRoom", userId: "user123")

// 3. Create room and set delegate
let room = OdinRoom()
room.delegate = self

// 4. Join room
try room.join(token: token)

// 5. Add microphone
let mediaId = try room.addMedia(type: .Audio)
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
room.userData           // Room user data
room.connectionStatus   // Current status
room.peers              // Connected peers
room.medias             // Audio streams
room.ownPeer            // Self reference

// Observable (Combine)
room.$connectionStatus  // Published status
room.$peers             // Published peers
room.$medias            // Published medias

// Methods
try room.join(token: token)
room.leave()
let mediaId = try room.addMedia(type: .Audio)
try room.removeMedia(streamHandle: mediaId)
try room.updateUserData(userData: bytes, target: .Peer)
try room.updatePosition(x: 100, y: 50)
room.setPositionScale(scale: 1.0)
try room.sendMessage(data: bytes, targetIds: [])
room.setAudioAutopilotMode(.Room)
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
        print("Connection: \(oldState) → \(newState)")
    }

    func onRoomJoined(room: OdinRoom, ownPeer: OdinPeer) {
        print("Joined as peer \(ownPeer.id)")
    }

    func onPeerJoined(room: OdinRoom, peer: OdinPeer) {
        print("Peer joined: \(peer.userId)")
    }

    func onPeerLeft(room: OdinRoom, peer: OdinPeer) {
        print("Peer left: \(peer.userId)")
    }

    func onMediaAdded(room: OdinRoom, peer: OdinPeer, media: OdinMedia) {
        print("Media added from \(peer.userId)")
    }

    func onMediaRemoved(room: OdinRoom, peer: OdinPeer, media: OdinMedia) {
        print("Media removed from \(peer.userId)")
    }

    func onMediaActiveStateChanged(room: OdinRoom, peer: OdinPeer, media: OdinMedia) {
        print("\(peer.userId) is \(media.activityStatus ? "speaking" : "silent")")
    }

    func onMessageReceived(room: OdinRoom, senderId: UInt64, data: [UInt8]) {
        let text = String(bytes: data, encoding: .utf8) ?? ""
        print("Message: \(text)")
    }

    func onRoomUserDataChanged(room: OdinRoom) {
        print("Room data updated")
    }

    func onPeerUserDataChanged(room: OdinRoom, peer: OdinPeer) {
        print("Peer \(peer.userId) data updated")
    }
}
```

### Option B: Combine Publishers

```swift
import Combine

var cancellables = Set<AnyCancellable>()

room.$connectionStatus
    .sink { status in
        print("Status: \(status)")
    }
    .store(in: &cancellables)

room.$peers
    .sink { peers in
        print("Peers: \(peers.count)")
    }
    .store(in: &cancellables)
```

## Audio Processing (APM)

```swift
var apmConfig = OdinApmConfig()

// Voice Activity Detection
apmConfig.voiceActivityDetection = true
apmConfig.vadAttackProbability = 0.9
apmConfig.vadReleaseProbability = 0.8

// Volume Gate
apmConfig.volumeGate = true
apmConfig.volumeGateAttackThreshold = -30  // dBFS
apmConfig.volumeGateReleaseThreshold = -40

// Audio Processing
apmConfig.echoCancellation = true
apmConfig.noiseSuppression = .Moderate  // or .Aggressive, .VeryAggressive
apmConfig.highPassFilter = true
apmConfig.preamplifier = false
apmConfig.transientSuppression = true
apmConfig.gainController = .V2

// Apply config
try room.updateAudioConfig(apmConfig)
```

## User Data & Messaging

### Setting User Data
```swift
// Using OdinCustomData helper
let userData = try OdinCustomData.encode(["name": "Alice", "level": 10])
try room.updateUserData(userData: userData, target: .Peer)

// Reading user data
if let data = peer.userData {
    let decoded = try OdinCustomData.decode(data) as [String: Any]
    let name = decoded["name"] as? String
}
```

### Sending Messages
```swift
// Broadcast to all
let message = Array("Hello everyone!".utf8)
try room.sendMessage(data: message, targetIds: [])

// Send to specific peers
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
// Access audio nodes for mixing
let roomAudioNode = room.audioNode
let mediaAudioNode = media.audioNode

// Custom sample rate
let config = OdinAudioStreamConfig(sampleRate: 48000)
let mediaId = try room.addMedia(audioConfig: config)
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
