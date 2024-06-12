<template>
  <div v-if="selectedFriend && selectedFriend.email" class="chat-container"> 
    <div class="chat-header">
      <h2>Chat with {{ selectedFriend.name }}</h2>
    </div>
    <div class="message-container">
      <div v-for="(message, index) in messages" :key="index" :class="['message', {'from-current-user': message.user.email === currentUser.email, 'from-friend': message.user.email !== currentUser.email}]">
        <div class="message-content">
          <div class="avatar-container" :class="{'align-left': message.user.email === currentUser.email, 'align-right': message.user.email !== currentUser.email}">
            <span v-if="message.user" class="user-avatar">
              <i class="fas fa-user-circle"></i>
            </span>
          </div>
          <div :class="{'align-left': message.user.email === currentUser.email, 'align-right': message.user.email !== currentUser.email}">
            <span v-if="message.user" class="username">{{ message.user.username }}</span>
            <span class="message-text">{{ message.text }}</span>
          </div>
        </div>
      </div>
    </div>
    <div class="input-container">
      <input type="text" v-model="newMessage" @keyup.enter="sendMessage" placeholder="Type your message..." class="message-input" />
      <button @click="sendMessage" class="send-button">Send</button>
    </div>
  </div>
  <div v-else>
    <p>Please select a friend to start chatting.</p>
  </div>
</template>



<script>
import { mapState } from 'vuex';
import io from 'socket.io-client';

export default {
  data() {
    return {
      messages: [],
      newMessage: ''
    };
  },
  computed: {
    ...mapState({
      currentUser: state => state.currentUser,
      selectedFriend: state => state.userProfile.selectedFriend  
    })
  },
  mounted() {
    console.log('Current selected friend in Chat.vue:', this.selectedFriend); // Provjerava je li odabran prijatelj i ima li email
    if (this.selectedFriend) {
      const URL = 'http://localhost:3000'; 
      console.log('Initializing WebSocket connection to:', URL);
      this.socket = io(URL);

      this.socket.on('connect', () => {
        console.log('WebSocket connection established');
        this.socket.emit('register', this.currentUser.email);
      });

      this.socket.on('connect_error', (error) => {
        console.error('Connection Error:', error);
      });

      this.socket.on('message', message => {
        console.log('New message received:', message);
        if (message.user.email === this.selectedFriend.email) {
          this.messages.push({
            text: message.text,
            user: message.user
          });
        }
      });

      this.socket.on('disconnect', (reason) => {
        console.log('Disconnected:', reason);
        if (reason === 'io server disconnect') {
          this.socket.connect();
        }
      });

      this.socket.on('error', (error) => {
        console.error('WebSocket error:', error);
      });
    }
  },
  methods: {
    sendMessage() {
      if (!this.selectedFriend || !this.selectedFriend.email) {
        console.error('Recipient is not selected or does not have an email attribute.');
        return;
      }

      if (this.newMessage.trim() !== '' && this.socket && this.socket.connected) {
        const messageData = {
          text: this.newMessage.trim(),
          recipientEmail: this.selectedFriend.email,
          userEmail: this.currentUser.email
        };

        this.socket.emit('message', messageData, (response) => {
          if (response && response.success) {
            console.log('Message sent successfully');
          } else {
            console.error('Failed to send message');
          }
        });

        this.messages.push({
          text: this.newMessage.trim(),
          user: { email: this.currentUser.email, username: this.currentUser.username }
        });
        this.newMessage = ''; // Reset input field
        console.log('Message sent:', messageData);
      } else {
        console.error('Message is empty, WebSocket is not open, or recipient is not selected.');
      }
    }
  },
  beforeUnmount() {
    if (this.socket) {
      this.socket.close();
    }
  }
};
</script>


<style scoped>
.chat-container {
  max-width: 600px;
  margin: 40px auto; 
  padding: 20px;
  background: #f0f0f0; 
  border-radius: 10px;
}

.chat-header {
  background-color: #007BFF;
  color: white;
  padding: 10px;
  border-radius: 4px 4px 0 0;
}

.message-container {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 20px;
  max-height: 300px;
  overflow-y: auto;
}

.message {
  margin-bottom: 10px;
  display: flex;
  justify-content: flex-start;
}

.from-friend {
  justify-content: flex-end;
}

.message-content {
  display: flex;
  align-items: center;
}

.user-avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  margin: 0 8px;
}

.username {
  font-weight: bold;
  margin-right: 4px;
}

.align-left {
  text-align: left;
}

.align-right {
  text-align: right;
}

.message-text {
  background: #e1ffc7;
  padding: 8px 12px;
  border-radius: 20px;
  max-width: 200px;
  word-wrap: break-word;
}

.from-friend .message-text {
  background: #f1f1f1;
}

.message-input {
  padding: 12px; 
  margin-top: 10px;
  border: none;
  border-radius: 8px;
  box-shadow: inset 2px 2px 5px #bababe, inset -5px -5px 10px #fff; 
  transition: all 0.3s ease; 
}

.message-input:focus {
  box-shadow: inset 1px 1px 2px #bababe, inset -1px -1px 2px #fff; 
}

.send-button {
  padding: 12px 24px; 
  background-color: #007BFF;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  margin-top: 20px;
  transition: background-color 0.3s, transform 0.3s ease;
}

.send-button:hover {
  background-color: #0056b3; 
  transform: translateY(-2px); 
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); 
}

@media only screen and (max-width: 768px) {
  .chat-container {
    max-width: 90%; 
  }
}
</style>

