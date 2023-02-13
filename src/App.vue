<template>
  <ChatRoom></ChatRoom>
</template>

<script>
import ChatRoom from "./components/ChatRoom/ChatRoom.vue";

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
