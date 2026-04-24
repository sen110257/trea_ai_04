<template>
  <div class="chat-header">
    <div class="header-left" @click="$emit('back')">
      <el-icon><ArrowLeft /></el-icon>
    </div>
    <div class="header-title">
      <span>{{ currentFriend?.name || '聊天室' }}</span>
    </div>
    <div class="header-right" @click="$emit('more')">
      <el-icon><MoreFilled /></el-icon>
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
  height: 50px;
  padding: 0 15px;
  background-color: #409eff;
  color: white;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
}

.header-left, .header-right {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.header-title {
  flex: 1;
  text-align: center;
  font-size: 18px;
  font-weight: 500;
}
</style>
