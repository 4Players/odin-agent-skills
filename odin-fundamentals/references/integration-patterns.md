# Common Integration Patterns

Implementation patterns for multi-room apps, push-to-talk, reconnection, and data sync.

## Table of Contents

- [Multi-Room Applications](#multi-room-applications)
- [Push-to-Talk](#push-to-talk)
- [Reconnection Logic](#reconnection-logic)
- [User Data Synchronization](#user-data-synchronization)
- [FAQ](#faq)

---

## Multi-Room Applications

```javascript
const connectionPool = new ODIN.ConnectionPool();
const room1 = connectionPool.createRoom();
const room2 = connectionPool.createRoom();

await room1.join(token1, { gateway: 'https://gateway.odin.4players.io' });
await room2.join(token2, { gateway: 'https://gateway.odin.4players.io' });
```

---

## Push-to-Talk

```javascript
let audioInput = await ODIN.DeviceManager.createAudioInput(null, { /* APM settings */ });
audioInput.setVolume(0); // Start muted
await room.addAudioInput(audioInput);

// On key press
document.addEventListener('keydown', (e) => {
    if (e.code === 'Space') audioInput.setVolume(1); // Unmute
});

// On key release
document.addEventListener('keyup', (e) => {
    if (e.code === 'Space') audioInput.setVolume(0); // Mute
});
```

---

## Reconnection Logic

```javascript
let reconnectAttempts = 0;
const MAX_ATTEMPTS = 5;

room.onLeft = async () => {
    if (reconnectAttempts < MAX_ATTEMPTS) {
        reconnectAttempts++;
        const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000);
        await new Promise(resolve => setTimeout(resolve, delay));

        try {
            await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
            reconnectAttempts = 0; // Reset on success
        } catch (error) {
            console.error('Reconnection failed:', error);
        }
    }
};
```

---

## User Data Synchronization

```javascript
// Set user data
const userData = ODIN.valueToUint8Array({
    nickname: 'Player1',
    level: 42,
    status: 'online'
});
room.userData = userData;
room.flushUserData();

// Receive peer data updates
room.onUserDataChanged = (payload) => {
    const data = ODIN.uint8ArrayToValue(payload.peer.data);
    console.log(`${payload.peer.userId} updated:`, data);
};
```

---

## FAQ

**Q: What's the difference between Access Keys and Access Tokens?**
A: Access Keys are server-side credentials used to generate Access Tokens. Access Tokens are client-side JWT tokens that allow users to join specific rooms. Always keep Access Keys secret on your backend.

**Q: Can I use ODIN for free?**
A: Yes, the Trial tier supports up to 25 concurrent users forever at no cost. Perfect for development and small projects.

**Q: Does ODIN work across different platforms?**
A: Yes, ODIN is fully cross-platform. A web user can talk to a Unity user in the same room seamlessly.

**Q: How do I scale from Trial to Production?**
A: Upgrade to Indie Starter (€49/month, 500 users) or Pay-as-you-Go (scales infinitely) based on your needs and budget.

**Q: Where is my data hosted?**
A: 4Players hosts ODIN infrastructure globally. You can choose EU or US gateways. Self-hosted on-premise deployment is also available.

**Q: Do I need to manage servers?**
A: No, 4Players handles all server infrastructure, maintenance, and scaling automatically.

**Q: What audio codecs does ODIN use?**
A: ODIN uses Opus codec for high-quality, low-latency audio transmission.

**Q: Can I send custom data alongside voice?**
A: Yes, ODIN supports binary data channels for game state, chat messages, and custom events.

**Q: Is there a bandwidth limit?**
A: No hidden traffic fees. Voice pricing is based on concurrent users, not bandwidth consumption.

**Q: How do I implement Push-to-Talk?**
A: Set audio input volume to 0 (muted), then set to 1 on key press. See pattern above.
