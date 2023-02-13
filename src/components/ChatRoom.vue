<template>
  <div v-if="!joined" class="container">
    <img
      src="https://pbs.twimg.com/profile_images/1565423195909398531/EZ77K6EX_400x400.jpg"
      alt="Legendaryum"
      class="logo"
    />
    <div class="nameContainer">
      <p class="putName">Put your name here and join to the Chat Room</p>
      <input type="text" class="name" v-model="currentUser" />
      <button class="joinBtn" @click="join">Join</button>
    </div>
  </div>

  <div v-if="joined">
    <div class="signedNameContainer">
      <p>Joined as ' {{ currentUser }} '</p>
      <button class="signOutBtn" @click="signOut">SignOut</button>
    </div>
    <div class="chatMasterContainer">
      <div class="messagesC">
        <div v-for="message in messages" :key="message.id">
          <b>{{ message.user }}</b>
          : {{ message.text }}
        </div>
      </div>
      <div class="chatContainer">
        <textarea
          v-model="text"
          class="textMessage"
          v-on:keyup.enter="sendMessage"
        >
        </textarea>
      </div>
    </div>
  </div>
</template>

<script>
// import io from "socket.io-client";
import { Client } from "colyseus.js";
// import { Client } from "colyseus.js";

export default {
  name: "ChatRoom",
  data() {
    return {
      joined: false,
      currentUser: "",
      text: "",
      room: null,
      messages: [
        {
          id: 1,
          text: "hi men",
          user: "manoj",
        },
      ],
    };
  },
  methods: {
    async join() {
      this.joined = true;
      // this.socketInstance = io("http://localhost:3000");
      //   this.socketInstance.on("message:received", (message) => {
      //     this.messages.push(message);
      //   });
      try {
        console.log(Client);
        // var host = window.document.location.host.replace(/:.*/, "");
        // console.log(host);
        var client = new Client(
          //   location.protocol.replace("http", "ws") + "//" + host + ":8001"
          "ws://localhost:8001"
        );
        // console.log(client);
        const self = this;
        this.room = await client.joinOrCreate("chatRoom");
        // client.getAvailableRooms().then((data) => console.log(data));

        console.log(client, this.room);
        if (this.room !== undefined) {
          this.room.onMessage("messages", function ({ message }) {
            console.log(message);
            self.addMessage(message);
          });
        }
      } catch (err) {
        console.log("Client connection error", err);
        alert("Client connection error", err);
      }

      /*Listen to the messages sent by the server by subscribing to the message event on the room instance. */
      this.room.onStateChange.once((state) => {
        console.log("initial room state:", state);
      });
    },
    signOut() {
      this.joined = false;
      this.currentUser = "";
    },
    sendMessage() {
      this.addMessage();

      this.room.send("messageToServer", {
        client: this.currentUser,
        message: this.text,
      });
      this.text = "";
    },
    addMessage(payload) {
      let message;
      !payload
        ? (message = {
            id: new Date().getTime(),
            text: this.text,
            user: this.currentUser,
          })
        : (message = {
            id: new Date().getTime(),
            text: payload.message,
            user: payload.client,
          });
      this.messages.push(message);
      //   this.socketInstance.emit("message", message);
    },
  },
};
</script>

<style scoped>
.logo {
  width: 23vw;
  height: 40vh;
  position: relative;
  /* justify-content: center; */
  /* margin-right: 200px; */
  left: 280px;
}
.container {
  width: 98vw;
  height: 98vh;
  display: flex;
  /* position: relative; */
  /* align-items: center; */
  justify-content: center;
  margin-top: 110px;
}
.nameContainer {
  display: flex;
  flex-direction: column;
  align-content: center;
  justify-content: center;
  margin-right: 330px;
  margin-top: 70px;
}
.putName {
  /* justify-content: center; */
  font-size: 20px;
  align-content: center;
}
.signedNameContainer {
  /* position: absolute; */
}

.name {
  height: 40px;
  font-size: 20px;
  padding: 5px;
  margin-bottom: 10px;
  text-align: center;
  font-weight: bold;
  border-radius: 5px;
}
.joinBtn {
  height: 30px;
  font-size: 20px;
  background-color: rgb(243, 253, 229);
  border-radius: 15px;
}
.signOutBtn {
  height: 30px;
  font-size: 20px;
  background-color: rgb(255, 230, 230);
}
.chatContainer {
  /* height: 100vh; */
}
.textMessage {
  width: 97vw;
  height: 80px;
  position: absolute;
  bottom: 0px;
  margin: 10px;
  padding: 10px;
}
</style>
