<template>
  <div class="page-container">
    <div class="md-layout-item">
      <!-- Change room select -->
      <md-field>
        <label for="room">Room</label>
        <md-select v-model="room" @md-selected="onChangeRoom" name="room" id="room">
          <md-option v-for="room in this.$store.state.rooms" :key="room.id" :value="room.name">{{room.name}}</md-option>
        </md-select>
      </md-field>
    </div>

    <md-app md-waterfall md-mode="fixed">
      <!-- Room title and logout -->
      <md-app-toolbar class="md-primary">
        <span class="md-title page-container__room">{{room}}</span>
        <md-button class="md-icon-button page-container-logout" @click.native="logout()">
          <md-icon>power_settings_new</md-icon>
        </md-button>
      </md-app-toolbar>

      <!-- Connected users list -->
      <md-app-drawer md-permanent="full">
        <!-- Display the users and emit an event when the user open a private chat -->
        <UserList
          :users="users"
          :openPrivateChat="openPrivateChat.chat"
          @open-chat="openChat($event)"
        ></UserList>
      </md-app-drawer>

      <!-- Chat area with all the messages -->
      <md-app-content>
        <!-- As an input it display the messages received from the server -->
        <ChatArea :messages="messages"></ChatArea> 
      </md-app-content>
    </md-app>

    <!-- Text area to write on. It emits an event each time the user sends a message -->
    <MessageArea 
      @send-message="sendMessage($event)"> 
    </MessageArea>

    <!-- Private chat. The showDialog input controls whether we open a private chat or not -->
    <!-- The openPrivateChat is an object defined in the Vue chat component that contains 
         information to handle the private chat -->
    <ChatDialog 
      :showDialog="openPrivateChat" 
      @close-chat="closePrivateChat()">
    </ChatDialog>
  </div>
</template>


<script>

import UserList from "./../components/UserList";
import ChatArea from "./../components/ChatArea";
import MessageArea from "./../components/MessageArea";
import ChatDialog from "./../components/ChatDialog";

import { url, STORE_ACTIONS, WS_EVENTS } from "./../utils/config";

export default {
   name:"chat",

   components: {
    UserList,
    ChatArea,
    MessageArea,
    ChatDialog
  },

   sockets:{
       newUser({users,username}){
           const isMe= this.$store.state.username===username;
           if(!isMe){
               if(users.length>this.users.length){
                   this.messages.push({join:true, msg:`${username} has joined the room`});       
                }
                else if(users.length<this.users.length){
                    this.messages.push({join:true, msg:`${username} has left the room`});
                }
            }
       this.users.length=0;
       this.users=users;
   },

    newMessage({message,username}){
        //if message is empty
        if(message.replace(/\s/g, "").length === 0)
        return;

        const isMe=this.$store.state.username===username;
        const msg=isMe? `${message}`:{username,message};

        const msgObj={msg,isMe};
        this.messages.push(msgObj);
    },

    privateChat({username,to}){
        const isForMe=this.$store.state.username===to;
        if(isForMe && !this.openPrivateChat.chat){
            //then joiin the private room
            this.$socket.emit(WS_EVENTS.joinPrivateRoom, {
                to:this.$store.state.username,
                room:null,
                username
            });
        }
    },

    privateMessage({privateMessage, to, from , room}){
        console.log(`New private message from ${from} in the room ${room}`);

        const isObj=typeof privateMessage === "object";
        const isForMe=this.$store.state.username===to;
        const isFromMe=this.$store.state.username===from;

        if(isObj && isFromMe)
        return false;

        //openPrivateChat with given information
        if(!this.openPrivateChat.chat){
            this.openPrivateChat={
                ...this.openPrivateChat,
                chat:true,
                user:from,
                room:room
            };
        }

        const msgObj={
            msg:isObj ?privateMessage.msg:privateMessage ,
            isMe:!isForMe
        };
        this.openPrivateChat.msg.push(msgObj);
    },

    leavePrivateRoom({privateMessage,from}){
        if((from ===this.openPrivateChat.user || from ===this.$store.state.username) && this.openPrivateChat.chat){
            this.openPrivateChat.msg.push({msg:privateMessage});
            this.openPrivateChat={...this.openPrivateChat, closed:true};
        }
    },

    leaveChat({users, username}){
        this.messages.push({join:true,msg:`${username} has left the room`});
        this.users.length=0;
        this.users=users;

        if(username ===this.$store.state.username){
            this.$store.dispatch(STORE_ACTIONS.leaveChat)
                .then(()=>{
                    this.$router.push("/");
                });
            }
        }
    },

    beforeCreate() {
        this.$socket.emit(WS_EVENTS.joinRoom, this.$store.state);
    },

    data(){
        return{
            room:this.$store.state.room,
            users:[],
            messages:[],
            openPrivateChat:{
                chat:false,
                user:null,
                masg:[],
                room:null,
                closed:false
            }
        };
    },

    methods:{
        onChangeRoom(val){
            if(this.$store.state.room!==val){
                this.$socket.emit(WS_EVENTS.leaveRoom, this.$store.state);
                this.$store.dispatch(STORE_ACTIONS.ChangeRoom, val);
                this.messages.length=0;
                this.$socket.emit(WS_EVENTS.joinRoom, this.$store.state);
            }
        },

        sendMessage(msg){
            this.$socket.emit(WS_EVENTS.publicMessage, {
                ...this.$store.state,
                message:msg
            });
        },

        openChat(user) {
        this.openPrivateChat = {
                ...this.openPrivateChat,
                chat: true,
                user: user,
                room: user // The room is the username who talk with
            };
        },

        closePrivateChat() {
        // leavePrivate room emit
        this.$socket.emit(WS_EVENTS.leavePrivateRoom, {
            room: this.$store.state.room,
            to: this.openPrivateChat.room,
            from: this.$store.state.username
        });

        this.openPrivateChat = {
            ...this.openPrivateChat,
            chat: false,
            closed: false,
            user: null,
            msg: [],
            room: null
        };
    },

    async logout() {
        try {
            let response=await this.$http.post(`http://${url}/auth/logout`, { 
                username:this.$store.state.username
            });
            if(response.body.code===200){
                this.$socket.emit(WS_EVENTS.leaveChat, {
                    room:this.$store.state.room,
                    username:this.$store.state.username
                });
            }
        }
        catch(err){
            console.log(err);
        }
    }

    }
};
</script>

<style lang="scss">
@import "./../styles/variables";
.page-container {
  height: 100%;
  background: url("./../assets/bck.jpg");
  .md-field {
    width: 200px;
    margin: 0 auto;
    margin-bottom: 3rem;
    & label {
      color: white;
    }
    & .md-icon-image svg{
      fill: white;
    }
    & .md-menu.md-select {
      border-bottom: 1px solid white;
    }
    &.md-theme-default.md-has-value .md-input {
      -webkit-text-fill-color: white !important;
    }
    .md-menu.md-select .md-input {
      -webkit-text-fill-color: white !important;
    }
  }
  .md-toolbar.md-theme-default {
    &.md-transparent {
      background: $secondary_blue;
      color: white;
      /* position: fixed; */
      font-size: 17px;
    }
    &.md-primary {
      background-color: $main_blue;
      & .md-title {
        font-weight: bold;
      }
      & .md-icon {
        font-weight: bold;
      }
    }
  }
  .md-layout-item {
    padding-top: 2rem;
  }
  .md-app {
    width: 85%;
    margin: 0 auto;
    height: 70vh;
    max-width: 1300px;
    & .md-content.md-theme-default {
      background: url("./../assets/msg_bck.png");
      background-attachment: fixed;
      background-size: 100% 100%;
      border-left: 0;
    }
    & .md-layout-column.md-flex.md-theme-default.md-scrollbar{
        background: url("./../assets/msg_bck.png");
        background-attachment: fixed;
        background-size: 100% 100%;
    }
    &.md-fixed .md-app-scroller{
      background: url("./../assets/msg_bck.png");
      background-attachment: fixed;
      background-size: 100% 100%;
      border-left: 1px solid rgba(0,0,0,0.12);
    }
  }
  
  .md-app-toolbar {
    display: block;
  }
  .md-drawer {
    width: 270px;
  }
  &__room {
    float: left;
    padding-top: 19px;
  }
  &-logout {
    float: right;
    margin-top: 12px;
  }
}
</style>