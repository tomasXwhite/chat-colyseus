<template>
  <div v-if="!joined" class="landing-page">
    <img
      src="https://pbs.twimg.com/profile_images/1565423195909398531/EZ77K6EX_400x400.jpg"
      alt="Legendaryum"
      class="logo"
    />
    <div class="nameContainer">
      <p class="putName">Put your name here and join to the Chat Room</p>
      <input type="text" class="name-input" v-model="currentUser" />
      <button class="submit-button" @click="join">Join</button>
    </div>
  </div>

  <div v-if="joined">
    <div class="signedNameContainer">
      <p>Joined as '{{ currentUser }}' on room -{{ room.id }}-</p>
      <button class="signOutBtn" @click="signOut">SignOut</button>
    </div>
    <div class="chat-container">
      <!-- <div class="chat-message"> -->
      <div
        v-for="message in messages"
        :key="message.id"
        v-bind:class="{
          'chat-message-sender': message.user === currentUser,
          'chat-message-receiver': message.user !== currentUser,
        }"
      >
        <b class="username" v-if="message.user !== currentUser"
          >{{ message.user }} :</b
        >
        <b class="username" v-else>Tu :</b>
        {{ message.text }}
      </div>
      <!-- </div> -->
      <div class="inputAndBtn">
        <input
          type="text"
          v-model="text"
          class="chat-input"
          v-on:keyup.enter="sendMessage"
        />
        <!-- </input> -->
        <button class="chat-button" @click="sendMessage">Send</button>
      </div>
    </div>

    <ul class="list">
      Users connected in the room:
      <li class="list-item" v-for="user in connectedUsers" :key="user">
        {{ user }}
      </li>
    </ul>
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
      messages: [],
      connectedUsers: [],
    };
  },
  methods: {
    async join() {
      this.joined = true;
      try {
        console.log(Client);
        var host = window.document.location.host.replace(/:.*/, "");
        var client = new Client(
          location.protocol.replace("http", "ws") + "//" + host + ":8001"
        );
        const self = this;
        this.room = await client.joinOrCreate("chatRoom", {
          username: this.currentUser,
        });
        if (this.room.hasJoined) console.log(this.room);

        // client.getAvailableRooms().then((data) => console.log(data));

        console.log(client, this.room);
        if (this.room !== undefined) {
          this.room.onMessage("messages", function ({ message }) {
            console.log(message);
            self.addMessage(message);
          });
          this.room.onMessage("joined", function (userJoined) {
            console.log(userJoined);
            self.addConnectedUser(userJoined.joinedUser);
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
    addConnectedUser(user) {
      this.connectedUsers.push(user);
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
      if (message.text.length) {
        this.messages.push(message);
      }

      //   this.socketInstance.emit("message", message);
    },
  },
};
</script>

<style scoped>
.landing-page {
  display: flex;
  flex-direction: column;
  align-items: center;
  height: 100vh;
  /* justify-content: center; */
  background-color: #f0f0f0;
}
.name-input {
  padding: 10px 20px;
  font-size: 20px;
  border-radius: 5px;
  border: 1px solid #ccc;
  margin-bottom: 20px;
}

.submit-button {
  padding: 10px 20px;
  font-size: 20px;
  border-radius: 5px;
  background-color: #4caf50;
  color: #fff;
  cursor: pointer;
  border: none;
  font-weight: bold;
  margin-left: 30px;
}
.logo {
  width: 23vw;
  height: 40vh;
  position: relative;
  justify-content: center;
  margin-top: 30px;
  /* left: 250px; */
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
/* .nameContainer {
  display: flex;
  flex-direction: column;
  align-content: center;
  justify-content: center;
  margin-right: 330px;
  margin-top: 70px;
} */
.putName {
  /* justify-content: center; */
  font-size: 20px;
  align-content: center;
  border-radius: 20px;
}
.signOutBtn {
  padding: 15px 30px;
  border-radius: 5px;
  background-color: #f44336;
  color: #fff;
  cursor: pointer;
  border: none;
  font-weight: bold;
  float: right;
  display: flex;
  /* align-items: flex-end; */
  /* text-align: right; */
  position: absolute;
  right: 130px;
  top: 56px;
}
.signedNameContainer {
  font-size: 36px;
  font-weight: bold;
  text-align: center;
  margin: 50px 0;
}

.joinBtn {
  margin-top: 20px;
  height: 30px;
  font-size: 20px;
  background-color: rgb(243, 253, 229);
  border-radius: 15px;
}

.chat-container {
  width: 70vw;
  height: 555px;
  overflow: auto;
  margin: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.1);
}

.inputAndBtn {
}

/* Style for the chat messages */
.chat-message-sender {
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 10px;
  background-color: #f2f2f2;
  text-align: right;
}
.chat-message-receiver {
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 10px;
  background-color: #f2f2f2;
  text-align: left;
}

/* Style for the username */
.username {
  font-weight: bold;
  color: #333;
}

/* Input field for sending messages */
.chat-input {
  width: 60vw;
  padding: 10px;
  /* margin-top: 10px; */
  border-radius: 10px;
  border: 1px solid #ccc;
  box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.1);
}

/* Button for sending messages */
.chat-button {
  width: 100px;
  padding: 10px;
  margin: 10px;
  /* margin-left: 50px; */
  right: 20px;
  /* align-items: right; */
  border-radius: 10px;
  background-color: #4caf50;
  color: #fff;
  cursor: pointer;
}

.list {
  list-style: none;
  /* position: absolute; */
  right: 0;
  top: 0;
  margin: 0;
  padding: 0;
  background-color: #f0f0f0;
}

.list-item {
  display: block;
  padding: 10px;
}
</style>
