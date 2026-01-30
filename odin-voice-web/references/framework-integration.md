# Framework Integration

Complete examples for React, Vue 3, and Angular.

## Table of Contents

- [React](#react)
- [Vue 3 (Composition API)](#vue-3-composition-api)
- [Angular](#angular)

---

## React

```typescript
import React, { useState, useEffect, useRef } from 'react';
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

function VoiceChat() {
    const [isConnected, setIsConnected] = useState(false);
    const [peers, setPeers] = useState<Map<number, any>>(new Map());
    const [isMuted, setIsMuted] = useState(false);
    const roomRef = useRef<ODIN.Room | null>(null);
    const audioInputRef = useRef<any>(null);
    const pluginRef = useRef<any>(null);

    // Initialize plugin once
    useEffect(() => {
        const initPlugin = async () => {
            if (pluginRef.current) return;

            pluginRef.current = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
                const audioContext = new AudioContext({ sampleRate });
                await audioContext.resume();
                return audioContext;
            });

            ODIN.init(pluginRef.current);
        };

        initPlugin();
    }, []);

    // Cleanup on unmount
    useEffect(() => {
        return () => {
            if (roomRef.current) {
                roomRef.current.leave();
            }
            if (audioInputRef.current) {
                audioInputRef.current.close();
            }
        };
    }, []);

    const joinRoom = async (token: string) => {
        // Create room
        const room = new ODIN.Room();
        roomRef.current = room;

        // Setup events
        room.onJoined = () => setIsConnected(true);
        room.onLeft = () => setIsConnected(false);

        room.onPeerJoined = (payload) => {
            setPeers(prev => new Map(prev).set(payload.peer.id, payload.peer));
        };

        room.onPeerLeft = (payload) => {
            setPeers(prev => {
                const next = new Map(prev);
                next.delete(payload.peer.id);
                return next;
            });
        };

        // Join room
        await room.join(token, { gateway: 'https://gateway.odin.4players.io' });
        await pluginRef.current.setOutputDevice({});

        // Add microphone
        const audioInput = await ODIN.DeviceManager.createAudioInput(null, {
            echoCanceller: true,
            noiseSuppression: true,
            gainController: true,
        });
        audioInputRef.current = audioInput;
        await room.addAudioInput(audioInput);
    };

    const leaveRoom = async () => {
        if (audioInputRef.current) {
            await roomRef.current?.removeAudioInput(audioInputRef.current);
            audioInputRef.current.close();
            audioInputRef.current = null;
        }

        if (roomRef.current) {
            roomRef.current.leave();
            roomRef.current = null;
        }
    };

    const toggleMute = async () => {
        if (audioInputRef.current) {
            const newVolume = isMuted ? 1 : 0;
            audioInputRef.current.setVolume(newVolume);
            setIsMuted(!isMuted);
        }
    };

    return (
        <div>
            {!isConnected ? (
                <button onClick={() => joinRoom(/* fetch token */)}>
                    Join Voice Chat
                </button>
            ) : (
                <>
                    <button onClick={leaveRoom}>Leave</button>
                    <button onClick={toggleMute}>
                        {isMuted ? 'Unmute' : 'Mute'}
                    </button>
                    <div>
                        <h3>Peers in room: {peers.size}</h3>
                        {Array.from(peers.values()).map(peer => (
                            <div key={peer.id}>{peer.userId}</div>
                        ))}
                    </div>
                </>
            )}
        </div>
    );
}

export default VoiceChat;
```

---

## Vue 3 (Composition API)

```typescript
<template>
  <div>
    <button v-if="!isConnected" @click="joinRoom">Join Voice Chat</button>
    <template v-else>
      <button @click="leaveRoom">Leave</button>
      <button @click="toggleMute">{{ isMuted ? 'Unmute' : 'Mute' }}</button>
      <div>
        <h3>Peers: {{ peers.size }}</h3>
        <div v-for="peer in Array.from(peers.values())" :key="peer.id">
          {{ peer.userId }}
        </div>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, onUnmounted } from 'vue';
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

const isConnected = ref(false);
const peers = ref(new Map());
const isMuted = ref(false);
const room = ref<ODIN.Room | null>(null);
const audioInput = ref<any>(null);
let audioPlugin: any;

// Initialize plugin
const initPlugin = async () => {
  if (audioPlugin) return audioPlugin;

  audioPlugin = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
    const audioContext = new AudioContext({ sampleRate });
    await audioContext.resume();
    return audioContext;
  });

  ODIN.init(audioPlugin);
  return audioPlugin;
};

const joinRoom = async () => {
  await initPlugin();

  // Fetch token from backend
  const response = await fetch('/api/odin-token');
  const { token } = await response.json();

  room.value = new ODIN.Room();

  room.value.onJoined = () => { isConnected.value = true; };
  room.value.onLeft = () => { isConnected.value = false; };
  room.value.onPeerJoined = (payload) => {
    peers.value.set(payload.peer.id, payload.peer);
  };
  room.value.onPeerLeft = (payload) => {
    peers.value.delete(payload.peer.id);
  };

  await room.value.join(token, { gateway: 'https://gateway.odin.4players.io' });
  await audioPlugin.setOutputDevice({});

  audioInput.value = await ODIN.DeviceManager.createAudioInput(null, {
    echoCanceller: true,
    noiseSuppression: true,
    gainController: true,
  });
  await room.value.addAudioInput(audioInput.value);
};

const leaveRoom = async () => {
  if (audioInput.value) {
    await room.value?.removeAudioInput(audioInput.value);
    audioInput.value.close();
    audioInput.value = null;
  }

  if (room.value) {
    room.value.leave();
    room.value = null;
  }
};

const toggleMute = () => {
  if (audioInput.value) {
    const newVolume = isMuted.value ? 1 : 0;
    audioInput.value.setVolume(newVolume);
    isMuted.value = !isMuted.value;
  }
};

onUnmounted(() => {
  leaveRoom();
});
</script>
```

---

## Angular

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import * as ODIN from '@4players/odin';
import * as ODIN_PLUGIN from '@4players/odin-plugin-web';

@Component({
  selector: 'app-voice-chat',
  template: `
    <div>
      <button *ngIf="!isConnected" (click)="joinRoom()">Join Voice Chat</button>
      <ng-container *ngIf="isConnected">
        <button (click)="leaveRoom()">Leave</button>
        <button (click)="toggleMute()">{{ isMuted ? 'Unmute' : 'Mute' }}</button>
        <div>
          <h3>Peers: {{ peers.size }}</h3>
          <div *ngFor="let peer of getPeersArray()">
            {{ peer.userId }}
          </div>
        </div>
      </ng-container>
    </div>
  `
})
export class VoiceChatComponent implements OnInit, OnDestroy {
  isConnected = false;
  peers = new Map<number, any>();
  isMuted = false;

  private room: ODIN.Room | null = null;
  private audioInput: any = null;
  private audioPlugin: any;

  async ngOnInit() {
    await this.initPlugin();
  }

  ngOnDestroy() {
    this.leaveRoom();
  }

  private async initPlugin() {
    if (this.audioPlugin) return;

    this.audioPlugin = ODIN_PLUGIN.createPlugin(async (sampleRate) => {
      const audioContext = new AudioContext({ sampleRate });
      await audioContext.resume();
      return audioContext;
    });

    ODIN.init(this.audioPlugin);
  }

  async joinRoom() {
    // Fetch token from backend
    const response = await fetch('/api/odin-token');
    const { token } = await response.json();

    this.room = new ODIN.Room();

    this.room.onJoined = () => { this.isConnected = true; };
    this.room.onLeft = () => { this.isConnected = false; };
    this.room.onPeerJoined = (payload) => {
      this.peers.set(payload.peer.id, payload.peer);
    };
    this.room.onPeerLeft = (payload) => {
      this.peers.delete(payload.peer.id);
    };

    await this.room.join(token, { gateway: 'https://gateway.odin.4players.io' });
    await this.audioPlugin.setOutputDevice({});

    this.audioInput = await ODIN.DeviceManager.createAudioInput(null, {
      echoCanceller: true,
      noiseSuppression: true,
      gainController: true,
    });
    await this.room.addAudioInput(this.audioInput);
  }

  async leaveRoom() {
    if (this.audioInput) {
      await this.room?.removeAudioInput(this.audioInput);
      this.audioInput.close();
      this.audioInput = null;
    }

    if (this.room) {
      this.room.leave();
      this.room = null;
    }
  }

  toggleMute() {
    if (this.audioInput) {
      const newVolume = this.isMuted ? 1 : 0;
      this.audioInput.setVolume(newVolume);
      this.isMuted = !this.isMuted;
    }
  }

  getPeersArray() {
    return Array.from(this.peers.values());
  }
}
```
