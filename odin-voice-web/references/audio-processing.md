# Audio Processing Module (APM) Settings

Configure advanced audio processing for optimal voice quality.

## Complete Configuration

```javascript
const audioInput = await ODIN.DeviceManager.createAudioInput(null, {
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

## Recommended Presets

### Gaming (Low Latency)

```javascript
{
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.85,
        releaseThreshold: 0.75,
    },
    volumeGate: {
        attackThreshold: -35,
        releaseThreshold: -45,
    }
}
```

### Podcasting (High Quality)

```javascript
{
    echoCanceller: false,       // Use hardware echo cancellation
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.95,
        releaseThreshold: 0.90,
    },
    volumeGate: {
        attackThreshold: -25,
        releaseThreshold: -35,
    }
}
```

### Noisy Environment

```javascript
{
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
    voiceActivity: {
        attackThreshold: 0.95,  // Higher threshold = less false positives
        releaseThreshold: 0.90,
    },
    volumeGate: {
        attackThreshold: -20,   // Higher gate = blocks more noise
        releaseThreshold: -30,
    }
}
```

## Understanding dBFS Values

- **0 dBFS**: Maximum possible level (digital clipping)
- **-6 dBFS**: Loud speaking
- **-12 dBFS**: Normal speaking
- **-20 dBFS**: Quiet speaking
- **-30 dBFS**: Whisper
- **-40 dBFS**: Very quiet background noise
- **-50+ dBFS**: Silence/noise floor
