<script setup>
import {io} from 'socket.io-client'
import {onBeforeMount, ref} from 'vue';

const socket = io('http://localhost:3001');

const messages = ref([]);
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');
const messageText = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessages', {}, (response) =>{
    messages.value = response;
  });
    socket.on('message', (message) => {
      messages.value.push(message);
    });
    socket.on('typing', ({ name, isTyping}) =>{
      if(isTyping)
      {  typingDisplay.value = `${name} is typing...`;}
      else
        {typingDisplay.value = '';}
    });
});



let timeout;
const emitTyping = () =>{
  socket.emit('typing', {isTyping: true});
  timeout = setTimeout(() => {
    socket.emit('typing', {isTyping: false});
  }, 2000);
}

const join = () =>{
  socket.emit('join', {name: name.value}, () => {
    joined.value = true;
  } )
}

const sendMessage = () =>{
  socket.emit('createMessage', { text: messageText.value}, () => {
    messageText.value = '';
  })
}
</script>

<template>
<div class="chat">
  
  <div v-if="!joined">
    <form @submit.prevent="join">
    <label> What's your name?</label>
    <input v-model="name" />
    <button type="submit">Send</button>
    </form>
  </div>
 <div v-if="joined">
  <div class= "chat-container">
   
    <div class= "messages-container">
      <div v-for="message in messages">
        [{{message.name}}]: {{message.text}}
      </div>
    </div>
    </div>
    <div v-if="typingDisplay">{{ typingDisplay }}</div>
    <div class="messageInput">
      <form @submit.prevent="sendMessage">
        <label>Message:</label>
        <input v-model="messageText" @input="emitTyping" />
        <button type="submit">Send</button>
      </form>
    </div>
    
  </div>
  
</div>
  
</template>

<style>
@import './assets/base.css';
.chat{
  padding: 20px;
  height: 100vh;
  color: blueviolet;
}

.chat-container{
  display: flex;
  flex-direction: column;
  height: 100%;
}

.message-container{
  flex: 1;
}
</style>
