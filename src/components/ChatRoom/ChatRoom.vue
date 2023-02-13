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
        <b class="username" v-else>You :</b>
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

    <div class="connectedUsersC">
      <ul class="list">
        Users connected:
        <li class="list-item" v-for="user in connectedUsers" :key="user">
          `{{ user }}` Connected to the server
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import { Client } from "colyseus.js";
import "./styles.css";

export default {
  name: "ChatRoom",
  data() {
    return {
      joined: false,
      currentUser: "",
      text: "",
      room: null,
      client: null,
      messages: [],
      connectedUsers: [],
      availableRooms: [],
    };
  },
  mounted() {
    console.log("MOUNTED");
    let host = window.document.location.host.replace(/:.*/, "");
    this.client = new Client(
      location.protocol.replace("http", "ws") + "//" + host + ":8001"
    );
    this.client
      .getAvailableRooms()
      .then((data) => (this.availableRooms = data));
    console.log(this.availableRooms);
  },
  methods: {
    async join() {
      this.joined = true;
      try {
        this.room = await this.client.joinOrCreate("chatRoom", {
          username: this.currentUser,
        });
        console.log(this.currentUser);
        console.log(this.room.clients);
        // if (this.room.hasJoined) console.log(this.room);

        console.log(this.client, this.room);

        if (this.room) {
          this.handleConnectedUsers();
          this.handleMessages();
          this.handleDisconnectedUsers();
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
      if (message.text.length) {
        this.messages.push(message);
      }
    },
    async handleMessages() {
      if (this.room) {
        await this.room.onMessage("messages", ({ message }) => {
          console.log(message);
          this.addMessage(message);
        });
      }
    },
    async handleConnectedUsers() {
      //   if (this.room) {
      await this.room.onMessage("joined", ({ joinedUser }) => {
        console.log(joinedUser);
        this.connectedUsers.push(joinedUser);
      });
      //   }
    },
    async handleDisconnectedUsers() {
      if (this.room) {
        await this.room.onMessage("left", ({ username }) => {
          console.log(username);
          this.connectedUsers.filter((u) => u === username);
        });
      }
    },
  },
};
</script>
