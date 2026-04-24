<template>
  <div class="chat-message-list" ref="messageListRef" @scroll="handleScroll">
    <div v-if="messages.length === 0" class="no-messages">
      <span>暂无消息</span>
    </div>
    <div 
      v-for="msg in messages" 
      :key="msg.id" 
      class="message-item"
      :class="{ 'message-me': msg.sender === 'me' }"
    >
      <div class="message-avatar">
        <div class="avatar">{{ msg.avatar }}</div>
      </div>
      <div class="message-content">
        <div class="message-time">{{ msg.time }}</div>
        <div class="message-bubble" :class="getBubbleClass(msg)">
          <div v-if="msg.type === 'text'" class="message-text">
            {{ msg.content }}
          </div>
          <div v-else-if="msg.type === 'emoji'" class="message-emoji">
            {{ msg.content }}
          </div>
          <div v-else-if="msg.type === 'image'" class="message-image">
            <img :src="msg.content" alt="图片" @click="previewImage(msg.content)" />
          </div>
          <div v-else-if="msg.type === 'video'" class="message-video">
            <video :src="msg.content" controls :poster="msg.poster" class="video-player"></video>
          </div>
          <div v-else-if="msg.type === 'voice'" class="message-voice" @click="playVoice(msg)">
            <div class="voice-icon">
              <el-icon v-if="msg.isPlaying"><VideoPause /></el-icon>
              <el-icon v-else><VideoPlay /></el-icon>
            </div>
            <div class="voice-info">
              <div class="voice-wave">
                <span v-for="i in 5" :key="i" class="wave-bar" :class="{ 'playing': msg.isPlaying }"></span>
              </div>
              <div class="voice-duration">{{ msg.duration }}''</div>
            </div>
            <div v-if="msg.text" class="voice-text">{{ msg.text }}</div>
          </div>
        </div>
      </div>
    </div>
    <div ref="scrollBottomRef"></div>
  </div>
</template>

<script setup>
import { ref, nextTick, watch } from 'vue'
import { VideoPlay, VideoPause } from '@element-plus/icons-vue'

const props = defineProps({
  messages: {
    type: Array,
    default: () => []
  }
})

const messageListRef = ref(null)
const scrollBottomRef = ref(null)
const playingMessageId = ref(null)

watch(() => props.messages.length, () => {
  nextTick(() => {
    scrollToBottom()
  })
}, { deep: true })

const scrollToBottom = () => {
  if (scrollBottomRef.value) {
    scrollBottomRef.value.scrollIntoView({ behavior: 'smooth' })
  }
}

const handleScroll = (e) => {
  const { scrollTop, scrollHeight, clientHeight } = e.target
  if (scrollTop === 0) {
    console.log('加载更多历史消息')
  }
}

const getBubbleClass = (msg) => {
  const classes = ['bubble-' + msg.type]
  if (msg.sender === 'me') {
    classes.push('bubble-me')
  }
  return classes
}

const previewImage = (src) => {
  console.log('预览图片:', src)
}

const playVoice = (msg) => {
  if (playingMessageId.value === msg.id) {
    playingMessageId.value = null
    msg.isPlaying = false
  } else {
    if (playingMessageId.value) {
      const prevMsg = props.messages.find(m => m.id === playingMessageId.value)
      if (prevMsg) prevMsg.isPlaying = false
    }
    playingMessageId.value = msg.id
    msg.isPlaying = true
    
    setTimeout(() => {
      if (playingMessageId.value === msg.id) {
        playingMessageId.value = null
        msg.isPlaying = false
      }
    }, msg.duration * 1000)
  }
}

defineExpose({
  scrollToBottom
})
</script>

<style scoped>
.chat-message-list {
  flex: 1;
  overflow-y: auto;
  padding: 15px;
  background-color: #f5f5f5;
}

.no-messages {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #999;
}

.message-item {
  display: flex;
  margin-bottom: 15px;
}

.message-me {
  flex-direction: row-reverse;
}

.message-avatar {
  flex-shrink: 0;
  margin: 0 10px;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #409eff;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  font-weight: bold;
}

.message-me .avatar {
  background-color: #67c23a;
}

.message-content {
  max-width: 70%;
  display: flex;
  flex-direction: column;
}

.message-time {
  font-size: 12px;
  color: #999;
  margin-bottom: 5px;
  text-align: center;
}

.message-bubble {
  padding: 10px 15px;
  border-radius: 8px;
  background-color: white;
  position: relative;
  word-break: break-all;
}

.message-me .message-bubble {
  background-color: #95ec69;
}

.bubble-text, .bubble-emoji {
  min-height: 20px;
}

.message-emoji {
  font-size: 32px;
  padding: 5px;
}

.message-image img {
  max-width: 200px;
  max-height: 200px;
  border-radius: 5px;
  cursor: pointer;
}

.message-video .video-player {
  max-width: 200px;
  max-height: 200px;
  border-radius: 5px;
}

.message-voice {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 10px 15px;
  min-width: 80px;
}

.voice-icon {
  margin-right: 10px;
  font-size: 20px;
}

.voice-info {
  display: flex;
  align-items: center;
  flex: 1;
}

.voice-wave {
  display: flex;
  align-items: center;
  margin-right: 10px;
}

.wave-bar {
  width: 3px;
  height: 15px;
  background-color: #999;
  margin: 0 1px;
  border-radius: 2px;
  animation: none;
}

.wave-bar.playing {
  animation: wave 0.5s ease-in-out infinite;
}

.wave-bar:nth-child(1) { animation-delay: 0s; }
.wave-bar:nth-child(2) { animation-delay: 0.1s; }
.wave-bar:nth-child(3) { animation-delay: 0.2s; }
.wave-bar:nth-child(4) { animation-delay: 0.3s; }
.wave-bar:nth-child(5) { animation-delay: 0.4s; }

@keyframes wave {
  0%, 100% { height: 10px; }
  50% { height: 20px; }
}

.voice-duration {
  font-size: 12px;
  color: #666;
  min-width: 30px;
}

.voice-text {
  font-size: 12px;
  color: #666;
  margin-top: 5px;
  padding-top: 5px;
  border-top: 1px solid #eee;
}
</style>
