<template>
    
    <h1>Welcome to Real Time Chat App with Colyseus!</h1>
    <div v-if="!joined" class="container">
        <div class="nameContainer">
            <input type="text" class="name" v-model="currentUser">
            <button class="joinBtn" @click="join">Join</button>
        </div>
    </div>
    <div v-if="joined" >
        <div class="nameContainer">
            <p>{{currentUser}} Joined</p>
            <button class="signOutBtn" @click="signOut">SignOut</button>
        </div>

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
            v-on:keyup.enter="sendMessage">
        </textarea>
        </div>
    </div>
</template>

<script>
import io from "socket.io-client"
export default {
    name: "ChatRoom",
    data() {
        return {
            joined: false,
            currentUser: "",
            text: "",
            messages: [
                {
                    id: 1,
                    text: "hi men",
                    user: "manoj"
                }
            ],

        }
    },
    methods: {
        join() {
            this.joined = true
            this.socketInstance =  io("http://localhost:3000")

            this.socketInstance.on(
                "message:received", (message) => {
                this.messages.push(message)
                }
            
            
            )
        },
        signOut() {
            this.joined = false
            this.currentUser = ""

        },
        sendMessage() {
            this.addMessage()
            this.text = ""
        },
        addMessage() {
            const message = {
                id: new Date().getTime(),
                text: this.text,
                user: this.currentUser
            }
            this.messages.push(message)
            this.socketInstance.emit("message", message)
        }

    }
}
</script>

<style scoped>
.container{
    width: 100vw;
    height: 100vh;
    display: flex;
    position: fixed;
    justify-content: center;
    padding-top:50px;

}
.nameContainer{
    display: flex;
    flex-direction: column;
}
.name{
    height: 30px;
    font-size: 20px;
    padding: 5px;
    margin-bottom: 5px;
    text-align: center;
    font-weight: bold;

}
.joinBtn{
    height: 30px;
    font-size: 20px;
}
.signOutBtn{
    height: 30px;
    font-size: 20px;
    color:red
}
.chatContainer{
/* height: 100vh; */
}
.textMessage{
    width: 97vw;
    height: 80px;
    position: absolute;
    bottom:0px;
    margin: 10px;
    padding: 10px;

}
</style>