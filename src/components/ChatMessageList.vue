<template>
  <div class="chat-message-list" ref="messageListRef" @scroll="handleScroll">
    <div v-if="messages.length === 0" class="no-messages">
      <div class="empty-icon">
        <el-icon><ChatRound /></el-icon>
      </div>
      <div class="empty-text">暂无消息</div>
      <div class="empty-hint">发送一条消息开始聊天吧</div>
    </div>
    
    <div 
      v-for="(msg, index) in messages" 
      :key="msg.id" 
      class="message-item"
      :class="{ 
        'message-me': msg.sender === 'me',
        'message-animate': msg.isNew,
        'message-loading': msg.status === 'loading',
        'message-failed': msg.status === 'failed'
      }"
    >
      <div class="message-avatar">
        <div class="avatar" :class="{ 'avatar-me': msg.sender === 'me' }">
          {{ msg.avatar }}
        </div>
      </div>
      
      <div class="message-content">
        <div class="message-time">{{ msg.time }}</div>
        <div class="message-bubble" :class="getBubbleClass(msg)" @click="handleBubbleClick(msg)">
          <div v-if="msg.type === 'text'" class="message-text">
            {{ msg.content }}
          </div>
          <div v-else-if="msg.type === 'emoji'" class="message-emoji">
            {{ msg.content }}
          </div>
          <div v-else-if="msg.type === 'image'" class="message-image">
            <div class="image-wrapper" @click.stop="previewImage(msg.content, msg)">
              <img :src="msg.content" alt="图片" />
              <div class="image-overlay">
                <el-icon><ZoomIn /></el-icon>
              </div>
            </div>
          </div>
          <div v-else-if="msg.type === 'video'" class="message-video">
            <div class="video-wrapper" @click.stop="previewVideo(msg)">
              <video :src="msg.content" :poster="msg.poster" class="video-player" controls></video>
            </div>
          </div>
          <div v-else-if="msg.type === 'voice'" class="message-voice" @click="playVoice(msg)">
            <div class="voice-icon">
              <el-icon v-if="msg.isPlaying"><VideoPause /></el-icon>
              <el-icon v-else><VideoPlay /></el-icon>
            </div>
            <div class="voice-info">
              <div class="voice-progress">
                <div class="progress-bar" :style="{ width: getVoiceProgress(msg) + '%' }"></div>
              </div>
              <div class="voice-wave">
                <span 
                  v-for="i in 5" 
                  :key="i" 
                  class="wave-bar" 
                  :class="{ 'playing': msg.isPlaying }"
                ></span>
              </div>
            </div>
            <div class="voice-duration">{{ msg.duration }}''</div>
            <div v-if="msg.text" class="voice-text">{{ msg.text }}</div>
          </div>
          
          <div v-if="msg.status === 'loading'" class="message-status loading-status">
            <div class="loading-spinner"></div>
          </div>
          <div v-else-if="msg.status === 'failed'" class="message-status failed-status" @click.stop="retryMessage(msg)">
            <el-icon><Warning /></el-icon>
          </div>
        </div>
      </div>
    </div>
    
    <div ref="scrollBottomRef"></div>
    
    <div class="preview-overlay" v-if="showPreview" @click="closePreview">
      <div class="preview-content" @click.stop>
        <img v-if="previewType === 'image'" :src="previewSrc" class="preview-image" />
        <video v-else-if="previewType === 'video'" :src="previewSrc" controls class="preview-video"></video>
        <div class="preview-close" @click="closePreview">
          <el-icon><Close /></el-icon>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, watch, computed } from 'vue'
import { VideoPlay, VideoPause, ChatRound, ZoomIn, Warning, Close } from '@element-plus/icons-vue'

const props = defineProps({
  messages: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['retryMessage'])

const messageListRef = ref(null)
const scrollBottomRef = ref(null)
const playingMessageId = ref(null)
const playingMessage = ref(null)
const playingStartTime = ref(0)

const showPreview = ref(false)
const previewType = ref('image')
const previewSrc = ref('')
const currentPreviewMessage = ref(null)

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

const getVoiceProgress = (msg) => {
  if (!msg.isPlaying || !playingMessage.value) return 0
  const elapsed = (Date.now() - playingStartTime.value) / 1000
  const progress = (elapsed / msg.duration) * 100
  return Math.min(progress, 100)
}

const previewImage = (src, msg) => {
  previewType.value = 'image'
  previewSrc.value = src
  currentPreviewMessage.value = msg
  showPreview.value = true
}

const previewVideo = (msg) => {
  previewType.value = 'video'
  previewSrc.value = msg.content
  currentPreviewMessage.value = msg
  showPreview.value = true
}

const closePreview = () => {
  showPreview.value = false
  previewSrc.value = ''
  currentPreviewMessage.value = null
}

const handleBubbleClick = (msg) => {
  if (msg.type === 'text' || msg.type === 'emoji') {
    console.log('点击了消息:', msg)
  }
}

const playVoice = (msg) => {
  if (playingMessageId.value === msg.id) {
    playingMessageId.value = null
    playingMessage.value = null
    msg.isPlaying = false
  } else {
    if (playingMessageId.value) {
      const prevMsg = props.messages.find(m => m.id === playingMessageId.value)
      if (prevMsg) prevMsg.isPlaying = false
    }
    playingMessageId.value = msg.id
    playingMessage.value = msg
    playingStartTime.value = Date.now()
    msg.isPlaying = true
    
    setTimeout(() => {
      if (playingMessageId.value === msg.id) {
        playingMessageId.value = null
        playingMessage.value = null
        msg.isPlaying = false
      }
    }, msg.duration * 1000)
  }
}

const retryMessage = (msg) => {
  emit('retryMessage', msg)
}

defineExpose({
  scrollToBottom
})
</script>

<style scoped>
.chat-message-list {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
  background: linear-gradient(180deg, #f8fafc 0%, #f1f5f9 100%);
}

.no-messages {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  color: #94a3b8;
  padding: 40px;
}

.empty-icon {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, rgba(99, 102, 241, 0.1) 0%, rgba(139, 92, 246, 0.1) 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 16px;
}

.empty-icon .el-icon {
  font-size: 36px;
  color: #6366f1;
}

.empty-text {
  font-size: 18px;
  font-weight: 600;
  color: #64748b;
  margin-bottom: 8px;
}

.empty-hint {
  font-size: 14px;
  color: #94a3b8;
}

.message-item {
  display: flex;
  margin-bottom: 20px;
  opacity: 1;
  transform: translateY(0);
  transition: all 0.3s ease;
}

.message-me {
  flex-direction: row-reverse;
}

.message-animate {
  animation: message-in 0.4s ease;
}

@keyframes message-in {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.message-loading {
  opacity: 0.7;
}

.message-loading .message-bubble {
  position: relative;
}

.message-failed .message-bubble {
  border: 1px dashed #ef4444;
}

.message-avatar {
  flex-shrink: 0;
  margin: 0 10px;
}

.avatar {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  font-weight: 700;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
  transition: transform 0.2s ease;
}

.avatar-me {
  background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%);
  box-shadow: 0 4px 12px rgba(34, 197, 94, 0.3);
}

.message-content {
  max-width: 70%;
  display: flex;
  flex-direction: column;
}

.message-time {
  font-size: 11px;
  color: #94a3b8;
  margin-bottom: 6px;
  text-align: center;
  font-weight: 500;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 16px;
  background: #ffffff;
  position: relative;
  word-break: break-all;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
  border: 1px solid rgba(0, 0, 0, 0.03);
}

.message-bubble:active {
  transform: scale(0.98);
}

.message-me .message-bubble {
  background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%);
  color: white;
  box-shadow: 0 4px 12px rgba(34, 197, 94, 0.25);
}

.bubble-text, .bubble-emoji {
  min-height: 20px;
}

.message-text {
  font-size: 15px;
  line-height: 1.6;
  letter-spacing: 0.3px;
  color: #1e293b;
}

.message-me .message-text {
  color: white;
}

.message-emoji {
  font-size: 36px;
  padding: 8px;
  background: transparent;
  box-shadow: none;
  border: none;
}

.message-image .image-wrapper {
  position: relative;
  cursor: pointer;
  border-radius: 12px;
  overflow: hidden;
}

.message-image img {
  max-width: 200px;
  max-height: 200px;
  border-radius: 12px;
  display: block;
  transition: transform 0.3s ease;
}

.image-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.image-overlay .el-icon {
  font-size: 28px;
  color: white;
}

.image-wrapper:active .image-overlay {
  opacity: 1;
}

.image-wrapper:active img {
  transform: scale(1.02);
}

.message-video .video-wrapper {
  border-radius: 12px;
  overflow: hidden;
}

.message-video .video-player {
  max-width: 200px;
  max-height: 200px;
  border-radius: 12px;
  display: block;
}

.message-voice {
  display: flex;
  align-items: center;
  cursor: pointer;
  padding: 12px 16px;
  min-width: 100px;
  gap: 10px;
}

.voice-icon {
  font-size: 20px;
  color: #6366f1;
}

.message-me .voice-icon {
  color: white;
}

.voice-info {
  display: flex;
  flex-direction: column;
  flex: 1;
  gap: 6px;
}

.voice-progress {
  height: 4px;
  background: rgba(99, 102, 241, 0.2);
  border-radius: 2px;
  overflow: hidden;
}

.message-me .voice-progress {
  background: rgba(255, 255, 255, 0.3);
}

.progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #6366f1 0%, #8b5cf6 100%);
  border-radius: 2px;
  transition: width 0.1s linear;
}

.message-me .progress-bar {
  background: white;
}

.voice-wave {
  display: flex;
  align-items: center;
  gap: 3px;
}

.wave-bar {
  width: 3px;
  height: 12px;
  background: rgba(99, 102, 241, 0.5);
  border-radius: 2px;
  transition: all 0.2s ease;
}

.message-me .wave-bar {
  background: rgba(255, 255, 255, 0.5);
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
  0%, 100% { height: 8px; background: rgba(99, 102, 241, 0.3); }
  50% { height: 18px; background: rgba(99, 102, 241, 0.8); }
}

.message-me .wave-bar.playing {
  animation: wave-me 0.5s ease-in-out infinite;
}

@keyframes wave-me {
  0%, 100% { height: 8px; background: rgba(255, 255, 255, 0.3); }
  50% { height: 18px; background: rgba(255, 255, 255, 0.8); }
}

.voice-duration {
  font-size: 12px;
  color: #64748b;
  font-weight: 600;
  min-width: 30px;
}

.message-me .voice-duration {
  color: rgba(255, 255, 255, 0.9);
}

.voice-text {
  font-size: 12px;
  color: #64748b;
  margin-top: 8px;
  padding-top: 8px;
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

.message-me .voice-text {
  color: rgba(255, 255, 255, 0.8);
  border-top-color: rgba(255, 255, 255, 0.2);
}

.message-status {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
}

.loading-status {
  right: -24px;
}

.loading-spinner {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(99, 102, 241, 0.2);
  border-top-color: #6366f1;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.failed-status {
  right: -28px;
  width: 20px;
  height: 20px;
  background: #ef4444;
  border-radius: 50%;
  cursor: pointer;
}

.failed-status .el-icon {
  font-size: 12px;
  color: white;
}

.preview-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fade-in 0.3s ease;
}

@keyframes fade-in {
  from { opacity: 0; }
  to { opacity: 1; }
}

.preview-content {
  position: relative;
  max-width: 90%;
  max-height: 90%;
}

.preview-image {
  max-width: 100%;
  max-height: 80vh;
  border-radius: 8px;
}

.preview-video {
  max-width: 100%;
  max-height: 80vh;
  border-radius: 8px;
}

.preview-close {
  position: absolute;
  top: -40px;
  right: 0;
  width: 36px;
  height: 36px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.2s ease;
}

.preview-close:active {
  background: rgba(255, 255, 255, 0.2);
}

.preview-close .el-icon {
  font-size: 20px;
  color: white;
}
</style>
