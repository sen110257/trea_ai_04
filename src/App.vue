<script setup>
import { ref, computed, watch } from 'vue'
import ChatHeader from './components/ChatHeader.vue'
import ChatMessageList from './components/ChatMessageList.vue'
import ChatInput from './components/ChatInput.vue'
import FriendList from './components/FriendList.vue'
import { chatMessages } from './data/mockData.js'

const currentView = ref('friendList')
const currentFriend = ref(null)
const messages = ref([...chatMessages])

const messageListRef = ref(null)

const goToChat = (friend) => {
  currentFriend.value = friend
  currentView.value = 'chat'
  messages.value = [
    { id: 1, sender: 'other', type: 'text', content: '你好，最近怎么样？', time: '10:30', avatar: friend.avatar },
    { id: 2, sender: 'me', type: 'text', content: '我很好，你呢？', time: '10:31', avatar: 'Me' },
    { id: 3, sender: 'other', type: 'text', content: '我也很好，最近工作忙吗？', time: '10:32', avatar: friend.avatar },
  ]
}

const goBack = () => {
  currentView.value = 'friendList'
  currentFriend.value = null
}

const handleSendMessage = (message) => {
  messages.value.push(message)
  
  if (message.sender === 'me' && message.type === 'text') {
    setTimeout(() => {
      const replies = [
        '好的，我知道了。',
        '收到，谢谢！',
        '没问题。',
        '好的，稍后聊。',
        '嗯嗯，好的。',
        '我看看。',
        '明白了。'
      ]
      const reply = {
        id: Date.now(),
        sender: 'other',
        type: 'text',
        content: replies[Math.floor(Math.random() * replies.length)],
        time: getCurrentTime(),
        avatar: currentFriend.value?.avatar || 'A'
      }
      messages.value.push(reply)
    }, 1000 + Math.random() * 1500)
  }
}

const getCurrentTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  return `${hours}:${minutes}`
}

const handleInputFocus = () => {
  setTimeout(() => {
    messageListRef.value?.scrollToBottom?.()
  }, 100)
}
</script>

<template>
  <div class="app-container">
    <FriendList 
      v-if="currentView === 'friendList'" 
      @select-friend="goToChat"
    />
    
    <div v-else-if="currentView === 'chat'" class="chat-container">
      <ChatHeader 
        :current-friend="currentFriend" 
        @back="goBack"
        @more="() => console.log('更多操作')"
      />
      <ChatMessageList 
        ref="messageListRef"
        :messages="messages" 
      />
      <ChatInput 
        @send-message="handleSendMessage"
        @focus="handleInputFocus"
      />
    </div>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  height: 100%;
  width: 100%;
  overflow: hidden;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  background-color: #f5f5f5;
}

.app-container {
  height: 100%;
  width: 100%;
  max-width: 480px;
  margin: 0 auto;
  background-color: #fff;
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding-top: 50px;
}
</style>
