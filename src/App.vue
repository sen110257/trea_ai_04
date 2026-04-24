<script setup>
import { ref, watch } from 'vue'
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
    { id: 1, sender: 'other', type: 'text', content: '你好，最近怎么样？', time: '10:30', avatar: friend.avatar, isNew: false, status: 'success' },
    { id: 2, sender: 'me', type: 'text', content: '我很好，你呢？', time: '10:31', avatar: 'Me', isNew: false, status: 'success' },
    { id: 3, sender: 'other', type: 'text', content: '我也很好，最近工作忙吗？', time: '10:32', avatar: friend.avatar, isNew: false, status: 'success' },
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
        avatar: currentFriend.value?.avatar || 'A',
        isNew: true,
        status: 'success'
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

const handleRetryMessage = (msg) => {
  console.log('重试消息:', msg)
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
        @retry-message="handleRetryMessage"
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
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app-container {
  height: 100%;
  width: 100%;
  max-width: 480px;
  margin: 0 auto;
  background: #ffffff;
  position: relative;
  overflow: hidden;
  box-shadow: 0 0 40px rgba(0, 0, 0, 0.08);
}

.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding-top: 56px;
}

::-webkit-scrollbar {
  width: 4px;
  height: 4px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: rgba(99, 102, 241, 0.2);
  border-radius: 2px;
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(99, 102, 241, 0.4);
}

::selection {
  background: rgba(99, 102, 241, 0.15);
  color: #1e293b;
}
</style>
