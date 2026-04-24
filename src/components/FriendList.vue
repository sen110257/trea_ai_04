<template>
  <div class="friend-list">
    <div class="list-header">
      <div class="search-wrapper">
        <el-icon class="search-icon"><Search /></el-icon>
        <input 
          v-model="searchText" 
          type="text" 
          placeholder="搜索联系人" 
          class="search-input"
        />
      </div>
    </div>
    
    <div class="list-content" ref="listContentRef">
      <div class="quick-friends" v-if="quickFriends.length > 0">
        <div class="section-title">
          <span class="title-icon">📌</span>
          常用联系人
        </div>
        <div 
          v-for="friend in quickFriends" 
          :key="friend.id" 
          class="friend-item"
          @click="selectFriend(friend)"
        >
          <div class="friend-avatar">
            <div class="avatar">
              {{ friend.avatar }}
              <span class="unread-dot" v-if="friend.unread > 0"></span>
            </div>
          </div>
          <div class="friend-info">
            <div class="friend-name">
              <span class="name-text">{{ friend.name }}</span>
              <span class="unread-badge" v-if="friend.unread > 0">
                {{ friend.unread > 99 ? '99+' : friend.unread }}
              </span>
            </div>
            <div class="friend-last-message">{{ friend.lastMessage }}</div>
          </div>
          <div class="friend-time">{{ friend.time }}</div>
        </div>
      </div>
      
      <div 
        v-for="(group, letter) in groupedFriends" 
        :key="letter" 
        class="friend-group"
        :id="'group-' + letter"
      >
        <div class="section-title" :id="'title-' + letter">{{ letter }}</div>
        <div 
          v-for="friend in group" 
          :key="friend.id" 
          class="friend-item"
          @click="selectFriend(friend)"
        >
          <div class="friend-avatar">
            <div class="avatar">
              {{ friend.avatar }}
              <span class="unread-dot" v-if="friend.unread > 0"></span>
            </div>
          </div>
          <div class="friend-info">
            <div class="friend-name">
              <span class="name-text">{{ friend.name }}</span>
              <span class="unread-badge" v-if="friend.unread > 0">
                {{ friend.unread > 99 ? '99+' : friend.unread }}
              </span>
            </div>
            <div class="friend-last-message">{{ friend.lastMessage }}</div>
          </div>
          <div class="friend-time">{{ friend.time }}</div>
        </div>
      </div>
    </div>
    
    <div class="index-bar">
      <div 
        v-for="letter in indexLetters" 
        :key="letter" 
        class="index-item"
        :class="{ 'index-active': activeIndex === letter }"
        @click="scrollToGroup(letter)"
        @touchstart="onIndexTouchStart($event, letter)"
        @touchmove="onIndexTouchMove"
        @touchend="onIndexTouchEnd"
      >
        {{ letter }}
      </div>
    </div>
    
    <div class="index-toast" v-if="showIndexToast">
      {{ activeIndex }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { Search } from '@element-plus/icons-vue'
import { friends } from '../data/mockData.js'

const emit = defineEmits(['selectFriend'])

const searchText = ref('')
const listContentRef = ref(null)
const currentTouchLetter = ref(null)
const activeIndex = ref('')
const showIndexToast = ref(false)

const allFriends = ref([...friends])

const sortedFriends = computed(() => {
  return [...allFriends.value].sort((a, b) => {
    return a.name.localeCompare(b.name, 'zh-CN')
  })
})

const filteredFriends = computed(() => {
  if (!searchText.value) return sortedFriends.value
  return sortedFriends.value.filter(friend => 
    friend.name.toLowerCase().includes(searchText.value.toLowerCase())
  )
})

const quickFriends = computed(() => {
  return filteredFriends.value.filter(friend => friend.unread > 0)
    .sort((a, b) => b.unread - a.unread)
})

const groupedFriends = computed(() => {
  const groups = {}
  filteredFriends.value.forEach(friend => {
    const firstChar = friend.name[0].toUpperCase()
    let letter
    if (/^[A-Z]$/.test(firstChar)) {
      letter = firstChar
    } else {
      letter = '#'
    }
    
    if (!groups[letter]) {
      groups[letter] = []
    }
    groups[letter].push(friend)
  })
  
  const sortedGroups = {}
  Object.keys(groups).sort().forEach(key => {
    sortedGroups[key] = groups[key]
  })
  
  return sortedGroups
})

const indexLetters = computed(() => {
  const letters = Object.keys(groupedFriends.value)
  return letters.length > 0 ? letters : ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '#']
})

const selectFriend = (friend) => {
  emit('selectFriend', friend)
}

const scrollToGroup = (letter) => {
  activeIndex.value = letter
  showIndexToast.value = true
  setTimeout(() => {
    showIndexToast.value = false
  }, 500)
  
  const groupElement = document.getElementById('group-' + letter)
  if (groupElement) {
    groupElement.scrollIntoView({ behavior: 'auto' })
  }
}

const onIndexTouchStart = (e, letter) => {
  e.preventDefault()
  currentTouchLetter.value = letter
  scrollToGroup(letter)
}

const onIndexTouchMove = (e) => {
  if (!e.touches || e.touches.length === 0) return
  
  const touch = e.touches[0]
  const element = document.elementFromPoint(touch.clientX, touch.clientY)
  
  if (element && element.classList.contains('index-item')) {
    const letter = element.textContent
    if (letter !== currentTouchLetter.value) {
      currentTouchLetter.value = letter
      scrollToGroup(letter)
    }
  }
}

const onIndexTouchEnd = () => {
  currentTouchLetter.value = null
  showIndexToast.value = false
}

onMounted(() => {
  console.log('FriendList mounted')
})
</script>

<style scoped>
.friend-list {
  display: flex;
  flex-direction: column;
  height: 100%;
  background: linear-gradient(180deg, #f8fafc 0%, #f1f5f9 100%);
  position: relative;
}

.list-header {
  padding: 16px 16px 12px;
  background: linear-gradient(180deg, #ffffff 0%, #f8fafc 100%);
  border-bottom: 1px solid rgba(0, 0, 0, 0.05);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.03);
  z-index: 10;
}

.search-wrapper {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 20px;
  padding: 10px 16px;
  border: 1px solid rgba(0, 0, 0, 0.05);
  transition: all 0.3s ease;
}

.search-wrapper:focus-within {
  background: #ffffff;
  border-color: #6366f1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
}

.search-icon {
  color: #94a3b8;
  margin-right: 10px;
  font-size: 16px;
  transition: color 0.3s ease;
}

.search-wrapper:focus-within .search-icon {
  color: #6366f1;
}

.search-input {
  flex: 1;
  border: none;
  outline: none;
  background: transparent;
  font-size: 15px;
  color: #1e293b;
  placeholder: #94a3b8;
}

.search-input::placeholder {
  color: #94a3b8;
}

.list-content {
  flex: 1;
  overflow-y: auto;
  padding-right: 28px;
  background: transparent;
}

.quick-friends {
  background: #ffffff;
  margin: 12px 12px 0;
  border-radius: 16px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  overflow: hidden;
}

.section-title {
  position: sticky;
  top: 0;
  z-index: 10;
  padding: 10px 16px;
  background: linear-gradient(180deg, #f8fafc 0%, #f1f5f9 100%);
  font-size: 12px;
  font-weight: 600;
  color: #64748b;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.title-icon {
  margin-right: 6px;
  font-size: 14px;
}

.friend-group {
  background: #ffffff;
  margin: 12px 12px 0;
  border-radius: 16px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  overflow: hidden;
}

.friend-item {
  display: flex;
  align-items: center;
  padding: 14px 16px;
  cursor: pointer;
  transition: all 0.2s ease;
  position: relative;
}

.friend-item:not(:last-child) {
  border-bottom: 1px solid rgba(0, 0, 0, 0.03);
}

.friend-item::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
  background: linear-gradient(180deg, #6366f1 0%, #8b5cf6 100%);
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 0.2s ease;
}

.friend-item:active {
  background: rgba(99, 102, 241, 0.05);
}

.friend-item:active::before {
  transform: scaleX(1);
}

.friend-avatar {
  margin-right: 14px;
  position: relative;
}

.avatar {
  width: 50px;
  height: 50px;
  border-radius: 14px;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: 700;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
  position: relative;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.friend-item:active .avatar {
  transform: scale(0.95);
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.25);
}

.unread-dot {
  position: absolute;
  top: -2px;
  right: -2px;
  width: 12px;
  height: 12px;
  background: #ef4444;
  border: 2px solid #ffffff;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(239, 68, 68, 0.4);
}

.friend-info {
  flex: 1;
  min-width: 0;
}

.friend-name {
  display: flex;
  align-items: center;
  margin-bottom: 4px;
}

.name-text {
  font-size: 16px;
  font-weight: 600;
  color: #1e293b;
  letter-spacing: 0.3px;
}

.unread-badge {
  margin-left: 8px;
  padding: 2px 8px;
  background: linear-gradient(135deg, #ef4444 0%, #dc2626 100%);
  color: white;
  font-size: 11px;
  font-weight: 700;
  border-radius: 10px;
  min-width: 20px;
  text-align: center;
  box-shadow: 0 2px 6px rgba(239, 68, 68, 0.3);
  animation: pulse-badge 2s ease-in-out infinite;
}

@keyframes pulse-badge {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 2px 6px rgba(239, 68, 68, 0.3);
  }
  50% {
    transform: scale(1.05);
    box-shadow: 0 4px 12px rgba(239, 68, 68, 0.4);
  }
}

.friend-last-message {
  font-size: 13px;
  color: #64748b;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  letter-spacing: 0.2px;
}

.friend-time {
  font-size: 12px;
  color: #94a3b8;
  margin-left: 12px;
  font-weight: 500;
}

.index-bar {
  position: fixed;
  right: 6px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 20;
  background: rgba(255, 255, 255, 0.9);
  backdrop-filter: blur(10px);
  border-radius: 12px;
  padding: 6px 4px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.08);
}

.index-item {
  width: 24px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: 600;
  color: #64748b;
  cursor: pointer;
  border-radius: 6px;
  transition: all 0.2s ease;
}

.index-item:active,
.index-item.index-active {
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
  color: white;
  transform: scale(1.15);
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.4);
}

.index-toast {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 70px;
  height: 70px;
  background: rgba(0, 0, 0, 0.75);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  font-weight: 700;
  color: white;
  z-index: 100;
  animation: toast-in 0.2s ease;
}

@keyframes toast-in {
  from {
    opacity: 0;
    transform: translate(-50%, -50%) scale(0.8);
  }
  to {
    opacity: 1;
    transform: translate(-50%, -50%) scale(1);
  }
}
</style>
