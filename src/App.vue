<script setup>
import { onBeforeMount, ref } from "@vue/runtime-core";
import { io } from "socket.io-client";

const socket = io("http://localhost:3001");
const messages = ref([{ name: "Preshy Jones", text: "heyoo" }]);
const messageText = ref("");
const joined = ref(false);
const name = ref("");
const typingDisplay = ref("");

onBeforeMount(() => {
  socket.emit("findAllMessages", {}, (response) => {
    messages.value = response;
  });

  socket.on("message", (message) => {
    console.log(message);
    messages.value.push(message);
  });

  socket.on("typing", ({ name, isTyping }) => {
    if (isTyping) {
      typingDisplay.value = `${name} is typing...`;
    } else {
      typingDisplay.value = "";
    }
  });
});

const join = () => {
  socket.emit("join", { name: name.value }, () => {
    joined.value = true;
  });
};
const sendMessage = () => {
  socket.emit("createMessage", { text: messageText.value }, () => {
    messageText.value = "";
  });
};

let timeout;
const emitTyping = () => {
  socket.emit("typing", { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit("typing", { isTyping: false });
  }, 2000);
};
</script>

<template>
  <div class="chat">
    <h1>{{ joined }}</h1>
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label>What's your name</label>
        <input type="text" v-model="name" />
        <button>send</button>
      </form>
    </div>
    <div v-else>
      <pre>{{ JSON.stringify(messages, null, 2) }}</pre>
      <div>
        <div v-for="(message, index) in messages" :key="index">
          [{{ message.name }}]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay">
        {{ typingDisplay }}
      </div>
      <div>
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style></style>
