<template>
  <div class="chat-header">
    <div class="header-left" @click="$emit('back')">
      <div class="back-btn">
        <el-icon><ArrowLeft /></el-icon>
      </div>
    </div>
    <div class="header-title">
      <div class="friend-info">
        <div class="friend-avatar-mini" v-if="currentFriend">
          {{ currentFriend.avatar }}
        </div>
        <div class="friend-details">
          <span class="friend-name">{{ currentFriend?.name || '聊天室' }}</span>
          <span class="friend-status" v-if="currentFriend">在线</span>
        </div>
      </div>
    </div>
    <div class="header-right" @click="$emit('more')">
      <div class="more-btn">
        <el-icon><MoreFilled /></el-icon>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ArrowLeft, MoreFilled } from '@element-plus/icons-vue'

defineProps({
  currentFriend: {
    type: Object,
    default: null
  }
})

defineEmits(['back', 'more'])
</script>

<style scoped>
.chat-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  height: 56px;
  padding: 0 12px;
  background: linear-gradient(135deg, #ffffff 0%, #f8fafc 100%);
  color: #1e293b;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
}

.header-left, .header-right {
  width: 44px;
  height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.back-btn, .more-btn {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(99, 102, 241, 0.05);
  border-radius: 12px;
  transition: all 0.2s ease;
}

.back-btn:active, .more-btn:active {
  background: rgba(99, 102, 241, 0.1);
  transform: scale(0.95);
}

.back-btn .el-icon, .more-btn .el-icon {
  font-size: 20px;
  color: #64748b;
  transition: color 0.2s ease;
}

.back-btn:active .el-icon, .more-btn:active .el-icon {
  color: #6366f1;
}

.header-title {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
}

.friend-info {
  display: flex;
  align-items: center;
  gap: 10px;
}

.friend-avatar-mini {
  width: 36px;
  height: 36px;
  border-radius: 10px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: 700;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.3);
}

.friend-details {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.friend-name {
  font-size: 16px;
  font-weight: 600;
  color: #1e293b;
  letter-spacing: 0.3px;
}

.friend-status {
  font-size: 11px;
  color: #22c55e;
  display: flex;
  align-items: center;
  gap: 4px;
}

.friend-status::before {
  content: '';
  width: 6px;
  height: 6px;
  background: #22c55e;
  border-radius: 50%;
  animation: status-pulse 2s ease-in-out infinite;
}

@keyframes status-pulse {
  0%, 100% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.7;
    transform: scale(1.1);
  }
}
</style>
