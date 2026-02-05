# Troubleshooting

Common issues and solutions for the ODIN Voice Unity SDK.

## Version Detection

Before troubleshooting, identify your SDK version:

1. **Check Package Manager**: Window > Package Manager > Find "4Players ODIN"
   - Version number: 1.x.x or 2.x.x

or:

2. **Check for Key Components**:
   - **v1.x**: Has `OdinHandler` class (singleton pattern)
   - **v2.x**: Has `OdinRoom` class and `OdinInstance` prefab

## Installation Issues

### Remove Old Versions Before Upgrading

> **CRITICAL**: Unity does not remove old files automatically. Remove the entire ODIN package before installing a new version.

Clean these directories after removal:
- `Assets/4Players/ODIN` (if using Unity Package)
- Package from Package Manager

### Package Manager Issues

If Package Manager fails to import:
1. Close Unity
2. Delete `Library/PackageCache` folder
3. Reopen Unity and try again

## Audio Issues

### No Microphone Input

**All Versions:**
1. Check platform permissions (Android, iOS)
2. Verify microphone is not in use by another application
3. Check MicrophoneReader/OdinMicrophoneReader component is enabled

**v1.x Specific:**
- Ensure MicrophoneReader is attached to OdinHandler GameObject or uses `DontDestroyOnLoad`

**v2.x Specific:**
- Check `AutostartListen` is enabled
- Verify `RedirectCapturedAudio` is true

### Echo or Feedback

Enable audio processing features:
- Voice Activity Detection (VAD)
- Echo Cancellation
- High Pass Filter

See version-specific documentation:
- [v1.md](../versions/v1.md) for room-level APM
- [v2.md](../versions/v2.md) for audio pipeline effects

### No Audio from Remote Peers

**v1.x:**
- Verify `OnCreatedMediaObject` handler is calling `AddPlaybackComponent()`
- Check PlaybackComponent is attached and enabled

**v2.x:**
- Ensure OdinRoom is creating OdinPeer/OdinMedia GameObjects
- Check OdinMedia AudioSource is not muted

## Spatial Audio Issues

### 3D Positioning Not Working

1. Verify AudioSource `spatialBlend = 1.0f`
2. Check PlaybackComponent/OdinMedia is attached to the correct player GameObject
3. Ensure AudioListener is on the local player

### Sound Not Attenuating with Distance

```csharp
// v1.x
playback.PlaybackSource.rolloffMode = AudioRolloffMode.Linear;
playback.PlaybackSource.maxDistance = 10f;
playback.PlaybackSource.minDistance = 1f;

// v2.x
media.RolloffMode = AudioRolloffMode.Linear;
media.MaxDistance = 10f;
media.MinDistance = 1f;
```

## Threading Issues (v2.x)

### Unity API Errors in Event Handlers

> **v2.x Low-Level API**: Events fire on background threads. Never call Unity API directly.

```csharp
// WRONG - will crash
void OnMediaStarted(...) {
    GameObject obj = new GameObject();  // Unity API call on wrong thread!
}

// CORRECT - dispatch to main thread
ConcurrentQueue<Action> UnityQueue = new ConcurrentQueue<Action>();

void OnMediaStarted(...) {
    UnityQueue.Enqueue(() => {
        GameObject obj = new GameObject();  // Safe on main thread
    });
}

void Update() {
    while (UnityQueue.TryDequeue(out var action))
        action?.Invoke();
}
```

## Scene Change Issues

### v1.x: Audio Lost After Scene Change

OdinHandler uses `DontDestroyOnLoad` by default. Issues occur when:
- MicrophoneReader is on a separate GameObject without `DontDestroyOnLoad`
- PlaybackComponents are destroyed with scene

**Solution**: Keep OdinHandler in a persistent scene or re-create PlaybackComponents after scene loads.

## Platform-Specific Issues

### Android

**Permissions**: Configure microphone permissions in Project Settings > Player > Android > Permissions.

### iOS/Apple

Add microphone usage description to `Info.plist`:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Voice chat requires microphone access</string>
```

### Apple Silicon

ODIN supports Apple Silicon. See [FAQ](https://docs.4players.io/voice/unity/faq/#does-4players-odin-support-apple-silicon) for setup requirements.

## Testing

Use the ODIN Web Client at https://4players.app for cross-platform testing:
- Same access key as your game
- Same gateway URL and room name
- Test voice chat without multiple game instances

> **Note**: The web client is currently only compatible with SDK v1.x. It does not work with v2.x clients.

## Getting Help

If issues persist:
- **Discord**: [`#odin-unity` channel](https://4np.de/discord)
- **GitHub Issues**: https://github.com/4Players/odin-sdk-unity/issues
- **Support**: https://docs.4players.io/docs/support/
