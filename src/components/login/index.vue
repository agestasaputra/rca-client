<template>
  <div>
    <div v-if="!joined" class="parent-container">
      <div class="name-container">
        <h1 class="title"> Enter your name: </h1>
        <input type="text" class="user-name" v-model="currentUser" @keyup.enter="onJoin" />
        <button class="join-button" :disabled="!isJoined" @click="onJoin"> Join </button>
      </div>
    </div>
    <div v-if="joined">
      <div class="wrapper">
        <div class="list-container">
          <div v-for="message in messages" :key="message.id">
            <span>
              <strong>
                {{ message.user }} :
              </strong>
              <span v-if="message.type === 'file'" class="message-file">
                <Image :message="message" />
              </span>
              <span v-else class="message-text">
                {{ message.body}}
              </span>
            </span>
          </div>
        </div>
        <span v-if="isTyping && isTyping !== currentUser"> {{ isTyping }} is typing... </span>
        <div class="text-input-container">
          <div class="top-input">
            <textarea
              v-model="text"
              class="text-message"
              @keyup.enter="onSendMessage"
            />
            <button class="chat-button" :disabled="!isChatted" @click="onSendMessage"> Send Message </button>
          </div>
          <div class="bottom-input">
            <input type="file" id="input-file" @change="onAttachedFile"/>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { reactive, toRefs, watch } from 'vue'
import io from 'socket.io-client'
import Image from '../image/index.vue'

export default {
  name: 'Login',
  components: { Image },
  setup() {
    const state = reactive({
      isTyping: false,
      isJoined: false,
      isChatted: false,
      joined: false,
      currentUser: "",
      text: "",
      messages: [],
      file: null,
      socketInstance: ''
    });

    watch(() => state.currentUser, (currentValue) => {
      if (currentValue.length > 0) {
        state.isJoined = true;
        } else {
        state.isJoined = false;
      }
    })
    watch(() => state.text, (currentValue) => {
      if (currentValue.length > 0) {
        state.isChatted = true;
        state.isTyping = state.currentUser;
        state.socketInstance.emit('typing', state.currentUser);
      } else {
        state.isChatted = false;
        state.isTyping = false;
        state.socketInstance.emit('stopTyping', state.currentUser);
      }
    })
    watch(() => state.file, (currentValue) => {
      if (currentValue) {
        state.isChatted = true;
        state.isTyping = state.currentUser;
        state.socketInstance.emit('typing', state.currentUser);
      } else {
        state.isChatted = false;
        state.isTyping = false;
        state.socketInstance.emit('stopTyping', state.currentUser);
      }
    })

    function onJoin() {
      state.joined = true;
      
      // register socket on client
      state.socketInstance = io("http://localhost:3000");

      // recieved message from socket
      state.socketInstance.on(
        "message:chat", (data) => {
          state.messages = state.messages.concat(data);
        }
      )
      state.socketInstance.on(
        "message:typing", (name) => {
          state.isTyping = name;
        }
      )
      state.socketInstance.on(
        "message:stopTyping", () => {
          state.isTyping = false;
        }
      )
    }

    function onSendMessage() {
      try {
        let message = {}

        if (state.file) {
          message = {
            id: new Date().getTime(),
            user: state.currentUser,
            type: "file",
            body: state.file,
            fileType: state.file.type,
            fileName: state.file.name,
          };
          state.messages = state.messages.concat(message);
        } else {
          message = {
            id: new Date().getTime(),
            user: state.currentUser,
            type: "text",
            body: state.text,
          };
          state.messages = state.messages.concat(message);
        }

        // send message to socket
        state.socketInstance.emit('message', message);
        state.socketInstance.emit('stopTyping', state.currentUser);
        onReset();

      } catch (error) {
        alert(`${error} - ${error.message}`)
        throw error;
      }
    }
    function onAttachedFile(event) {
      state.file = event.target.files[0]
    }
    function onReset() {
      state.text = '';
      state.isTyping = false;
      state.file = null;
      document.querySelector('#input-file').value = null;
    }

    return {
      ...toRefs(state),
      onJoin,
      onSendMessage,
      onAttachedFile
    }
  }

}
</script>

<style scoped>
  .title {
    text-align: center;
  }
  .parent-container {
    width: 100%;
    height: 100%;
    display: flex;
    justify-content: center;
    position: fixed;
    padding-top: 150px;
  }
  .name-container {
    display: flex;
    flex-direction: column;
    width: 300px;
  }
  .user-name {
    height: 30px;
    font-size: 20px;
    padding: 5px;
    margin-bottom: 5px;
    text-align: center;
    font-weight: bold;
  }
  .join-button {
    height: 30px;
    font-size: 20px;
  }

  .wrapper {
    height: 98vh;
    display: flex;
    flex-flow: column;
  }
  .list-container {
    flex: 1;
  }
  .text-input-container {
    gap: 5px;
  }
  .text-message {
    width: 100%;
    height: 70px;
    padding: 10px;
    box-sizing: border-box;
  }
  .chat-button {
    height: fit-content;
    min-height: 70px;
    font-size: 20px;
  }
  .top-input {
    display: flex;
    flex-flow: row;
  }
</style>