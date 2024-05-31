<script setup lang="ts">
import { ref, onMounted } from 'vue';

const chatLog = ref<HTMLDivElement | null>(null);
const messageInput = ref<HTMLInputElement | null>(null);
const websocket = ref<WebSocket | null>(null);

onMounted(() => {
  websocket.value = new WebSocket("ws://localhost:8000/ws");

  websocket.value.onmessage = (event: MessageEvent) => {
    const message = event.data;
    displayMessage(message as string);
  };
});

function sendMessage() {
  if (messageInput.value && websocket.value) {
    const message = messageInput.value.value;
    websocket.value.send(message);
    messageInput.value.value = "";
  }
}

function displayMessage(message: string) {
  if (chatLog.value) {
    const messageElement = document.createElement("p");
    messageElement.innerText = message;
    chatLog.value.appendChild(messageElement);
    chatLog.value.scrollTop = chatLog.value.scrollHeight;
  }
}
</script>

<template>
  <div class="terminal-chat-app">
    <div class="container">
      <h1>Terminal Chat App</h1>
      <div ref="chatLog" class="chat-log"></div>
      <div class="input-container">
        <span class="prompt">&gt;</span>
        <input ref="messageInput" type="text" class="message-input" placeholder="Enter a message" @keyup.enter="sendMessage" />
      </div>
    </div>
  </div>
</template>

<style scoped>
.terminal-chat-app {
  background-color: black;
  height: 100vh;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  background-color: black;
  color: green;
  font-family: monospace;
  width: 80%;
  height: 80%;
  display: flex;
  flex-direction: column;
  border: 1px solid green;
}

h1 {
  margin: 0;
  padding: 20px;
}

.chat-log {
  flex: 1;
  overflow-y: scroll;
  padding: 10px;
  padding-bottom: 0;
}

.input-container {
  display: flex;
  align-items: center;
  padding: 0 10px;
}

.prompt {
  color: green;
  margin-right: 5px;
}

.message-input {
  flex: 1;
  background-color: black;
  color: green;
  border: none;
  padding: 5px;
  font-family: monospace;
}
</style>