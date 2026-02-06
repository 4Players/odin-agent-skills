# Audio Processing Patterns

Implementation patterns for recording bots, AI assistants, and audio processing.

## Table of Contents

- [Recording Bot](#recording-bot)
- [AI Voice Assistant](#ai-voice-assistant)
- [Audio Processing Pipeline](#audio-processing-pipeline)
- [Multi-Room Bot](#multi-room-bot)
- [Common Patterns](#common-patterns)

---

## Recording Bot

```javascript
const fs = require('fs');
const WavEncoder = require('wav').Writer;

room.onJoined((event) => {
    console.log(`Recording bot joined room ${event.roomId}`);
});

// Record all peers to separate WAV files
const recordings = new Map();

room.onMediaStarted((event) => {
    const filename = `peer_${event.peerId}_${Date.now()}.wav`;
    const writer = new WavEncoder({
        sampleRate: 48000,
        channels: 1,
        bitDepth: 16
    });
    writer.pipe(fs.createWriteStream(filename));
    recordings.set(event.media, writer);
    console.log(`Recording ${filename}`);
});

room.onAudioDataReceived((data) => {
    const writer = recordings.get(data.mediaId);
    if (writer) {
        // Convert Float32Array to Int16Array
        const buffer = Buffer.alloc(data.samples16.length * 2);
        for (let i = 0; i < data.samples16.length; i++) {
            buffer.writeInt16LE(data.samples16[i], i * 2);
        }
        writer.write(buffer);
    }
});

room.onMediaStopped((event) => {
    const writer = recordings.get(event.mediaId);
    if (writer) {
        writer.end();
        recordings.delete(event.mediaId);
        console.log(`Recording finished for media ${event.mediaId}`);
    }
});
```

---

## AI Voice Assistant

```javascript
const { OdinClient } = require('@4players/odin-nodejs');

// AI assistant that responds to voice
room.onAudioDataReceived(async (data) => {
    // 1. Transcribe audio
    const text = await speechToText(data.samples32);

    // 2. Generate AI response
    const response = await aiModel.generate(text);

    // 3. Synthesize and play response
    const audioResponse = await textToSpeech(response);
    const media = room.createAudioStream(48000, 1);
    media.sendAudioData(audioResponse);
    media.close();
});
```

---

## Audio Processing Pipeline

```javascript
// Apply real-time audio effects
room.onAudioDataReceived((data) => {
    // Access 32-bit float samples for processing
    let processed = data.samples32;

    // Apply effects
    processed = applyCompressor(processed);
    processed = applyEqualizer(processed);
    processed = applyReverb(processed);

    // Forward to output/storage
    outputDevice.write(processed);
});
```

---

## Multi-Room Bot

```javascript
// Bot that bridges multiple rooms
const client = new OdinClient();

const room1 = client.createRoom(token1);
const room2 = client.createRoom(token2);

await room1.join("https://gateway.odin.4players.io");
await room2.join("https://gateway.odin.4players.io");

// Forward audio from room1 to room2
room1.onAudioDataReceived((data) => {
    const media = room2.createAudioStream(48000, 1);
    media.sendAudioData(data.samples32);
    media.close();
});

// Forward audio from room2 to room1
room2.onAudioDataReceived((data) => {
    const media = room1.createAudioStream(48000, 1);
    media.sendAudioData(data.samples32);
    media.close();
});
```

---

## Common Patterns

### Audio Timing (20ms Chunks)

ODIN requires audio in 20ms intervals (50 times per second):

```javascript
const sampleRate = 48000;
const channels = 1;
const chunkSize = sampleRate / 50; // 960 samples = 20ms at 48kHz

const media = room.createAudioStream(sampleRate, channels);

// Stream audio in correct timing
setInterval(() => {
    const chunk = getNextAudioChunk(chunkSize); // Float32Array
    media.sendAudioData(chunk);
}, 20); // Every 20ms
```

### Graceful Shutdown

```javascript
process.on('SIGINT', async () => {
    console.log('Shutting down...');

    // Close all active media streams
    activeMediaStreams.forEach(media => media.close());

    // Disconnect from room
    room.close();

    // Wait for clean disconnect
    await new Promise(resolve => {
        room.onLeft(() => resolve());
        setTimeout(resolve, 1000); // Timeout after 1s
    });

    process.exit(0);
});
```

### Resource Management

```javascript
// Track active resources
const activeMedia = new Set();

room.onJoined((event) => {
    event.mediaIds.forEach(id => activeMedia.add(id));
});

room.onMediaStarted((event) => {
    activeMedia.add(event.media);
});

room.onMediaStopped((event) => {
    activeMedia.delete(event.mediaId);
});

room.onLeft(() => {
    // Cleanup all resources
    activeMedia.clear();
});
```
