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
const selectedDevice = devices.find(d => d.deviceId === deviceId);

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

```javascript
setInterval(() => {
    const stats = room.connectionStats;

    if (stats.rtt > 200) {
        console.warn('High latency detected:', stats.rtt, 'ms');
    }

    if (stats.packetLoss > 5) {
        console.warn('High packet loss:', stats.packetLoss, '%');
    }
}, 5000);
```

---

## Push-to-Talk Implementation

```javascript
let audioInput;
let isPushToTalkActive = false;

// Create muted input
audioInput = await ODIN.DeviceManager.createAudioInput(null, {
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
});
audioInput.setVolume(0); // Start muted
await room.addAudioInput(audioInput);

// Key down: activate
document.addEventListener('keydown', (e) => {
    if (e.code === 'Space' && !isPushToTalkActive) {
        isPushToTalkActive = true;
        audioInput.setVolume(1); // Unmute
    }
});

// Key up: deactivate
document.addEventListener('keyup', (e) => {
    if (e.code === 'Space' && isPushToTalkActive) {
        isPushToTalkActive = false;
        audioInput.setVolume(0); // Mute
    }
});
```

---

## Room Reconnection

```javascript
let reconnectAttempts = 0;
const MAX_RECONNECT_ATTEMPTS = 5;

room.onLeft = async (payload) => {
    console.log('Disconnected from room');

    if (reconnectAttempts < MAX_RECONNECT_ATTEMPTS) {
        reconnectAttempts++;
        const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000);

        console.log(`Reconnecting in ${delay}ms (attempt ${reconnectAttempts})`);

        await new Promise(resolve => setTimeout(resolve, delay));

        try {
            await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
            reconnectAttempts = 0; // Reset on successful reconnection
        } catch (error) {
            console.error('Reconnection failed:', error);
        }
    }
};
```
