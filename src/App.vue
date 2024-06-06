<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';

const chatLog = ref<HTMLDivElement | null>(null);
const messageInput = ref<HTMLInputElement | null>(null);
const websocket = ref<WebSocket | null>(null);
const challenges = ref<string[]>([]);
const selectedChallenge = ref<string>('');

const SERVER_URL = '167.99.159.184:8000';

onMounted(async () => {
  try {
    const response = await axios.get(`http://${SERVER_URL}/challenges`);
    challenges.value = response.data;
    console.log('Challenges:', challenges.value);
  } catch (error) {
    console.error('Failed to fetch challenges:', error);
  }
});

function sendMessage() {
  if (messageInput.value && websocket.value) {
    const message = messageInput.value.value;
    websocket.value.send(message);
    messageInput.value.value = '';
  }
}

function displayMessage(message: string) {
  if (chatLog.value) {
    const messageElement = document.createElement('p');
    messageElement.innerText = message;
    chatLog.value.appendChild(messageElement);
    chatLog.value.scrollTop = chatLog.value.scrollHeight;
  }
}

function sendSelectedChallenge() {
   websocket.value = new WebSocket(`ws://${SERVER_URL}/ws?challenge=` + selectedChallenge.value);

  websocket.value.onmessage = (event: MessageEvent) => {
    const message = event.data;
    displayMessage(message as string);
  };
}
</script>

<template>
  <div class="app">
    <div v-if="!websocket" class="select-screen">
      <el-select
        v-model="selectedChallenge"
        placeholder="Select"
        size="large"
        class="select"
      >
        <el-option
          v-for="item in challenges"
          :key="item"
          :label="item"
          :value="item"
        />
      </el-select>
      <el-button type="primary" @click="sendSelectedChallenge">Start Challenge</el-button>
    </div>
    <div class="terminal-chat-app" v-else>
      <h1>Terminal Chat App</h1>
      <div class="chat-container">
        <div ref="chatLog" class="chat-log"></div>
        <div class="input-container">
          <span class="prompt">&gt;</span>
          <input ref="messageInput" type="text" class="message-input" placeholder="Enter a message" @keyup.enter="sendMessage" />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.app {
  background-color: black;
}

.select-screen {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  flex-direction: column;
}

.select {
  width: 300px;
  margin-bottom: 20px;
}

.terminal-chat-app {
  font-family: monospace;
  color: magenta;
  height: 100vh;
  width: 100%;
  display: flex;
  align-items: center;
  flex-direction: column;
  border: 1px solid green;
}

h1 {
  margin: 0;
  padding: 20px;
}

.chat-container {
  display: flex;
  flex-direction: column;
  flex-grow: 1;
  width: 100%;
  overflow-y: auto;
}

.chat-log {
  flex-grow: 1;
  overflow-y: auto;
  padding: 10px;
}

.input-container {
  display: flex;
  align-items: center;
  padding: 10px;
}

.prompt {
  color: green;
  margin-right: 5px;
}

.message-input {
  background-color: black;
  color: green;
  border: none;
  padding: 5px;
  font-family: monospace;
  flex-grow: 1;
}
</style>