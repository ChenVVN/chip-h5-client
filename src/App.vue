<template>
  <van-config-provider :theme="theme">
    <div class="app-container">
      
      <!-- 首页：两个按钮 -->
      <div v-if="!currentRoom" class="page-home">
        <div class="home-content">
          <div class="logo-area">
            <img src="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 200 200'><circle cx='100' cy='100' r='90' fill='%231980ff'/><text x='100' y='130' text-anchor='middle' fill='white' font-size='80'>🎰</text></svg>" class="logo" />
            <h1 class="title">筹码平台</h1>
          </div>
          
          <!-- 如果有记住的房间，显示返回按钮 -->
          <div v-if="lastRoomCode" class="last-room-btn">
            <van-button type="warning" size="large" round @click="goToLastRoom" block>
              返回房间 {{ lastRoomCode }}
            </van-button>
          </div>
          
          <div class="btn-group">
            <van-button type="primary" size="large" round @click="handleCreateRoom" class="action-btn">
              创建房间
            </van-button>
            <van-button type="default" size="large" round @click="showJoinModal = true" class="action-btn">
              加入房间
            </van-button>
          </div>
        </div>
      </div>

      <!-- 房间页面 -->
      <div v-else class="page-room">
        <!-- 顶部：房间号 + 返回按钮 -->
        <div class="room-header">
          <div class="room-header-left" @click="goHome">
            <van-icon name="arrow-left" />
          </div>
          <div class="room-code">房间号: {{ currentRoom.roomCode }}</div>
          <div class="room-header-right"></div>
        </div>

        <!-- 积分区域 -->
        <div class="score-area">
          <div class="score-item">
            <div class="score-label">桌面积分</div>
            <div class="score-value">{{ currentRoom.deskScore || 0 }}</div>
          </div>
          <div class="score-divider"></div>
          <div class="score-item">
            <div class="score-label">我的积分</div>
            <div class="score-value mine">{{ myScore }}</div>
          </div>
        </div>

        <!-- 成员列表 -->
        <div class="members-area">
          <div class="area-title">成员</div>
          <div class="member-list">
            <div v-for="member in currentRoom.members" :key="member.odid" class="member-item">
              <img :src="member.avatar || defaultAvatar" class="member-avatar" />
              <span class="member-name">{{ member.nickname }}</span>
              <span class="member-score" :class="member.personalScore >= 0 ? 'positive' : 'negative'">
                {{ member.personalScore >= 0 ? '+' : '' }}{{ member.personalScore }}
              </span>
            </div>
          </div>
        </div>

        <!-- 操作日志 -->
        <div class="logs-area">
          <div class="area-title">记录</div>
          <div class="log-list" v-if="currentRoom.logs?.length > 0">
            <div v-for="(log, index) in currentRoom.logs" :key="index" class="log-item">
              <van-tag :type="log.action === '支出' ? 'danger' : 'success'" size="small">
                {{ log.action }}
              </van-tag>
              <span class="log-text">
                <strong>{{ log.nickname }}</strong>
                {{ log.action === '支出' ? '支出了' : '收回了' }}
                <strong>{{ log.amount }}</strong> 积分
              </span>
              <span class="log-time">{{ formatTime(log.timestamp) }}</span>
            </div>
          </div>
          <div v-else class="no-logs">暂无记录</div>
        </div>

        <!-- 底部操作按钮 -->
        <div class="room-actions">
          <van-button type="danger" round @click="showSpendModal = true">支出</van-button>
          <van-button type="success" round @click="showReclaimModal = true">收回</van-button>
        </div>
      </div>

      <!-- 加入房间弹窗 -->
      <van-popup v-model:show="showJoinModal" round position="bottom">
        <div class="popup">
          <h3>加入房间</h3>
          <van-field v-model="joinCode" type="digit" maxlength="6" placeholder="请输入6位房间号" />
          <van-button type="primary" size="large" round @click="handleJoinRoom" block>加入</van-button>
        </div>
      </van-popup>

      <!-- 设置昵称弹窗 -->
      <van-popup v-model:show="showSetNameModal" round position="bottom">
        <div class="popup">
          <h3>设置你的昵称</h3>
          <van-field v-model="tempNickname" placeholder="请输入昵称" maxlength="12" />
          <van-uploader :after-read="onAvatarRead" accept="image/*">
            <div class="avatar-upload">
              <img :src="tempAvatar || defaultAvatar" class="preview-avatar" />
              <span>点击更换头像</span>
            </div>
          </van-uploader>
          <van-button type="primary" size="large" round @click="confirmSetName" block>进入房间</van-button>
        </div>
      </van-popup>

      <!-- 支出弹窗 -->
      <van-popup v-model:show="showSpendModal" round position="bottom">
        <div class="popup">
          <h3>支出积分</h3>
          <p class="hint">我的积分: {{ myScore }}</p>
          <van-field v-model="spendAmount" type="digit" placeholder="请输入积分数量" />
          <van-button type="danger" size="large" round @click="handleSpend" block>确认支出</van-button>
        </div>
      </van-popup>

      <!-- 收回弹窗 -->
      <van-popup v-model:show="showReclaimModal" round position="bottom">
        <div class="popup">
          <h3>收回积分</h3>
          <p class="hint">桌面积分: {{ currentRoom?.deskScore || 0 }}</p>
          <van-field v-model="reclaimAmount" type="digit" placeholder="请输入积分数量" />
          <van-button type="success" size="large" round @click="handleReclaim" block>确认收回</van-button>
        </div>
      </van-popup>

    </div>
  </van-config-provider>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { io } from 'socket.io-client'
import { showToast } from 'vant'
import 'vant/lib/index.css'

const API_BASE = window.location.protocol + '//' + window.location.hostname + ':3000'
const WS_URL = window.location.protocol + '//' + window.location.hostname + ':3000'

// Socket.io 连接
const socket = io(WS_URL, {
  transports: ['websocket', 'polling']
})

socket.on('connect', () => {
  console.log('WebSocket connected')
})

socket.on('roomUpdate', (room) => {
  if (currentRoom.value && currentRoom.value.roomCode === room.roomCode) {
    currentRoom.value = room
  }
})

// 状态
const currentUser = ref(null)
const currentRoom = ref(null)
const theme = ref('light')
const lastRoomCode = ref('')

// 弹窗状态
const showJoinModal = ref(false)
const showSetNameModal = ref(false)
const showSpendModal = ref(false)
const showReclaimModal = ref(false)

// 表单数据
const joinCode = ref('')
const spendAmount = ref('')
const reclaimAmount = ref('')
const tempNickname = ref('')
const tempAvatar = ref('')

// 待加入的房间信息
const pendingRoom = ref(null)

const defaultAvatar = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="35" r="25" fill="%23bdc3c7"/><circle cx="50" cy="100" r="40" fill="%23bdc3c7"/></svg>'

const myScore = computed(() => {
  if (!currentRoom.value || !currentUser.value) return 0
  const member = currentRoom.value.members?.find(m => m.odid === currentUser.value.odid)
  return member?.personalScore || 0
})

// 生成用户ID
function generateUserId() {
  let odid = localStorage.getItem('odid')
  if (!odid) {
    odid = 'user_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9)
    localStorage.setItem('odid', odid)
  }
  return odid
}

// 初始化用户
async function initUser() {
  const odid = generateUserId()
  try {
    const res = await fetch(`${API_BASE}/api/user`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ 
        odid, 
        nickname: localStorage.getItem('nickname') || '',
        avatar: localStorage.getItem('avatar') || ''
      })
    })
    const data = await res.json()
    if (data.success) {
      currentUser.value = data.data
    }
  } catch (e) {
    console.error(e)
  }
}

// 创建房间
async function handleCreateRoom() {
  // 如果没有昵称，先设置
  if (!currentUser.value?.nickname) {
    pendingRoom.value = { type: 'create' }
    showSetNameModal.value = true
    return
  }
  await createRoom()
}

async function createRoom() {
  try {
    const res = await fetch(`${API_BASE}/api/rooms`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        ownerId: currentUser.value.odid,
        ownerName: currentUser.value.nickname,
        ownerAvatar: currentUser.value.avatar
      })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      saveLastRoom(data.data.roomCode)
      socket.emit('joinRoom', data.data.roomCode)
      showSetNameModal.value = false
    } else {
      showToast(data.error || '创建失败')
    }
  } catch (e) {
    showToast('网络错误')
  }
}

// 加入房间
async function handleJoinRoom() {
  if (!joinCode.value || joinCode.value.length !== 6) {
    showToast('请输入6位房间号')
    return
  }
  
  // 先检查房间是否存在，同时如果没有昵称也要先设置
  if (!currentUser.value?.nickname) {
    pendingRoom.value = { type: 'join', code: joinCode.value }
    showJoinModal.value = false
    showSetNameModal.value = true
    return
  }
  
  await joinRoom(joinCode.value)
}

async function joinRoom(code) {
  try {
    const res = await fetch(`${API_BASE}/api/rooms/join`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        roomCode: code,
        odid: currentUser.value.odid,
        nickname: currentUser.value.nickname,
        avatar: currentUser.value.avatar
      })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      saveLastRoom(code)
      socket.emit('joinRoom', code)
      showJoinModal.value = false
      joinCode.value = ''
    } else {
      showToast(data.error || '加入失败')
    }
  } catch (e) {
    showToast('网络错误')
  }
}

// 确认设置昵称
async function confirmSetName() {
  const nickname = tempNickname.value.trim()
  if (!nickname) {
    showToast('请输入昵称')
    return
  }
  
  // 保存用户信息
  try {
    const res = await fetch(`${API_BASE}/api/user`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname,
        avatar: tempAvatar.value || currentUser.value.avatar
      })
    })
    const data = await res.json()
    if (data.success) {
      currentUser.value = data.data
      localStorage.setItem('nickname', nickname)
      if (tempAvatar.value) {
        localStorage.setItem('avatar', tempAvatar.value)
      }
      showSetNameModal.value = false
      
      // 根据待处理操作进入房间
      if (pendingRoom.value?.type === 'create') {
        await createRoom()
      } else if (pendingRoom.value?.type === 'join') {
        await joinRoom(pendingRoom.value.code)
      } else {
        // 只是单纯设置昵称
        tempNickname.value = ''
        tempAvatar.value = ''
      }
      pendingRoom.value = null
    }
  } catch (e) {
    showToast('保存失败')
  }
}

// 头像上传 - 压缩图片
function onAvatarRead(file) {
  const img = new Image()
  img.onload = () => {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')
    const maxSize = 200
    let { width, height } = img
    if (width > height) {
      if (width > maxSize) {
        height = (height * maxSize) / width
        width = maxSize
      }
    } else {
      if (height > maxSize) {
        width = (width * maxSize) / height
        height = maxSize
      }
    }
    canvas.width = width
    canvas.height = height
    ctx.drawImage(img, 0, 0, width, height)
    tempAvatar.value = canvas.toDataURL('image/jpeg', 0.7)
  }
  img.src = file.content
}

// 支出积分
async function handleSpend() {
  const amount = parseInt(spendAmount.value)
  if (!amount || amount <= 0) {
    showToast('请输入有效积分')
    return
  }
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/spend`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname: currentUser.value.nickname,
        amount
      })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      showSpendModal.value = false
      spendAmount.value = ''
    } else {
      showToast(data.error || '积分不足')
    }
  } catch (e) {
    showToast('网络错误')
  }
}

// 收回积分
async function handleReclaim() {
  const amount = parseInt(reclaimAmount.value)
  if (!amount || amount <= 0) {
    showToast('请输入有效积分')
    return
  }
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/reclaim`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname: currentUser.value.nickname,
        amount
      })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      showReclaimModal.value = false
      reclaimAmount.value = ''
    } else {
      showToast(data.error || '桌面积分不足')
    }
  } catch (e) {
    showToast('网络错误')
  }
}

function formatTime(timestamp) {
  if (!timestamp) return ''
  const date = new Date(timestamp)
  return `${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`
}

onMounted(async () => {
  await initUser()
  tempNickname.value = currentUser.value?.nickname || ''
  tempAvatar.value = currentUser.value?.avatar || ''
  // 加载上次房间
  const savedRoomCode = localStorage.getItem('lastRoomCode')
  if (savedRoomCode) {
    lastRoomCode.value = savedRoomCode
  }
})

// 保存当前房间到本地
function saveLastRoom(roomCode) {
  localStorage.setItem('lastRoomCode', roomCode)
  lastRoomCode.value = roomCode
}

// 返回上次房间
async function goToLastRoom() {
  if (!lastRoomCode.value) return
  await goToRoom(lastRoomCode.value)
}

// 返回首页
function goHome() {
  currentRoom.value = null
}

// 跳转到指定房间
async function goToRoom(roomCode) {
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${roomCode}`)
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      saveLastRoom(roomCode)
      socket.emit('joinRoom', roomCode)
    } else {
      showToast('房间不存在或已过期')
      localStorage.removeItem('lastRoomCode')
      lastRoomCode.value = ''
    }
  } catch (e) {
    showToast('网络错误')
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #f5f5f5;
}

.app-container {
  min-height: 100vh;
  background: #f5f5f5;
}

/* 首页 */
.page-home {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(180deg, #e8f4ff 0%, #f5f5f5 100%);
}

.home-content {
  width: 100%;
  padding: 20px;
}

.logo-area {
  text-align: center;
  margin-bottom: 60px;
}

.logo {
  width: 100px;
  height: 100px;
  margin-bottom: 16px;
}

.title {
  font-size: 28px;
  font-weight: 600;
  color: #333;
}

.btn-group {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.last-room-btn {
  margin-bottom: 24px;
}

.action-btn { z-index: 100; position: relative; pointer-events: auto;
  height: 50px;
  font-size: 16px;
}

/* 房间页 */
.page-room {
  min-height: 100vh;
  padding-bottom: 80px;
}

.room-header {
  background: linear-gradient(135deg, #1989fa 0%, #1565c0 100%);
  padding: 16px 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.room-header-left, .room-header-right {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: white;
  font-size: 20px;
  cursor: pointer;
}

.room-code {
  color: white;
  font-size: 18px;
  font-weight: 500;
}

.score-area {
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
  margin: 16px;
  border-radius: 12px;
  padding: 24px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.score-item {
  flex: 1;
  text-align: center;
}

.score-divider {
  width: 1px;
  height: 40px;
  background: #eee;
}

.score-label {
  font-size: 14px;
  color: #999;
  margin-bottom: 8px;
}

.score-value {
  font-size: 28px;
  font-weight: 600;
  color: #333;
}

.score-value.mine {
  color: #1989fa;
}

.members-area, .logs-area {
  background: white;
  margin: 0 16px 16px;
  border-radius: 12px;
  padding: 16px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
}

.area-title {
  font-size: 14px;
  color: #999;
  margin-bottom: 12px;
}

.member-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.member-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 8px 0;
  border-bottom: 1px solid #f5f5f5;
}

.member-item:last-child {
  border-bottom: none;
}

.member-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.member-name {
  flex: 1;
  font-size: 15px;
  color: #333;
}

.member-score {
  font-size: 16px;
  font-weight: 500;
}

.member-score.positive {
  color: #07c160;
}

.member-score.negative {
  color: #ee0a24;
}

.log-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.log-item {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
}

.log-text {
  color: #666;
  flex: 1;
}

.log-text strong {
  color: #333;
}

.log-time {
  color: #999;
  font-size: 12px;
}

.no-logs {
  text-align: center;
  color: #999;
  padding: 20px;
  font-size: 14px;
}

.room-actions {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  gap: 16px;
  padding: 16px;
  background: white;
  box-shadow: 0 -2px 8px rgba(0,0,0,0.08);
}

.room-actions .van-button {
  flex: 1;
  height: 44px;
  font-size: 15px;
}

/* 弹窗 */
.popup {
  padding: 24px 16px;
}

.popup h3 {
  font-size: 18px;
  text-align: center;
  margin-bottom: 20px;
}

.popup .hint {
  text-align: center;
  color: #999;
  font-size: 14px;
  margin-bottom: 16px;
}

.avatar-upload {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  margin: 20px 0;
}

.preview-avatar {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  object-fit: cover;
}

.avatar-upload span {
  font-size: 13px;
  color: #999;
}
</style>
