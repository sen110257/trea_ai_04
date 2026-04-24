<template>
  <div class="friend-list">
    <div class="list-header">
      <input 
        v-model="searchText" 
        type="text" 
        placeholder="搜索联系人" 
        class="search-input"
      />
    </div>
    
    <div class="list-content" ref="listContentRef">
      <div class="quick-friends" v-if="quickFriends.length > 0">
        <div class="section-title">常用联系人</div>
        <div 
          v-for="friend in quickFriends" 
          :key="friend.id" 
          class="friend-item"
          @click="selectFriend(friend)"
        >
          <div class="friend-avatar">
            <div class="avatar">{{ friend.avatar }}</div>
          </div>
          <div class="friend-info">
            <div class="friend-name">
              <span>{{ friend.name }}</span>
              <span class="unread-badge" v-if="friend.unread > 0">{{ friend.unread }}</span>
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
            <div class="avatar">{{ friend.avatar }}</div>
          </div>
          <div class="friend-info">
            <div class="friend-name">
              <span>{{ friend.name }}</span>
              <span class="unread-badge" v-if="friend.unread > 0">{{ friend.unread }}</span>
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
        @click="scrollToGroup(letter)"
        @touchstart="onIndexTouchStart($event, letter)"
        @touchmove="onIndexTouchMove"
        @touchend="onIndexTouchEnd"
      >
        {{ letter }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { friends } from '../data/mockData.js'

const emit = defineEmits(['selectFriend'])

const searchText = ref('')
const listContentRef = ref(null)
const currentTouchLetter = ref(null)

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
  const groupElement = document.getElementById('group-' + letter)
  if (groupElement) {
    groupElement.scrollIntoView({ behavior: 'smooth' })
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
  background-color: #fff;
  position: relative;
}

.list-header {
  padding: 10px 15px;
  background-color: #f5f5f5;
  border-bottom: 1px solid #e5e5e5;
}

.search-input {
  width: 100%;
  height: 36px;
  padding: 0 15px;
  border: none;
  border-radius: 18px;
  background-color: #e5e5e5;
  font-size: 14px;
  outline: none;
}

.list-content {
  flex: 1;
  overflow-y: auto;
  padding-right: 25px;
}

.quick-friends {
  background-color: #fff;
}

.section-title {
  position: sticky;
  top: 0;
  z-index: 10;
  padding: 5px 15px;
  background-color: #f5f5f5;
  font-size: 12px;
  color: #999;
}

.friend-group {
  background-color: #fff;
}

.friend-item {
  display: flex;
  align-items: center;
  padding: 12px 15px;
  border-bottom: 1px solid #f0f0f0;
  cursor: pointer;
}

.friend-item:active {
  background-color: #f5f5f5;
}

.friend-avatar {
  margin-right: 12px;
}

.avatar {
  width: 45px;
  height: 45px;
  border-radius: 50%;
  background-color: #409eff;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  font-weight: bold;
}

.friend-info {
  flex: 1;
  min-width: 0;
}

.friend-name {
  display: flex;
  align-items: center;
  font-size: 16px;
  color: #333;
  margin-bottom: 4px;
}

.unread-badge {
  margin-left: 8px;
  padding: 1px 6px;
  background-color: #ff4d4f;
  color: white;
  font-size: 12px;
  border-radius: 10px;
  min-width: 18px;
  text-align: center;
}

.friend-last-message {
  font-size: 13px;
  color: #999;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.friend-time {
  font-size: 12px;
  color: #ccc;
  margin-left: 10px;
}

.index-bar {
  position: fixed;
  right: 5px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 20;
}

.index-item {
  width: 20px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  color: #666;
  cursor: pointer;
}

.index-item:active {
  background-color: #409eff;
  color: white;
  border-radius: 50%;
}
</style>
