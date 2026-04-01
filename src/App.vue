<template>
  <van-config-provider :theme="theme">
    <div class="app-container">

      <!-- 首页 -->
      <HomePage
        v-if="!currentRoom"
        :user="currentUser"
        :last-room-code="lastRoomCode"
        @edit-profile="handleEditProfile"
        @return-room="goToLastRoom"
        @create-room="handleCreateRoom"
        @show-join="showJoinModal = true"
      />

      <!-- 房间页面 -->
      <RoomPage
        v-else
        :room="currentRoom"
        :my-score="myScore"
        :refreshing="refreshing"
        @go-home="goHome"
        @refresh="refreshRoom"
        @show-settle="showSettlePlan"
        @show-spend="showSpendModal = true"
        @show-reclaim="showReclaimModal = true"
        @reclaim-all="handleReclaimAll"
      />

      <!-- 加入房间弹窗 -->
      <JoinModal
        v-model:show="showJoinModal"
        :loading="loading"
        @join="handleJoinRoom"
      />

      <!-- 设置昵称弹窗 -->
      <SetNameModal
        v-model:show="showSetNameModal"
        :loading="loading"
        :initial-nickname="tempNickname"
        :initial-avatar="tempAvatar"
        @confirm="confirmSetName"
      />

      <!-- 支出积分弹窗 -->
      <SpendModal
        v-model:show="showSpendModal"
        :loading="loading"
        :my-score="myScore"
        @spend="handleSpend"
      />

      <!-- 收回积分弹窗 -->
      <ReclaimModal
        v-model:show="showReclaimModal"
        :loading="loading"
        :desk-score="currentRoom?.deskScore || 0"
        @reclaim="handleReclaim"
        @reclaim-all="handleReclaimAll"
      />

      <!-- 结算方案弹窗 -->
      <SettleModal
        v-model:show="showSettleModal"
        :plan="settlePlan"
      />

    </div>
  </van-config-provider>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { io } from 'socket.io-client'
import { showToast } from 'vant'
import 'vant/lib/index.css'

// Components
import HomePage from './components/HomePage.vue'
import RoomPage from './components/RoomPage.vue'
import JoinModal from './components/JoinModal.vue'
import SetNameModal from './components/SetNameModal.vue'
import SpendModal from './components/SpendModal.vue'
import ReclaimModal from './components/ReclaimModal.vue'
import SettleModal from './components/SettleModal.vue'

const API_BASE = 'http://8.129.231.250'
const WS_URL = 'http://8.129.231.250'

// Socket.io 配置
const socket = io(WS_URL, {
  transports: ['websocket', 'polling'],
  reconnection: true,
  reconnectionAttempts: 10,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
  timeout: 20000,
  pingTimeout: 60000,
  pingInterval: 25000
})

let currentRoomCode = ref('')

// 监听连接事件
socket.on('connect', () => {
  console.log('WebSocket connected:', socket.id)
  if (currentRoomCode.value) {
    socket.emit('joinRoom', currentRoomCode.value)
  }
})

socket.on('disconnect', (reason) => {
  console.log('WebSocket disconnected:', reason)
})

socket.on('reconnect', (attemptNumber) => {
  console.log('WebSocket reconnected after', attemptNumber, 'attempts')
  if (currentRoomCode.value) {
    socket.emit('joinRoom', currentRoomCode.value)
  }
})

socket.on('reconnect_failed', () => {
  console.log('WebSocket reconnection failed')
})

// 监听房间更新
socket.on('roomUpdate', (room) => {
  if (currentRoom.value && currentRoom.value.roomCode === room.roomCode) {
    currentRoom.value = room
  }
})

// 监听成员资料更新
socket.on('memberUpdate', (data) => {
  if (!currentRoom.value || !currentRoom.value.members) return
  const idx = currentRoom.value.members.findIndex(m => m.odid === data.odid)
  if (idx !== -1) {
    if (data.nickname) currentRoom.value.members[idx].nickname = data.nickname
    if (data.avatar) currentRoom.value.members[idx].avatar = data.avatar
  }
})

const currentUser = ref(null)
const currentRoom = ref(null)
const theme = ref('light')
const lastRoomCode = ref('')

const showJoinModal = ref(false)
const showSetNameModal = ref(false)
const showSpendModal = ref(false)
const showReclaimModal = ref(false)
const showSettleModal = ref(false)
const settlePlan = ref([])

const joinCode = ref('')
const spendAmount = ref('')
const reclaimAmount = ref('')
const tempNickname = ref('')
const tempAvatar = ref('')

const pendingRoom = ref(null)
const loading = ref(false)
const refreshing = ref(false)

const defaultAvatar = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="35" r="25" fill="%23bdc3c7"/><circle cx="50" cy="100" r="40" fill="%23bdc3c7"/></svg>'

const myScore = computed(() => {
  if (!currentRoom.value || !currentUser.value) return 0
  const member = currentRoom.value.members?.find(m => m.odid === currentUser.value.odid)
  return member?.personalScore || 0
})

function generateUserId() {
  let odid = localStorage.getItem('odid')
  if (!odid) {
    odid = 'user_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9)
    localStorage.setItem('odid', odid)
  }
  return odid
}

async function initUser() {
  const odid = generateUserId()
  const localNickname = localStorage.getItem('nickname')
  const localAvatar = localStorage.getItem('avatar')

  currentUser.value = {
    odid,
    nickname: localNickname || '',
    avatar: localAvatar || ''
  }

  try {
    const res = await fetch(`${API_BASE}/api/user`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid,
        nickname: localNickname || '',
        avatar: localAvatar || ''
      })
    })
    const data = await res.json()
    if (data.success) {
      if (data.data.nickname) {
        currentUser.value.nickname = data.data.nickname
        if (!localNickname) {
          localStorage.setItem('nickname', data.data.nickname)
        }
      }
      if (data.data.avatar) {
        currentUser.value.avatar = data.data.avatar
        if (!localAvatar) {
          localStorage.setItem('avatar', data.data.avatar)
        }
      }
    }
  } catch (e) {
    console.error(e)
  }
}

function handleEditProfile() {
  tempNickname.value = currentUser.value?.nickname || ''
  tempAvatar.value = currentUser.value?.avatar || ''
  showSetNameModal.value = true
}

async function handleCreateRoom() {
  if (!currentUser.value?.nickname) {
    pendingRoom.value = { type: 'create' }
    showSetNameModal.value = true
    return
  }
  await createRoom()
}

async function createRoom() {
  loading.value = true
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
      currentRoomCode.value = data.data.roomCode
      saveLastRoom(data.data.roomCode)
      socket.emit('joinRoom', data.data.roomCode)
      showSetNameModal.value = false
    } else {
      showToast(data.error || '创建失败')
    }
  } catch (e) {
    showToast('网络错误')
  } finally {
    loading.value = false
  }
}

async function handleJoinRoom(code) {
  joinCode.value = code
  if (!joinCode.value || joinCode.value.length !== 6) {
    showToast('请输入6位房间号')
    return
  }
  loading.value = true
  try {
    const checkRes = await fetch(`${API_BASE}/api/rooms/${joinCode.value}`)
    const checkData = await checkRes.json()
    if (!checkData.success) {
      showToast('房间不存在')
      return
    }
    if (!currentUser.value?.nickname) {
      pendingRoom.value = { type: 'join', code: joinCode.value }
      showJoinModal.value = false
      showSetNameModal.value = true
      loading.value = false
      return
    }
    await joinRoom(joinCode.value)
  } catch (e) {
    showToast('网络错误')
  } finally {
    loading.value = false
  }
}

async function joinRoom(code) {
  loading.value = true
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
      currentRoomCode.value = code
      syncUserInfoInRoom()
      saveLastRoom(code)
      socket.emit('joinRoom', code)
      showJoinModal.value = false
      joinCode.value = ''
    } else {
      showToast(data.error || '加入失败')
    }
  } catch (e) {
    showToast('网络错误')
  } finally {
    loading.value = false
  }
}

function syncUserInfoInRoom() {
  if (!currentRoom.value || !currentRoom.value.members) return
  const localNickname = localStorage.getItem('nickname')
  const localAvatar = localStorage.getItem('avatar')
  const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value?.odid)
  if (idx !== -1 && (localNickname || localAvatar)) {
    if (localNickname) currentRoom.value.members[idx].nickname = localNickname
    if (localAvatar) currentRoom.value.members[idx].avatar = localAvatar
  }
}

async function confirmSetName({ nickname, avatar }) {
  loading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/user`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname,
        avatar
      })
    })
    const data = await res.json()
    if (data.success) {
      currentUser.value = { ...currentUser.value, nickname, avatar }
      localStorage.setItem('nickname', nickname)
      if (avatar) {
        localStorage.setItem('avatar', avatar)
      }

      if (currentRoom.value && currentRoom.value._id) {
        await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/updateMember`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ odid: currentUser.value.odid, nickname, avatar })
        })
      }

      if (currentRoom.value && currentRoom.value.members) {
        const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value.odid)
        if (idx !== -1) {
          currentRoom.value.members[idx] = { ...currentRoom.value.members[idx], nickname, avatar }
        }
      }

      showSetNameModal.value = false
      tempNickname.value = ''
      tempAvatar.value = ''

      if (pendingRoom.value?.type === 'create') {
        await createRoom()
      } else if (pendingRoom.value?.type === 'join') {
        await joinRoom(pendingRoom.value.code)
      }
      pendingRoom.value = null
      showToast('保存成功')
    }
  } catch (e) {
    showToast('保存失败')
  } finally {
    loading.value = false
  }
}

async function handleSpend(amount) {
  const myAvatar = currentUser.value?.avatar
  const myNickname = currentUser.value?.nickname
  loading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/spend`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ odid: currentUser.value.odid, nickname: currentUser.value.nickname, amount })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      if (currentRoom.value.members) {
        const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value.odid)
        if (idx !== -1) { currentRoom.value.members[idx].avatar = myAvatar; currentRoom.value.members[idx].nickname = myNickname }
      }
      showSpendModal.value = false
    } else { showToast(data.error || '积分不足') }
  } catch (e) { showToast('网络错误') }
  finally { loading.value = false }
}

function calculateSettlePlan() {
  if (!currentRoom.value || !currentRoom.value.members) return []

  const members = currentRoom.value.members
  const negatives = members.filter(m => m.personalScore < 0).map(m => ({ ...m }))
  const positives = members.filter(m => m.personalScore > 0).map(m => ({ ...m }))

  negatives.sort((a, b) => a.personalScore - b.personalScore)
  positives.sort((a, b) => b.personalScore - a.personalScore)

  const plan = []
  let i = 0, j = 0

  while (i < negatives.length && j < positives.length) {
    const debtor = negatives[i]
    const creditor = positives[j]

    const debt = Math.abs(debtor.personalScore)
    const credit = creditor.personalScore
    const amount = Math.min(debt, credit)

    if (amount > 0) {
      plan.push({
        from: { odid: debtor.odid, nickname: debtor.nickname, avatar: debtor.avatar },
        to: { odid: creditor.odid, nickname: creditor.nickname, avatar: creditor.avatar },
        amount
      })
    }

    debtor.personalScore += amount
    creditor.personalScore -= amount

    if (Math.abs(debtor.personalScore) < 1) { i++ }
    if (creditor.personalScore < 1) { j++ }
  }

  return plan
}

function showSettlePlan() {
  if (!currentRoom.value) return
  if (currentRoom.value.deskScore !== 0) {
    showToast('桌面还有积分未收回呢')
    return
  }
  settlePlan.value = calculateSettlePlan()
  showSettleModal.value = true
}

async function handleReclaim(amount) {
  const myAvatar = currentUser.value?.avatar
  const myNickname = currentUser.value?.nickname
  loading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/reclaim`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ odid: currentUser.value.odid, nickname: currentUser.value.nickname, amount })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      if (currentRoom.value.members) {
        const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value.odid)
        if (idx !== -1) { currentRoom.value.members[idx].avatar = myAvatar; currentRoom.value.members[idx].nickname = myNickname }
      }
      showReclaimModal.value = false
    } else { showToast(data.error || '桌面积分不足') }
  } catch (e) { showToast('网络错误') }
  finally { loading.value = false }
}

async function handleReclaimAll() {
  const deskScore = currentRoom.value?.deskScore || 0
  if (deskScore <= 0) { showToast('桌面积为0'); return }
  const myAvatar = currentUser.value?.avatar
  const myNickname = currentUser.value?.nickname
  loading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/reclaim`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ odid: currentUser.value.odid, nickname: currentUser.value.nickname, amount: deskScore })
    })
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      if (currentRoom.value.members) {
        const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value.odid)
        if (idx !== -1) { currentRoom.value.members[idx].avatar = myAvatar; currentRoom.value.members[idx].nickname = myNickname }
      }
      showReclaimModal.value = false
      showToast({ message: `收回 ${deskScore} 积分成功`, position: 'top' })
    } else { showToast(data.error || '桌面积分不足') }
  } catch (e) { showToast('网络错误') }
  finally { loading.value = false }
}

function handleVisibilityChange() {
  if (document.visibilityState === 'visible' && currentRoom.value) {
    refreshRoom(false)
  }
}

onMounted(async () => {
  await initUser()
  const saved = localStorage.getItem('lastRoomCode')
  if (saved) lastRoomCode.value = saved
  document.addEventListener('visibilitychange', handleVisibilityChange)
})

onUnmounted(() => {
  document.removeEventListener('visibilitychange', handleVisibilityChange)
})

function saveLastRoom(roomCode) { localStorage.setItem('lastRoomCode', roomCode); lastRoomCode.value = roomCode }
async function goToLastRoom() { if (lastRoomCode.value) await goToRoom(lastRoomCode.value) }

async function leaveRoom() {
  if (!currentRoom.value || !currentUser.value) return
  try {
    await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/leave`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ odid: currentUser.value.odid, nickname: currentUser.value.nickname })
    })
  } catch (e) { /* ignore */ }
  currentRoom.value = null
  currentRoomCode.value = ''
}

function goHome() {
  leaveRoom()
}

async function refreshRoom(showTip = true) {
  if (!currentRoom.value) return
  refreshing.value = true
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value.roomCode}`)
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      if (showTip) showToast('已刷新')
    }
  } catch (e) {
    showToast('刷新失败')
  } finally {
    refreshing.value = false
  }
}

async function goToRoom(roomCode) {
  try {
    if (currentUser.value) {
      const checkRes = await fetch(`${API_BASE}/api/rooms/${roomCode}`)
      const checkData = await checkRes.json()
      if (checkData.success) {
        const isInRoom = checkData.data.members?.some(m => m.odid === currentUser.value.odid)
        if (!isInRoom) {
          await fetch(`${API_BASE}/api/rooms/rejoin`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              roomCode,
              odid: currentUser.value.odid,
              nickname: currentUser.value.nickname,
              avatar: currentUser.value.avatar
            })
          })
        } else {
          await fetch(`${API_BASE}/api/rooms/${checkData.data._id}/updateMember`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              odid: currentUser.value.odid,
              nickname: currentUser.value.nickname,
              avatar: currentUser.value.avatar
            })
          })
        }
      }
    }

    const res = await fetch(`${API_BASE}/api/rooms/${roomCode}`)
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      currentRoomCode.value = roomCode
      syncUserInfoInRoom()
      saveLastRoom(roomCode)
      socket.emit('joinRoom', roomCode)
    }
    else { showToast('房间不存在或已过期'); localStorage.removeItem('lastRoomCode'); lastRoomCode.value = '' }
  } catch (e) { showToast('网络错误') }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&family=Inter:wght@400;500;600&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:wght,FILL@100..700,0..1&display=swap');

* { margin: 0; padding: 0; box-sizing: border-box; }

:root {
  --background: #131313;
  --surface: #1c1b1b;
  --surface-container: #20201f;
  --surface-container-high: #2a2a2a;
  --surface-container-highest: #353535;
  --on-background: #e5e2e1;
  --on-surface: #e5e2e1;
  --on-surface-variant: #d1c5b4;
  --primary: #e9c176;
  --on-primary: #261900;
  --outline: #9a8f80;
  --error: #ffb4ab;
  --success: #98d3ba;
}

body {
  background: var(--background);
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  -webkit-font-smoothing: antialiased;
}

.app-container {
  min-height: 100vh;
  background: var(--background);
}

.material-symbols-outlined {
  font-variation-settings: 'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24;
}

/* Vant overrides for dark theme */
.van-popup {
  background: var(--surface) !important;
}

.van-field__control {
  color: var(--on-surface) !important;
  background: var(--surface-container) !important;
  border-radius: 12px !important;
}

.van-field__body {
  height: 44px !important;
}

.van-button--primary {
  background: var(--primary) !important;
  color: var(--on-primary) !important;
  border: none !important;
}

.van-button--danger {
  background: #ff6b6b !important;
  border: none !important;
}

.van-button--success {
  background: var(--success) !important;
  color: #002116 !important;
  border: none !important;
}

.van-tag--danger {
  background: rgba(255, 107, 107, 0.2) !important;
  color: #ff6b6b !important;
}

.van-tag--success {
  background: rgba(152, 211, 186, 0.2) !important;
  color: var(--success) !important;
}

.van-tag--primary {
  background: rgba(233, 193, 118, 0.2) !important;
  color: var(--primary) !important;
}

.van-tag--warning {
  background: rgba(255, 151, 106, 0.2) !important;
  color: #ff976a !important;
}

.van-tag--default {
  background: rgba(154, 143, 128, 0.2) !important;
  color: var(--outline) !important;
}
</style>