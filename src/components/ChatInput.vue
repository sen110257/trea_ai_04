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
          <div class="voice-icon-wrapper">
            <el-icon><Microphone /></el-icon>
          </div>
          <span>{{ recording ? '松开结束' : '按住说话' }}</span>
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
          <div class="more-icon" style="background: linear-gradient(135deg, #f97316 0%, #ea580c 100%);">
            <el-icon><Picture /></el-icon>
          </div>
          <span>图片</span>
        </div>
        <div class="more-item" @click="selectVideo">
          <div class="more-icon" style="background: linear-gradient(135deg, #ec4899 0%, #db2777 100%);">
            <el-icon><VideoCamera /></el-icon>
          </div>
          <span>视频</span>
        </div>
        <div class="more-item" @click="startVoiceToText">
          <div class="more-icon" style="background: linear-gradient(135deg, #a855f7 0%, #9333ea 100%);">
            <el-icon><Document /></el-icon>
          </div>
          <span>语音转文字</span>
        </div>
      </div>
    </div>
    
    <div class="recording-overlay" v-show="recording">
      <div class="recording-content">
        <div class="recording-icon">
          <div class="pulse-ring"></div>
          <div class="pulse-ring pulse-ring-2"></div>
          <div class="mic-icon">
            <el-icon><Microphone /></el-icon>
          </div>
        </div>
        <div class="recording-time">
          <span class="time-value">{{ recordingTime }}</span>
          <span class="time-unit">''</span>
        </div>
        <div class="recording-wave">
          <span v-for="i in 8" :key="i" class="recording-bar"></span>
        </div>
        <div class="recording-hint">
          <span class="hint-text">上滑取消</span>
          <span class="hint-arrow">↑</span>
        </div>
      </div>
    </div>
    
    <div class="voice-to-text-overlay" v-show="showVoiceToText">
      <div class="voice-to-text-content">
        <div class="voice-to-text-header">
          <div class="header-icon">
            <el-icon><Microphone /></el-icon>
          </div>
          <span class="header-title">语音转文字</span>
          <div class="header-close" @click="cancelVoiceToText">
            <el-icon><Close /></el-icon>
          </div>
        </div>
        <div class="voice-to-text-body">
          <div class="voice-wave-container" v-if="!voiceToTextResult">
            <div class="voice-wave-outer">
              <span v-for="i in 8" :key="i" class="wave-item"></span>
            </div>
            <div class="wave-text">正在识别...</div>
          </div>
          <div class="voice-result" v-else>
            <div class="result-text">{{ voiceToTextResult }}</div>
          </div>
        </div>
        <div class="voice-to-text-actions">
          <el-button class="cancel-btn" @click="cancelVoiceToText">取消</el-button>
          <el-button class="send-btn" type="primary" @click="sendVoiceToText" :disabled="!voiceToTextResult">发送</el-button>
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
import { ref, onUnmounted } from 'vue'
import { Microphone, Edit, ChatRound, Plus, Picture, VideoCamera, Document, Close } from '@element-plus/icons-vue'
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
    avatar: 'Me',
    isNew: true,
    status: 'success'
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
      avatar: 'Me',
      isNew: true,
      status: 'success'
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
      avatar: 'Me',
      isNew: true,
      status: 'success'
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
    text: null,
    isNew: true,
    status: 'success'
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
    avatar: 'Me',
    isNew: true,
    status: 'success'
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
  background: linear-gradient(180deg, #ffffff 0%, #f8fafc 100%);
  border-top: 1px solid rgba(0, 0, 0, 0.05);
}

.input-toolbar {
  display: flex;
  align-items: center;
  padding: 10px 12px;
}

.toolbar-btn {
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 22px;
  color: #64748b;
  cursor: pointer;
  border-radius: 12px;
  background: rgba(99, 102, 241, 0.05);
  transition: all 0.2s ease;
}

.toolbar-btn:active {
  background: rgba(99, 102, 241, 0.15);
  color: #6366f1;
  transform: scale(0.95);
}

.input-wrapper {
  flex: 1;
  margin: 0 10px;
}

.input-wrapper input {
  width: 100%;
  height: 44px;
  padding: 0 16px;
  border: 1px solid rgba(0, 0, 0, 0.08);
  border-radius: 22px;
  font-size: 15px;
  outline: none;
  background: white;
  transition: all 0.3s ease;
}

.input-wrapper input:focus {
  border-color: #6366f1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}

.voice-btn {
  flex: 1;
  margin: 0 10px;
}

.voice-press-btn {
  width: 100%;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 6px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  border-radius: 22px;
  font-size: 14px;
  color: white;
  user-select: none;
  transition: all 0.2s ease;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
}

.voice-press-btn:active {
  transform: scale(0.98);
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.25);
}

.voice-icon-wrapper {
  font-size: 18px;
}

.send-btn {
  padding: 10px 20px;
  background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%);
  color: white;
  border-radius: 22px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s ease;
  box-shadow: 0 4px 12px rgba(34, 197, 94, 0.3);
}

.send-btn:active {
  transform: scale(0.95);
  box-shadow: 0 2px 8px rgba(34, 197, 94, 0.25);
}

.emoji-panel {
  border-top: 1px solid rgba(0, 0, 0, 0.05);
  background: white;
  max-height: 220px;
  overflow-y: auto;
}

.emoji-list {
  display: flex;
  flex-wrap: wrap;
  padding: 12px;
}

.emoji-item {
  width: 12.5%;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 26px;
  cursor: pointer;
  border-radius: 10px;
  transition: all 0.2s ease;
}

.emoji-item:active {
  background: rgba(99, 102, 241, 0.1);
  transform: scale(0.9);
}

.more-panel {
  border-top: 1px solid rgba(0, 0, 0, 0.05);
  background: white;
  padding: 20px;
}

.more-list {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}

.more-item {
  width: calc(33.333% - 11px);
  display: flex;
  flex-direction: column;
  align-items: center;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.more-item:active {
  transform: scale(0.95);
}

.more-icon {
  width: 56px;
  height: 56px;
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 26px;
  color: white;
  margin-bottom: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.more-item span {
  font-size: 12px;
  color: #64748b;
  font-weight: 500;
}

.recording-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(10px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.recording-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 50px;
}

.recording-icon {
  position: relative;
  width: 100px;
  height: 100px;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 20px;
}

.pulse-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 3px solid rgba(99, 102, 241, 0.5);
  border-radius: 50%;
  animation: pulse 1.5s ease-out infinite;
}

.pulse-ring-2 {
  animation-delay: 0.5s;
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 1;
  }
  100% {
    transform: scale(1.4);
    opacity: 0;
  }
}

.mic-icon {
  width: 60px;
  height: 60px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 28px;
  color: white;
  box-shadow: 0 6px 20px rgba(99, 102, 241, 0.4);
  z-index: 1;
}

.recording-time {
  display: flex;
  align-items: baseline;
  gap: 2px;
  margin-bottom: 16px;
}

.time-value {
  font-size: 48px;
  font-weight: 700;
  color: white;
}

.time-unit {
  font-size: 20px;
  color: rgba(255, 255, 255, 0.7);
}

.recording-wave {
  display: flex;
  align-items: center;
  gap: 4px;
  margin-bottom: 24px;
}

.recording-bar {
  width: 4px;
  height: 24px;
  background: rgba(99, 102, 241, 0.8);
  border-radius: 2px;
  animation: wave 0.5s ease-in-out infinite;
}

.recording-bar:nth-child(1) { animation-delay: 0s; }
.recording-bar:nth-child(2) { animation-delay: 0.08s; }
.recording-bar:nth-child(3) { animation-delay: 0.16s; }
.recording-bar:nth-child(4) { animation-delay: 0.24s; }
.recording-bar:nth-child(5) { animation-delay: 0.32s; }
.recording-bar:nth-child(6) { animation-delay: 0.4s; }
.recording-bar:nth-child(7) { animation-delay: 0.48s; }
.recording-bar:nth-child(8) { animation-delay: 0.56s; }

@keyframes wave {
  0%, 100% { height: 16px; }
  50% { height: 32px; }
}

.recording-hint {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.hint-text {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
}

.hint-arrow {
  font-size: 20px;
  color: rgba(255, 255, 255, 0.5);
  animation: arrow-up 1s ease-in-out infinite;
}

@keyframes arrow-up {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}

.voice-to-text-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.voice-to-text-content {
  width: 85%;
  max-width: 340px;
  background: white;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
  animation: slide-up 0.3s ease;
}

@keyframes slide-up {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.voice-to-text-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
}

.header-icon {
  font-size: 20px;
  color: white;
}

.header-title {
  font-size: 16px;
  font-weight: 600;
  color: white;
}

.header-close {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.header-close:active {
  background: rgba(255, 255, 255, 0.25);
  transform: scale(0.95);
}

.header-close .el-icon {
  font-size: 16px;
  color: white;
}

.voice-to-text-body {
  padding: 20px;
  min-height: 120px;
}

.voice-wave-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100px;
}

.voice-wave-outer {
  display: flex;
  align-items: center;
  gap: 3px;
  margin-bottom: 12px;
}

.wave-item {
  width: 4px;
  height: 24px;
  background: linear-gradient(180deg, #6366f1 0%, #8b5cf6 100%);
  border-radius: 2px;
  animation: wave 0.5s ease-in-out infinite;
}

.wave-item:nth-child(1) { animation-delay: 0s; }
.wave-item:nth-child(2) { animation-delay: 0.08s; }
.wave-item:nth-child(3) { animation-delay: 0.16s; }
.wave-item:nth-child(4) { animation-delay: 0.24s; }
.wave-item:nth-child(5) { animation-delay: 0.32s; }
.wave-item:nth-child(6) { animation-delay: 0.4s; }
.wave-item:nth-child(7) { animation-delay: 0.48s; }
.wave-item:nth-child(8) { animation-delay: 0.56s; }

.wave-text {
  font-size: 14px;
  color: #94a3b8;
}

.voice-result {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100px;
}

.result-text {
  font-size: 15px;
  line-height: 1.8;
  color: #1e293b;
  text-align: center;
}

.voice-to-text-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  padding: 0 20px 20px;
}

.cancel-btn {
  padding: 10px 24px;
  border-radius: 10px;
  font-size: 14px;
  color: #64748b;
  background: #f1f5f9;
  border: none;
}

.send-btn {
  padding: 10px 24px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  background: linear-gradient(135deg, #22c55e 0%, #16a34a 100%);
  border: none;
}

.send-btn:disabled {
  background: #94a3b8;
  opacity: 0.5;
}
</style>
