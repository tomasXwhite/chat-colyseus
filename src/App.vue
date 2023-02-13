<template>
  <!-- <div class="title">

    <img alt="Vue logo" src="./assets/logo.png"> <h1 class="titleText">Vue+sockets Chat</h1>
  </div> -->
  <ChatRoom></ChatRoom>
</template>

<script>
import ChatRoom from "./components/ChatRoom.vue";

export default {
  name: "App",
  components: {
    ChatRoom,
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /* text-align: center; */
  color: #2c3e50;
  /* margin-top: 30px; */
  width: 100%;
  height: 70vh;
}
:root {
}
.title {
  display: flex;
  flex-direction: row;
  margin: 10px;
  justify-content: flex-start;
  /* position:absolute */
}
.titleText {
  margin: 50px;
  font-weight: bold;
}
</style>
