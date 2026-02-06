# Framework Integration

Complete examples for React, Vue 3, and Angular based on the tested working examples in `ari/examples/`.

## Table of Contents

- [React](#react)
- [Vue 3 (Composition API)](#vue-3-composition-api)
- [Angular](#angular)

---

## React

```tsx
import { useState, useEffect, useRef, useCallback } from "react";
import { init, Room, DeviceManager, AudioInput } from "@4players/odin";
import { createPlugin } from "@4players/odin-plugin-web";
import { TokenGenerator } from "@4players/odin-tokens";

interface PeerInfo {
  id: number;
  userId: string;
  isLocal: boolean;
}

function App() {
  const [isConnected, setIsConnected] = useState(false);
  const [peers, setPeers] = useState<PeerInfo[]>([]);
  const [speakingPeers, setSpeakingPeers] = useState<Set<number>>(new Set());
  const [isMuted, setIsMuted] = useState(false);

  const roomRef = useRef<Room | null>(null);
  const audioInputRef = useRef<AudioInput | null>(null);
  const pluginRef = useRef<ReturnType<typeof createPlugin> | null>(null);

  // Cleanup on unmount
  useEffect(() => {
    return () => {
      audioInputRef.current?.close();
      roomRef.current?.leave();
    };
  }, []);

  // Initialize plugin (only once)
  const initPlugin = useCallback(() => {
    if (pluginRef.current) return;
    pluginRef.current = createPlugin(async (sampleRate) => {
      const audioContext = new AudioContext({ sampleRate });
      await audioContext.resume();
      return audioContext;
    });
    init(pluginRef.current);
  }, []);

  const joinRoom = async () => {
    initPlugin();

    // Generate token (in production, fetch from your backend instead)
    const generator = new TokenGenerator("YOUR_ACCESS_KEY");
    const token = await generator.createToken("default", "react-user");

    const room = new Room();
    roomRef.current = room;

    // Register event handlers BEFORE joining
    room.onJoined = () => setIsConnected(true);
    room.onLeft = () => {
      setIsConnected(false);
      setPeers([]);
    };

    room.onPeerJoined = (payload) => {
      const isLocal = payload.peer.id === room.ownPeerId;
      setPeers((prev) => [
        ...prev.filter((p) => p.id !== payload.peer.id),
        { id: payload.peer.id, userId: payload.peer.userId, isLocal },
      ]);

      // Listen for audio activity on this peer
      payload.peer.onAudioActivity = ({ media }) => {
        setSpeakingPeers((prev) => {
          const next = new Set(prev);
          if (media.isActive) {
            next.add(payload.peer.id);
          } else {
            next.delete(payload.peer.id);
          }
          return next;
        });
      };
    };

    room.onPeerLeft = (payload) => {
      setPeers((prev) => prev.filter((p) => p.id !== payload.peer.id));
      setSpeakingPeers((prev) => {
        const next = new Set(prev);
        next.delete(payload.peer.id);
        return next;
      });
    };

    // Set output device BEFORE joining (required to hear other peers)
    await pluginRef.current!.setOutputDevice({});
    await room.join(token, { gateway: "https://gateway.odin.4players.io" });

    // Create and add microphone input (defaults: system mic with echo cancellation, noise suppression, gain control)
    const audioInput = await DeviceManager.createAudioInput();
    audioInputRef.current = audioInput;
    await room.addAudioInput(audioInput);
  };

  const leaveRoom = () => {
    if (audioInputRef.current) {
      roomRef.current?.removeAudioInput(audioInputRef.current);
      audioInputRef.current.close();
      audioInputRef.current = null;
    }
    if (roomRef.current) {
      roomRef.current.leave();
      roomRef.current = null;
    }
    setIsMuted(false);
  };

  const toggleMute = async () => {
    if (!audioInputRef.current) return;

    if (isMuted) {
      // Unmute: restore volume and re-add to room to resume encoding
      await audioInputRef.current.setVolume(1);
      await roomRef.current?.addAudioInput(audioInputRef.current);
    } else {
      // Mute: remove from room to stop encoder, use 'muted' to stop MediaStream
      // (removes browser recording indicator)
      roomRef.current?.removeAudioInput(audioInputRef.current);
      await audioInputRef.current.setVolume("muted");
    }
    setIsMuted(!isMuted);
  };

  return (
    <div>
      {!isConnected ? (
        <button onClick={joinRoom}>Join Voice Chat</button>
      ) : (
        <div>
          <button onClick={toggleMute}>{isMuted ? "Unmute" : "Mute"}</button>
          <button onClick={leaveRoom}>Leave</button>

          <h3>Peers ({peers.length})</h3>
          <ul>
            {peers.map((peer) => (
              <li key={peer.id}>
                {peer.isLocal ? "You" : peer.userId}
                {speakingPeers.has(peer.id) && " (speaking)"}
              </li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
}

export default App;
```

---

## Vue 3 (Composition API)

```vue
<template>
  <div>
    <template v-if="!isConnected">
      <button @click="joinRoom" :disabled="isConnecting">
        {{ isConnecting ? "Connecting..." : "Join Voice Chat" }}
      </button>
    </template>

    <template v-else>
      <button @click="toggleMute">{{ isMuted ? "Unmute" : "Mute" }}</button>
      <button @click="leaveRoom">Leave</button>

      <h3>Peers ({{ peers.length }})</h3>
      <ul>
        <li v-for="peer in peers" :key="peer.id">
          {{ peer.isLocal ? "You" : peer.userId }}
          <span v-if="speakingPeers.has(peer.id)"> (speaking)</span>
        </li>
      </ul>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, shallowRef, onUnmounted } from "vue";
import {
  init,
  Room,
  DeviceManager,
  type AudioInput as AudioInputType,
} from "@4players/odin";
import { createPlugin } from "@4players/odin-plugin-web";
import { TokenGenerator } from "@4players/odin-tokens";

interface PeerInfo {
  id: number;
  userId: string;
  isLocal: boolean;
}

const isConnected = ref(false);
const isConnecting = ref(false);
const peers = ref<PeerInfo[]>([]);
const speakingPeers = ref<Set<number>>(new Set());
const isMuted = ref(false);

const room = shallowRef<Room | null>(null);
const audioInput = shallowRef<AudioInputType | null>(null);
let audioPlugin: ReturnType<typeof createPlugin>;

// Initialize plugin (only once)
const initPlugin = () => {
  if (audioPlugin) return;
  audioPlugin = createPlugin(async (sampleRate) => {
    const audioContext = new AudioContext({ sampleRate });
    await audioContext.resume();
    return audioContext;
  });
  init(audioPlugin);
};

const joinRoom = async () => {
  isConnecting.value = true;

  try {
    initPlugin();

    // Generate token (in production, fetch from your backend instead)
    const generator = new TokenGenerator("YOUR_ACCESS_KEY");
    const token = await generator.createToken("default", "vue-user");

    room.value = new Room();

    // Register event handlers BEFORE joining
    room.value.onJoined = () => {
      isConnected.value = true;
      isConnecting.value = false;
    };

    room.value.onLeft = () => {
      isConnected.value = false;
      isConnecting.value = false;
      peers.value = [];
    };

    room.value.onPeerJoined = (payload) => {
      const isLocal = payload.peer.id === room.value!.ownPeerId;
      peers.value = [
        ...peers.value.filter((p) => p.id !== payload.peer.id),
        { id: payload.peer.id, userId: payload.peer.userId, isLocal },
      ];

      // Listen for audio activity on this peer
      payload.peer.onAudioActivity = ({ media }) => {
        const next = new Set(speakingPeers.value);
        if (media.isActive) {
          next.add(payload.peer.id);
        } else {
          next.delete(payload.peer.id);
        }
        speakingPeers.value = next;
      };
    };

    room.value.onPeerLeft = (payload) => {
      peers.value = peers.value.filter((p) => p.id !== payload.peer.id);
      const next = new Set(speakingPeers.value);
      next.delete(payload.peer.id);
      speakingPeers.value = next;
    };

    // Set output device BEFORE joining (required to hear other peers)
    await audioPlugin.setOutputDevice({});
    await room.value.join(token, {
      gateway: "https://gateway.odin.4players.io",
    });

    // Create and add microphone input (defaults: system mic with echo cancellation, noise suppression, gain control)
    audioInput.value = await DeviceManager.createAudioInput();
    await room.value.addAudioInput(audioInput.value);
  } catch (e) {
    isConnecting.value = false;
    throw e;
  }
};

const leaveRoom = () => {
  if (audioInput.value) {
    room.value?.removeAudioInput(audioInput.value);
    audioInput.value.close();
    audioInput.value = null;
  }
  if (room.value) {
    room.value.leave();
    room.value = null;
  }
  isMuted.value = false;
};

const toggleMute = async () => {
  if (!audioInput.value) return;

  if (isMuted.value) {
    // Unmute: restore volume and re-add to room to resume encoding
    await audioInput.value.setVolume(1);
    await room.value?.addAudioInput(audioInput.value);
  } else {
    // Mute: remove from room to stop encoder, use 'muted' to stop MediaStream
    // (removes browser recording indicator)
    room.value?.removeAudioInput(audioInput.value);
    await audioInput.value.setVolume("muted");
  }
  isMuted.value = !isMuted.value;
};

// Cleanup on component unmount
onUnmounted(() => {
  leaveRoom();
});
</script>
```

---

## Angular

```typescript
import {
  Component,
  OnDestroy,
  signal,
  ChangeDetectionStrategy,
} from "@angular/core";
import { FormsModule } from "@angular/forms";
import { init, Room, DeviceManager, AudioInput } from "@4players/odin";
import { createPlugin } from "@4players/odin-plugin-web";
import { TokenGenerator } from "@4players/odin-tokens";

interface PeerInfo {
  id: number;
  userId: string;
  isLocal: boolean;
}

@Component({
  selector: "app-root",
  standalone: true,
  imports: [FormsModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    @if (!isConnected()) {
      <button (click)="joinRoom()" [disabled]="isConnecting()">
        {{ isConnecting() ? "Connecting..." : "Join Voice Chat" }}
      </button>
    } @else {
      <button (click)="toggleMute()">
        {{ isMuted() ? "Unmute" : "Mute" }}
      </button>
      <button (click)="leaveRoom()">Leave</button>

      <h3>Peers ({{ peers().length }})</h3>
      <ul>
        @for (peer of peers(); track peer.id) {
          <li>
            {{ peer.isLocal ? "You" : peer.userId }}
            @if (speakingPeers().has(peer.id)) {
              (speaking)
            }
          </li>
        }
      </ul>
    }
  `,
})
export class AppComponent implements OnDestroy {
  isConnected = signal(false);
  isConnecting = signal(false);
  peers = signal<PeerInfo[]>([]);
  speakingPeers = signal<Set<number>>(new Set());
  isMuted = signal(false);

  private room: Room | null = null;
  private audioInput: AudioInput | null = null;
  private audioPlugin!: ReturnType<typeof createPlugin>;

  ngOnDestroy() {
    this.leaveRoom();
  }

  // Initialize plugin (only once)
  private initPlugin() {
    if (this.audioPlugin) return;
    this.audioPlugin = createPlugin(async (sampleRate) => {
      const audioContext = new AudioContext({ sampleRate });
      await audioContext.resume();
      return audioContext;
    });
    init(this.audioPlugin);
  }

  async joinRoom() {
    this.isConnecting.set(true);

    try {
      this.initPlugin();

      // Generate token (in production, fetch from your backend instead)
      const generator = new TokenGenerator("YOUR_ACCESS_KEY");
      const token = await generator.createToken("default", "angular-user");

      this.room = new Room();

      // Register event handlers BEFORE joining
      this.room.onJoined = () => {
        this.isConnected.set(true);
        this.isConnecting.set(false);
      };

      this.room.onLeft = () => {
        this.isConnected.set(false);
        this.isConnecting.set(false);
        this.peers.set([]);
      };

      this.room.onPeerJoined = (payload) => {
        const isLocal = payload.peer.id === this.room!.ownPeerId;
        this.peers.update((prev) => [
          ...prev.filter((p) => p.id !== payload.peer.id),
          { id: payload.peer.id, userId: payload.peer.userId, isLocal },
        ]);

        // Listen for audio activity on this peer
        payload.peer.onAudioActivity = ({ media }) => {
          this.speakingPeers.update((prev) => {
            const next = new Set(prev);
            if (media.isActive) {
              next.add(payload.peer.id);
            } else {
              next.delete(payload.peer.id);
            }
            return next;
          });
        };
      };

      this.room.onPeerLeft = (payload) => {
        this.peers.update((prev) =>
          prev.filter((p) => p.id !== payload.peer.id),
        );
        this.speakingPeers.update((prev) => {
          const next = new Set(prev);
          next.delete(payload.peer.id);
          return next;
        });
      };

      // Set output device BEFORE joining (required to hear other peers)
      await this.audioPlugin.setOutputDevice({});
      await this.room.join(token, {
        gateway: "https://gateway.odin.4players.io",
      });

      // Create and add microphone input (defaults: system mic with echo cancellation, noise suppression, gain control)
      this.audioInput = await DeviceManager.createAudioInput();
      await this.room.addAudioInput(this.audioInput);
    } catch (e) {
      this.isConnecting.set(false);
      throw e;
    }
  }

  leaveRoom() {
    if (this.audioInput) {
      this.room?.removeAudioInput(this.audioInput);
      this.audioInput.close();
      this.audioInput = null;
    }
    if (this.room) {
      this.room.leave();
      this.room = null;
    }
    this.isMuted.set(false);
  }

  async toggleMute() {
    if (!this.audioInput) return;

    if (this.isMuted()) {
      // Unmute: restore volume and re-add to room to resume encoding
      await this.audioInput.setVolume(1);
      await this.room?.addAudioInput(this.audioInput);
    } else {
      // Mute: remove from room to stop encoder, use 'muted' to stop MediaStream
      // (removes browser recording indicator)
      this.room?.removeAudioInput(this.audioInput);
      await this.audioInput.setVolume("muted");
    }
    this.isMuted.update((v) => !v);
  }
}
```
