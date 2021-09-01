<template>
  <div>
    <div class="online-users">
      <p>{{ onlineUsers }}</p>
    </div>
    <form
      class="user-form"
      :class="{ hide: isHidden }"
      @submit.prevent="addUserName"
    >
      <input
        class="user-input"
        name="user-input"
        placeholder="your name?"
        ref="userInput"
        autocomplete="off"
        v-model="userName"
      />
    </form>

    <div class="messages-container">
      <ul id="messages" v-for="(message, index) in messages" :key="index">
        <li
          class="message__row"
          :class="message.id == id ? 'message-out' : 'message-in'"
        >
          <div
            class="message__container"
            :style="{ background: message.background }"
            ref="message"
          >
            <div class="message__top">
              <p class="message__user">{{ message.user }}</p>
              <p class="message__date">{{ message.date }}</p>
            </div>
            <div v-if="message.messageType === 'text'">
              <p class="message__message">{{ message.message }}</p>
            </div>
            <div v-if="message.messageType === 'image'">
              <img
                class="image"
                :src="require('./assets/' + message.message)"
                alt=""
              />
            </div>
          </div>
        </li>
      </ul>

      <form id="form" @submit.prevent="sendMessage">
        <input
          type="text"
          id="input"
          ref="messageInput"
          autocomplete="off"
          v-model="message"
        />
        <div class="icon">
          <label for="upload">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 24 24"
              width="24"
              height="24"
            >
              <path
                fill="currentColor"
                d="M1.816 15.556v.002c0 1.502.584 2.912 1.646 3.972s2.472 1.647 3.974 1.647a5.58 5.58 0 0 0 3.972-1.645l9.547-9.548c.769-.768 1.147-1.767 1.058-2.817-.079-.968-.548-1.927-1.319-2.698-1.594-1.592-4.068-1.711-5.517-.262l-7.916 7.915c-.881.881-.792 2.25.214 3.261.959.958 2.423 1.053 3.263.215l5.511-5.512c.28-.28.267-.722.053-.936l-.244-.244c-.191-.191-.567-.349-.957.04l-5.506 5.506c-.18.18-.635.127-.976-.214-.098-.097-.576-.613-.213-.973l7.915-7.917c.818-.817 2.267-.699 3.23.262.5.501.802 1.1.849 1.685.051.573-.156 1.111-.589 1.543l-9.547 9.549a3.97 3.97 0 0 1-2.829 1.171 3.975 3.975 0 0 1-2.83-1.173 3.973 3.973 0 0 1-1.172-2.828c0-1.071.415-2.076 1.172-2.83l7.209-7.211c.157-.157.264-.579.028-.814L11.5 4.36a.572.572 0 0 0-.834.018l-7.205 7.207a5.577 5.577 0 0 0-1.645 3.971z"
              ></path>
            </svg>
            <input
              type="file"
              ref="file"
              id="upload"
              style="display: none"
              @change="selectFile"
            />
          </label>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import io from "socket.io-client";
import axios from "axios";
export default {
  data() {
    return {
      isHidden: false,
      userName: "",
      messages: [],
      users: [],
      message: "",
      file: "",
      socket: {},
      id: "",
      update: 0,
    };
  },
  computed: {
    onlineUsers() {
      return this.users.join(", ");
    },
  },
  methods: {
    focusInput() {
      this.$refs.userInput.focus();
    },
    sendMessage(e) {
      e.preventDefault();
      this.socket.emit("new message", {
        messageType: "text",
        message: this.message,
        user: this.userName,
      });
      this.message = "";
      this.scrollToLastMessage();
    },
    scrollToLastMessage() {
      const el = this.$refs.message;
      if (el) {
        el.scrollIntoView({ behavior: "smooth" });
      }
    },
    addUserName() {
      this.isHidden = true;
      this.$refs.messageInput.focus();
      this.socket.emit("user online", {
        user: this.userName,
      });
    },
    selectFile() {
      const file = this.$refs.file.files[0];
      this.file = file;
      this.sendFile();
    },
    async sendFile() {
      const formData = new FormData();
      formData.append("file", this.file);
      await axios
        .post("/api/upload", formData)
        .then((response) => {
          const image = response.data.slice(18);
          this.socket.emit("new image", {
            messageType: "image",
            message: image,
            user: this.userName,
          });
        })
        .catch((err) => {
          console.log(err);
        });
    },
    showOnlineUsers(data) {
      data.users.map((user) => {
        if (user.id === this.id) {
          user.user = "you";
        }
      });
      let userNames = data.users.map((user) => user.user);
      userNames.push(userNames.splice(userNames.indexOf("you"), 1)[0]);
      this.users = userNames;
    },
  },
  mounted() {
    this.focusInput();
  },
  created() {
    this.socket = io(process.env.PORT);

    this.socket.on("send id", (data) => {
      this.id = data.id;
    });

    this.socket.on("user online", (data) => {
      this.showOnlineUsers(data);
    });

    this.socket.on("user disconnected", (data) => {
      this.showOnlineUsers(data);
    });

    this.socket.on("new message", (data) => {
      this.scrollToLastMessage();
      this.messages = [...this.messages, data];
    });

    this.socket.on("new image", (data) => {
      this.scrollToLastMessage();
      this.messages = [...this.messages, data];
    });
  },
};
</script>

<style>
body {
  margin-top: 3rem;
  margin-bottom: 8rem;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica,
    Arial, sans-serif;
  background: rgb(243, 238, 238);
  overflow: auto;
}
.online-users {
  background: rgba(0, 0, 0, 0.925);
  color: white;
  padding: 0.25rem;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  display: flex;
  height: 3rem;
  box-sizing: border-box;
  backdrop-filter: blur(10px);
}
.online-users > p {
  margin-left: 1rem;
}

#form {
  background: rgba(0, 0, 0, 0.925);
  padding: 0.25rem;
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  height: 3rem;
  box-sizing: border-box;
  backdrop-filter: blur(10px);
}
.user-form {
  background: rgb(243, 238, 238);
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.user-input {
  border: 0;
  outline: 0;
  background: transparent;
  color: black;
  margin: auto;
  font-size: 3rem;
  width: 16rem;
  cursor: pointer;
}

#input {
  border: none;
  padding: 0 1rem;
  flex-grow: 1;
  border-radius: 2rem;
  margin: 0.25rem;
}

#input:focus {
  outline: none;
}

#form > button {
  background: #333;
  border: none;
  padding: 0 1rem;
  margin: 0.25rem;
  border-radius: 3px;
  outline: none;
  color: #fff;
}

.icon {
  background: #333;
  border: none;
  padding: 0 1rem;
  margin: 0.25rem;
  border-radius: 3px;
  outline: none;
  color: #fff;
}
.icon:hover {
  background: #fff;
  color: #333;
}

#messages {
  list-style-type: none;
  margin: 0;
  padding: 0;
}

.message__row {
  display: flex;
  flex-direction: column;
}

.message__container {
  max-width: 70%;
  min-width: 20%;
  margin-left: 1rem;
  margin-top: 1rem;
  padding: 0.3rem 0.5rem;
  border: 1px solid #aa9d9c;
  color: #474040;
  border-radius: 10px;
  box-shadow: 5px 5px 10px -12px #000000;
  display: flex;
  flex-direction: column;
}
.image {
  max-width: 100%;
  border-radius: 5px;
}

.message__top {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

.message__message {
  margin: 0;
  padding: 0;
}

.message__user {
  padding: 0;
  margin: 0;
  font-weight: 700;
}

.message__date {
  font-size: 8px;
}

.hide {
  display: none;
}

.user-name {
  font-size: 0.7rem;
}

.message-out {
  align-items: flex-end;
}

.message-out > .message__container {
  margin-right: 1rem;
}

.message-out > .message__container > .message__top > .message__user {
  color: #7e4436;
}

.message-in > .message__container > .message__top > .message__user {
  color: rgb(54, 126, 102);
}

svg {
  margin-top: 3px;
}

.input {
  display: hidden;
}

@media screen and (min-width: 600px) {
  .message__container {
    max-width: 25rem;
  }
}
</style>

