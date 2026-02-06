# Advanced Patterns

Implementation patterns for common advanced use cases.

## Table of Contents

- [Device Selection](#device-selection)
- [Connection Quality Monitoring](#connection-quality-monitoring)
- [Push-to-Talk Implementation](#push-to-talk-implementation)
- [Room Reconnection](#room-reconnection)

---

## Device Selection

```javascript
// List available devices
const devices = await ODIN.DeviceManager.listAudioInputs();

// Show device picker
const deviceId = await showDevicePicker(devices);
const selectedDevice = devices.find((d) => d.deviceId === deviceId);

// Create input with specific device
const audioInput = await ODIN.DeviceManager.createAudioInput(selectedDevice, {
  echoCanceller: true,
  noiseSuppression: true,
  gainController: true,
});

// Switch device later
await audioInput.setDevice(newDevice);
```

---

## Connection Quality Monitoring

You can access connection statistics via the `room.connectionStats` property at any time. Additionally, the room emits a `ConnectionStats` event at a regular interval (default: 1000ms) providing updated stats including computed `bytesReceivedLastSecond` and `bytesSentLastSecond` values.

### Using the ConnectionStats Event (Recommended)

```javascript
// Pattern 1: Property assignment
room.onConnectionStats = (stats) => {
    if (stats.rtt > 200) {
        console.warn('High latency detected:', stats.rtt, 'ms');
    }
    console.log('Bytes/s received:', stats.bytesReceivedLastSecond);
    console.log('Bytes/s sent:', stats.bytesSentLastSecond);
};

// Pattern 2: addEventListener (multiple handlers)
room.addEventListener('ConnectionStats', (event) => {
    const stats = event.payload;
    console.log('RTT:', stats.rtt, 'ms');
});
```

### Polling connectionStats Manually

```javascript
setInterval(() => {
  const stats = room.connectionStats;

  if (stats.rtt > 200) {
    console.warn("High latency detected:", stats.rtt, "ms");
  }

  if (stats.packetLoss > 5) {
    console.warn("High packet loss:", stats.packetLoss, "%");
  }
}, 5000);
```

---

## Push-to-Talk Implementation

The recommended way to implement Push to Talk (PTT) is to use the `volumeGate` setting on `InputSettings`. Setting `volumeGate` to `-90` (dBFS) keeps the gate closed by default, effectively silencing the input. When the PTT button is pressed, set `volumeGate` to `false` to disable gating and allow transmission. On release, set it back to `-90` to close the gate.

### VAD Defaults Reference

The SDK provides `VAD_DEFAULTS` with two components:

```javascript
{
  voiceActivity: {
    attackThreshold: 0.9,   // 90% speech probability to start transmission
    releaseThreshold: 0.8,  // 80% speech probability to stop transmission
  },
  volumeGate: {
    attackThreshold: -30,   // dBFS level to open the gate
    releaseThreshold: -40,  // dBFS level to close the gate
  },
}
```

### Basic Setup

```javascript
// Create AudioInput with volumeGate set to -90 dBFS (gate closed by default)
const audioInput = await ODIN.DeviceManager.createAudioInput(
  {},
  { volumeGate: -90 },
);

room.addAudioInput(audioInput);
```

### voiceActivity Options

By default, `voiceActivity` remains enabled. This is recommended because it filters out background noise even while the PTT button is held down. The trade-off is that the `onAudioActivity` indicator won't be active when pressing PTT if there's no detected speech.

If you want the audio activity indicator to always reflect the PTT button state (active when pressed, regardless of speech detection), disable `voiceActivity`:

```javascript
await audioInput.setInputSettings({
  voiceActivity: false,
});
```

This may result in transmitting background noise while the PTT button is held.

### PTT Button Handlers

```javascript
const pttButton = document.getElementById("ptt-button");

// Press: open the gate
pttButton.addEventListener("pointerdown", async () => {
  await audioInput.setInputSettings({ volumeGate: false });
});

// Release: close the gate
pttButton.addEventListener("pointerup", async () => {
  await audioInput.setInputSettings({ volumeGate: -90 });
});

// Handle pointer leaving the button while pressed
pttButton.addEventListener("pointerleave", async () => {
  await audioInput.setInputSettings({ volumeGate: -90 });
});
```

> **Important**: Always handle the `pointerleave` event to ensure the gate closes if the user moves their pointer away from the button while holding it down.

### Complete Example

```javascript
let audioInput;

async function setupPTT(room) {
  // Create AudioInput with gate closed by default
  audioInput = await ODIN.DeviceManager.createAudioInput(
    {},
    { volumeGate: -90 },
  );

  room.addAudioInput(audioInput);

  // Setup PTT button handlers
  const pttButton = document.getElementById("ptt-button");

  pttButton.addEventListener("pointerdown", async () => {
    await audioInput.setInputSettings({ volumeGate: false });
  });

  pttButton.addEventListener("pointerup", async () => {
    await audioInput.setInputSettings({ volumeGate: -90 });
  });

  pttButton.addEventListener("pointerleave", async () => {
    await audioInput.setInputSettings({ volumeGate: -90 });
  });
}
```
