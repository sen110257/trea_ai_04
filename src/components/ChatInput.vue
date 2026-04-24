<template>
  <div class="chat-input">
    <div class="input-toolbar">
      <div class="toolbar-btn" @click="toggleVoiceMode">
        <el-icon><Microphone v-if="!isVoiceMode" /><Edit v-else /></el-icon>
      </div>
      <div class="input-wrapper" v-show="!isVoiceMode">
        <input 
          v-model="inputText" 
          type="text" 
          placeholder="输入消息..." 
          @keyup.enter="sendTextMessage"
          @focus="onInputFocus"
        />
      </div>
      <div class="voice-btn" v-show="isVoiceMode">
        <div 
          class="voice-press-btn" 
          @touchstart="startRecord" 
          @touchend="endRecord"
          @mousedown="startRecord"
          @mouseup="endRecord"
          @mouseleave="cancelRecord"
        >
          {{ recording ? '松开结束' : '按住说话' }}
        </div>
      </div>
      <div class="toolbar-btn" @click="toggleEmojiPanel" v-show="!isVoiceMode">
        <el-icon><ChatRound /></el-icon>
      </div>
      <div class="toolbar-btn" @click="toggleMorePanel" v-show="!isVoiceMode && !inputText">
        <el-icon><Plus /></el-icon>
      </div>
      <div class="send-btn" v-show="inputText && !isVoiceMode" @click="sendTextMessage">
        发送
      </div>
    </div>
    
    <div class="emoji-panel" v-show="showEmojiPanel">
      <div class="emoji-list">
        <div 
          v-for="(emoji, index) in emojiList" 
          :key="index" 
          class="emoji-item"
          @click="selectEmoji(emoji)"
        >
          {{ emoji }}
        </div>
      </div>
    </div>
    
    <div class="more-panel" v-show="showMorePanel">
      <div class="more-list">
        <div class="more-item" @click="selectImage">
          <div class="more-icon" style="background-color: #ff9800;">
            <el-icon><Picture /></el-icon>
          </div>
          <span>图片</span>
        </div>
        <div class="more-item" @click="selectVideo">
          <div class="more-icon" style="background-color: #e91e63;">
            <el-icon><VideoCamera /></el-icon>
          </div>
          <span>视频</span>
        </div>
        <div class="more-item" @click="startVoiceToText">
          <div class="more-icon" style="background-color: #9c27b0;">
            <el-icon><Document /></el-icon>
          </div>
          <span>语音转文字</span>
        </div>
      </div>
    </div>
    
    <div class="recording-overlay" v-show="recording">
      <div class="recording-content">
        <div class="recording-wave">
          <span v-for="i in 5" :key="i" class="recording-bar"></span>
        </div>
        <div class="recording-time">{{ recordingTime }}''</div>
        <div class="recording-hint">上滑取消</div>
      </div>
    </div>
    
    <div class="voice-to-text-overlay" v-show="showVoiceToText">
      <div class="voice-to-text-content">
        <div class="voice-to-text-wave">
          <span v-for="i in 5" :key="i" class="voice-to-text-bar"></span>
        </div>
        <div class="voice-to-text-text">{{ voiceToTextResult || '正在识别...' }}</div>
        <div class="voice-to-text-actions">
          <el-button size="small" @click="cancelVoiceToText">取消</el-button>
          <el-button type="primary" size="small" @click="sendVoiceToText">发送</el-button>
        </div>
      </div>
    </div>
    
    <input 
      type="file" 
      ref="imageInput" 
      accept="image/*" 
      style="display: none;" 
      @change="onImageSelect"
    />
    <input 
      type="file" 
      ref="videoInput" 
      accept="video/*" 
      style="display: none;" 
      @change="onVideoSelect"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { Microphone, Edit, ChatRound, Plus, Picture, VideoCamera, Document } from '@element-plus/icons-vue'
import { emojis } from '../data/mockData.js'

const emit = defineEmits(['sendMessage', 'focus'])

const inputText = ref('')
const isVoiceMode = ref(false)
const showEmojiPanel = ref(false)
const showMorePanel = ref(false)
const emojiList = ref(emojis)

const imageInput = ref(null)
const videoInput = ref(null)

const recording = ref(false)
const recordingTime = ref(0)
let recordingTimer = null

const showVoiceToText = ref(false)
const voiceToTextResult = ref('')
let voiceToTextTimer = null

const toggleVoiceMode = () => {
  isVoiceMode.value = !isVoiceMode.value
  showEmojiPanel.value = false
  showMorePanel.value = false
}

const toggleEmojiPanel = () => {
  showEmojiPanel.value = !showEmojiPanel.value
  showMorePanel.value = false
  if (showEmojiPanel.value) {
    emit('focus')
  }
}

const toggleMorePanel = () => {
  showMorePanel.value = !showMorePanel.value
  showEmojiPanel.value = false
  if (showMorePanel.value) {
    emit('focus')
  }
}

const onInputFocus = () => {
  showEmojiPanel.value = false
  showMorePanel.value = false
  emit('focus')
}

const selectEmoji = (emoji) => {
  inputText.value += emoji
}

const sendTextMessage = () => {
  if (!inputText.value.trim()) return
  
  const message = {
    id: Date.now(),
    sender: 'me',
    type: 'text',
    content: inputText.value.trim(),
    time: getCurrentTime(),
    avatar: 'Me'
  }
  
  emit('sendMessage', message)
  inputText.value = ''
  showEmojiPanel.value = false
  showMorePanel.value = false
}

const selectImage = () => {
  imageInput.value.click()
}

const onImageSelect = (e) => {
  const file = e.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (event) => {
    const message = {
      id: Date.now(),
      sender: 'me',
      type: 'image',
      content: event.target.result,
      time: getCurrentTime(),
      avatar: 'Me'
    }
    emit('sendMessage', message)
    showMorePanel.value = false
  }
  reader.readAsDataURL(file)
  e.target.value = ''
}

const selectVideo = () => {
  videoInput.value.click()
}

const onVideoSelect = (e) => {
  const file = e.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (event) => {
    const message = {
      id: Date.now(),
      sender: 'me',
      type: 'video',
      content: event.target.result,
      poster: '',
      time: getCurrentTime(),
      avatar: 'Me'
    }
    emit('sendMessage', message)
    showMorePanel.value = false
  }
  reader.readAsDataURL(file)
  e.target.value = ''
}

const startRecord = () => {
  recording.value = true
  recordingTime.value = 0
  recordingTimer = setInterval(() => {
    recordingTime.value++
  }, 1000)
}

const endRecord = () => {
  if (!recording.value) return
  
  clearInterval(recordingTimer)
  recordingTimer = null
  
  if (recordingTime.value < 1) {
    recording.value = false
    return
  }
  
  const message = {
    id: Date.now(),
    sender: 'me',
    type: 'voice',
    content: 'voice_data',
    duration: recordingTime.value,
    time: getCurrentTime(),
    avatar: 'Me',
    isPlaying: false,
    text: null
  }
  
  emit('sendMessage', message)
  recording.value = false
  recordingTime.value = 0
}

const cancelRecord = () => {
  clearInterval(recordingTimer)
  recordingTimer = null
  recording.value = false
  recordingTime.value = 0
}

const startVoiceToText = () => {
  showVoiceToText.value = true
  showMorePanel.value = false
  voiceToTextResult.value = ''
  
  const mockTexts = [
    '你好，我现在有点忙，稍后联系你。',
    '好的，我知道了，明天见。',
    '收到，谢谢！',
    '没问题，就这样吧。',
    '周末一起吃饭怎么样？'
  ]
  
  voiceToTextTimer = setTimeout(() => {
    voiceToTextResult.value = mockTexts[Math.floor(Math.random() * mockTexts.length)]
  }, 2000)
}

const cancelVoiceToText = () => {
  clearTimeout(voiceToTextTimer)
  showVoiceToText.value = false
  voiceToTextResult.value = ''
}

const sendVoiceToText = () => {
  if (!voiceToTextResult.value) return
  
  const message = {
    id: Date.now(),
    sender: 'me',
    type: 'text',
    content: voiceToTextResult.value,
    time: getCurrentTime(),
    avatar: 'Me'
  }
  
  emit('sendMessage', message)
  showVoiceToText.value = false
  voiceToTextResult.value = ''
}

const getCurrentTime = () => {
  const now = new Date()
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  return `${hours}:${minutes}`
}

onUnmounted(() => {
  clearInterval(recordingTimer)
  clearTimeout(voiceToTextTimer)
})
</script>

<style scoped>
.chat-input {
  background-color: #fff;
  border-top: 1px solid #e5e5e5;
}

.input-toolbar {
  display: flex;
  align-items: center;
  padding: 8px 10px;
}

.toolbar-btn {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  color: #666;
  cursor: pointer;
}

.input-wrapper {
  flex: 1;
  margin: 0 8px;
}

.input-wrapper input {
  width: 100%;
  height: 36px;
  padding: 0 10px;
  border: 1px solid #ddd;
  border-radius: 18px;
  font-size: 14px;
  outline: none;
}

.input-wrapper input:focus {
  border-color: #409eff;
}

.voice-btn {
  flex: 1;
  margin: 0 8px;
}

.voice-press-btn {
  width: 100%;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f0f0f0;
  border-radius: 18px;
  font-size: 14px;
  color: #333;
  user-select: none;
}

.voice-press-btn:active {
  background-color: #e0e0e0;
}

.send-btn {
  padding: 6px 15px;
  background-color: #409eff;
  color: white;
  border-radius: 18px;
  font-size: 14px;
  cursor: pointer;
}

.emoji-panel {
  border-top: 1px solid #e5e5e5;
  background-color: #f8f8f8;
  max-height: 200px;
  overflow-y: auto;
}

.emoji-list {
  display: flex;
  flex-wrap: wrap;
  padding: 10px;
}

.emoji-item {
  width: 12.5%;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  cursor: pointer;
}

.emoji-item:active {
  background-color: #e5e5e5;
  border-radius: 4px;
}

.more-panel {
  border-top: 1px solid #e5e5e5;
  background-color: #f8f8f8;
  padding: 15px;
}

.more-list {
  display: flex;
  flex-wrap: wrap;
}

.more-item {
  width: 25%;
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
}

.more-icon {
  width: 50px;
  height: 50px;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  color: white;
  margin-bottom: 5px;
}

.more-item span {
  font-size: 12px;
  color: #666;
}

.recording-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.recording-content {
  background-color: rgba(0, 0, 0, 0.8);
  padding: 30px 40px;
  border-radius: 10px;
  text-align: center;
  color: white;
}

.recording-wave {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 15px;
}

.recording-bar {
  width: 4px;
  height: 30px;
  background-color: #409eff;
  margin: 0 2px;
  border-radius: 2px;
  animation: recording-wave 0.5s ease-in-out infinite;
}

.recording-bar:nth-child(1) { animation-delay: 0s; }
.recording-bar:nth-child(2) { animation-delay: 0.1s; }
.recording-bar:nth-child(3) { animation-delay: 0.2s; }
.recording-bar:nth-child(4) { animation-delay: 0.3s; }
.recording-bar:nth-child(5) { animation-delay: 0.4s; }

@keyframes recording-wave {
  0%, 100% { height: 20px; }
  50% { height: 40px; }
}

.recording-time {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 10px;
}

.recording-hint {
  font-size: 12px;
  color: #ccc;
}

.voice-to-text-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.voice-to-text-content {
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  width: 80%;
  max-width: 300px;
}

.voice-to-text-wave {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 15px;
}

.voice-to-text-bar {
  width: 4px;
  height: 30px;
  background-color: #409eff;
  margin: 0 2px;
  border-radius: 2px;
  animation: recording-wave 0.5s ease-in-out infinite;
}

.voice-to-text-text {
  min-height: 60px;
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 5px;
  font-size: 14px;
  color: #333;
  margin-bottom: 15px;
}

.voice-to-text-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}
</style>
