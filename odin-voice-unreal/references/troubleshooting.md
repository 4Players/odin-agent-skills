# Troubleshooting

Common issues and solutions for the ODIN Voice Unreal Plugin.

## Build and Packaging Issues

### Duplicate Plugin Installation

- **Never** install ODIN in both Engine and Project directories
- Keep plugin in one location only
- Clean these directories after removal: `Binaries`, `Build`, `Intermediate`, `DerivedDataCache`

### Blueprint-Only Project Crashes

Blueprint-only projects may encounter packaging errors. Solutions:

1. Install plugin in Engine (Fab Marketplace) directory instead of Project, OR
2. Convert to C++ project by adding a dummy C++ class

### Engine Version Upgrades

> **CRITICAL**: Install ODIN plugin in new engine version BEFORE opening your project.

Compiling blueprints without the plugin present causes Blueprint corruption that requires manual repair.

## Spatial Audio Issues

### No 3D Positioning

1. Verify Synth Component is attached to the correct character
2. Check that attenuation settings have `bSpatialize = true`
3. For non-Unreal clients: Spawn placeholder actors on peer join to attach synth components

### Sound Occlusion Not Working

- Works automatically when synth component is properly configured with attenuation
- Verify attenuation settings are applied to the synth component

## Audio Issues

### No Microphone Input

1. Check platform permissions (Android, iOS)
2. Verify Audio Capture Plugin is enabled
3. Confirm correct audio capture device is selected

### Echo or Feedback

Enable audio processing features:
- Voice Activity Detection (VAD)
- Echo Cancellation
- High Pass Filter

See [v1.md](../versions/v1.md#apm-settings-reference) or [v2.md](../versions/v2.md#audio-pipeline-configuration) for APM configuration details.

## Platform-Specific Issues

### Android/Meta Quest

**Permissions**: Configure microphone permissions in Project Settings > Android > Permissions.

**v1.x JoinRoom Callback Issue**: Using standard `FVector2D()` constructor prevents callbacks from triggering.

```cpp
// WRONG - callbacks won't trigger
Room->JoinRoom(URL, Token, UserData, FVector2D());

// CORRECT - explicitly initialize
Room->JoinRoom(URL, Token, UserData, FVector2D(0, 0));
```

### iOS/Apple

Add microphone usage description to `Info.plist`:

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Voice chat requires microphone access</string>
```

## Testing

Use the ODIN Web Client at https://4players.app for cross-platform testing:
- Same access key as your game
- Same gateway URL and room name
- Test without multiple game instances

> **Note**: This works with v1.x only! The web client currently is not compatible with v2.x Unreal clients.
