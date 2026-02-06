# Audio Processing Module (APM) Settings

Configure advanced audio processing for optimal voice quality.

## Complete Configuration

All settings are optional. Only specify the ones you want to override; unset properties will use their defaults.

```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput({}, {
    // Volume control
    volume: 1,                  // 0-2 (0=mute, 1=normal, 2=boost)

    // Acoustic Echo Cancellation (AEC)
    echoCanceller: true,        // Remove speaker echo from microphone

    // Noise Suppression
    noiseSuppression: true,     // Remove background noise

    // Automatic Gain Control (AGC)
    gainController: true,       // Normalize volume levels

    // Voice Activity Detection (VAD)
    voiceActivity: {
        attackThreshold: 0.9,   // 0-1: Probability to start speaking (0.9 = 90%)
        releaseThreshold: 0.8,  // 0-1: Probability to stop speaking (0.8 = 80%)
    },

    // Volume Gate (noise gate)
    volumeGate: {
        attackThreshold: -30,   // dBFS: Level to open gate (start transmitting)
        releaseThreshold: -40,  // dBFS: Level to close gate (stop transmitting)
    }
});
```

## Update Settings Dynamically

```javascript
// Update individual settings
await audioInput.setInputSettings({
    noiseSuppression: false,
    volumeGate: { attackThreshold: -25 }
});

// Toggle echo cancellation
await audioInput.setInputSettings({
    echoCanceller: !currentSettings.echoCanceller
});
```
