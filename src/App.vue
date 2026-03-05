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
          
          <!-- 用户信息展示 -->
          <div class="user-info-card" @click="showSetNameModal = true">
            <img :src="currentUser?.avatar || defaultAvatar" class="user-avatar" />
            <span class="user-name">{{ currentUser?.nickname || '未设置昵称' }}</span>
            <van-icon name="edit" class="edit-icon" />
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
          <div class="room-header-right">
            <van-button 
              size="small" 
              round 
              type="primary"
              @click="refreshRoom" 
              :loading="refreshing"
              class="refresh-btn"
            >
              刷新
            </van-button>
          </div>
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
          <div class="area-title-wrap">
            <span class="area-title">成员</span>
            <span class="settle-btn" @click="showSettlePlan">结算方案</span>
          </div>
          <div class="member-grid">
            <div 
              v-for="member in currentRoom.members" 
              :key="member.odid" 
              class="member-card"
            >
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
              <van-tag :type="getLogTagType(log.action)" size="small">
                {{ getLogActionText(log.action) }}
              </van-tag>
              <span class="log-text" :class="{ 'log-join': log.action === '加入', 'log-return': log.action === '返回' }">
                <strong>{{ log.nickname }}</strong>
                <template v-if="log.action === '支出'">支出了</template>
                <template v-else-if="log.action === '收回'">收回了</template>
                <template v-else-if="log.action === '加入'">加入了房间</template>
                <template v-else-if="log.action === '返回'">返回了房间</template>
                <template v-else-if="log.action === '离开'">离开了房间</template>
                <strong v-if="log.action === '支出' || log.action === '收回'">{{ log.amount }}</strong>
                <template v-if="log.action === '支出' || log.action === '收回'"> 积分</template>
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
          <van-button type="primary" size="large" round @click="handleJoinRoom" block :loading="loading">加入</van-button>
        </div>
      </van-popup>

      <!-- 设置昵称弹窗（首页修改资料用） -->
      <van-popup v-model:show="showSetNameModal" round position="bottom">
        <div class="popup">
          <h3>设置昵称</h3>
          <van-field v-model="tempNickname" placeholder="请输入昵称" maxlength="12" />
          <div class="avatar-upload-wrap">
            <van-uploader :after-read="onAvatarRead" accept="image/*">
              <div class="avatar-upload">
                <img :src="tempAvatar || defaultAvatar" class="preview-avatar" />
              </div>
            </van-uploader>
            <span class="avatar-tip">点击更换头像</span>
          </div>
          <van-button 
            type="primary" 
            size="large" 
            round 
            @click="confirmSetName" 
            :loading="loading" 
            block
          >
            保存
          </van-button>
        </div>
      </van-popup>

      <!-- 支出弹窗 -->
      <van-popup v-model:show="showSpendModal" round position="bottom">
        <div class="popup">
          <h3>支出积分</h3>
          <p class="hint">我的积分: {{ myScore }}</p>
          <van-field v-model="spendAmount" type="digit" maxlength="5" placeholder="请输入积分数量" />
          <div class="quick-numbers">
            <div v-for="n in 5" :key="n" class="quick-num-btn" @click="quickSpend(n)">{{ n }}</div>
          </div>
          <van-button type="danger" size="large" round @click="handleSpend" block :loading="loading">确认支出</van-button>
        </div>
      </van-popup>

      <!-- 收回弹窗 -->
      <van-popup v-model:show="showReclaimModal" round position="bottom">
        <div class="popup">
          <div class="reclaim-header">
            <h3>收回积分</h3>
            <span class="reclaim-all-text" @click="handleReclaimAll">全收</span>
          </div>
          <p class="hint">桌面积分: {{ currentRoom?.deskScore || 0 }}</p>
          <van-field v-model="reclaimAmount" type="digit" maxlength="5" placeholder="请输入积分数量" />
          <van-button type="success" size="large" round @click="handleReclaim" block :loading="loading">确认收回</van-button>
        </div>
      </van-popup>

      <!-- 结算方案弹窗 -->
      <van-popup v-model:show="showSettleModal" round closeable class="settle-popup">
        <div class="settle-content">
          <h3>结算方案</h3>
          <div class="settle-list" v-if="settlePlan.length > 0">
            <div v-for="(item, idx) in settlePlan" :key="idx" class="settle-item">
              <div class="settle-from">
                <img :src="item.from.avatar || defaultAvatar" class="settle-avatar" />
                <span class="settle-name">{{ item.from.nickname }}</span>
              </div>
              <div class="settle-arrow">
                <span class="settle-amount">{{ item.amount }}</span>
                <span class="settle-arrow-icon">→</span>
              </div>
              <div class="settle-to">
                <img :src="item.to.avatar || defaultAvatar" class="settle-avatar" />
                <span class="settle-name">{{ item.to.nickname }}</span>
              </div>
            </div>
          </div>
          <div v-else class="settle-empty">已结清，无需结算</div>
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

const API_BASE = 'https://chip-h5-server-production.up.railway.app'
const WS_URL = 'https://chip-h5-server-production.up.railway.app'

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
  
  // 先用本地信息初始化用户（确保不丢失）
  currentUser.value = {
    odid,
    nickname: localNickname || '',
    avatar: localAvatar || ''
  }
  
  try {
    // 发送请求，如果有本地昵称/头像就传递，让服务器更新
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
      // 确保本地存储的数据同步到 currentUser
      // 如果服务器返回了数据且本地没有，则用服务器的
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

async function handleJoinRoom() {
  if (!joinCode.value || joinCode.value.length !== 6) {
    showToast('请输入6位房间号')
    return
  }
  // 先检查房间是否存在
  loading.value = true
  try {
    const checkRes = await fetch(`${API_BASE}/api/rooms/${joinCode.value}`)
    const checkData = await checkRes.json()
    if (!checkData.success) {
      showToast('房间不存在')
      return
    }
    // 如果没有昵称，先弹出设置框
    if (!currentUser.value?.nickname) {
      pendingRoom.value = { type: 'join', code: joinCode.value }
      showJoinModal.value = false
      showSetNameModal.value = true
      loading.value = false
      return
    }
    // 有昵称，直接加入
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
      // 用本地最新信息更新房间成员
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

// 用本地最新信息同步房间内的成员数据
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

async function confirmSetName() {
  const nickname = tempNickname.value.trim()
  if (!nickname) {
    showToast('请输入昵称')
    return
  }
  const newAvatar = tempAvatar.value || currentUser.value?.avatar || ''
  loading.value = true
  try {
    // 1. 更新用户基础信息
    const res = await fetch(`${API_BASE}/api/user`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname,
        avatar: newAvatar
      })
    })
    const data = await res.json()
    if (data.success) {
      currentUser.value = { ...currentUser.value, nickname, avatar: newAvatar }
      localStorage.setItem('nickname', nickname)
      if (tempAvatar.value) {
        localStorage.setItem('avatar', tempAvatar.value)
      }
      
      // 2. 如果已经在房间里，调用房间API通知其他成员
      if (currentRoom.value && currentRoom.value._id) {
        await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/updateMember`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            odid: currentUser.value.odid,
            nickname,
            avatar: newAvatar
          })
        })
      }
      
      // 3. 更新本地房间显示
      if (currentRoom.value && currentRoom.value.members) {
        const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value.odid)
        if (idx !== -1) {
          currentRoom.value.members[idx] = { ...currentRoom.value.members[idx], nickname, avatar: newAvatar }
        }
      }
      
      showSetNameModal.value = false
      tempNickname.value = ''
      tempAvatar.value = ''
      
      // 根据待处理操作进入房间
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

function handleMemberClick(member) {
  if (member.odid === currentUser.value?.odid) {
    tempNickname.value = currentUser.value.nickname
    tempAvatar.value = currentUser.value.avatar
    showSetNameModal.value = true
  }
}

async function updateProfileInRoom() {
  const nickname = tempNickname.value.trim()
  if (!nickname) {
    showToast('请输入昵称')
    return
  }
  const newAvatar = tempAvatar.value || currentUser.value.avatar
  loading.value = true
  try {
    const res = await fetch(`${API_BASE}/api/user`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname,
        avatar: newAvatar
      })
    })
    const data = await res.json()
    if (data.success) {
      currentUser.value = { ...currentUser.value, nickname, avatar: newAvatar }
      localStorage.setItem('nickname', nickname)
      localStorage.setItem('avatar', newAvatar)
      if (currentRoom.value && currentRoom.value.members) {
        const idx = currentRoom.value.members.findIndex(m => m.odid === currentUser.value.odid)
        if (idx !== -1) {
          currentRoom.value.members[idx] = { ...currentRoom.value.members[idx], nickname, avatar: newAvatar }
        }
      }
      showSetNameModal.value = false
      tempNickname.value = ''
      tempAvatar.value = ''
      showToast('修改成功')
    }
  } catch (e) {
    showToast('保存失败')
  } finally {
    loading.value = false
  }
}

function onAvatarRead(file) {
  const img = new Image()
  img.onload = () => {
    const canvas = document.createElement('canvas')
    const ctx = canvas.getContext('2d')
    const maxSize = 200
    let { width, height } = img
    if (width > height) {
      if (width > maxSize) { height = (height * maxSize) / width; width = maxSize }
    } else {
      if (height > maxSize) { width = (width * maxSize) / height; height = maxSize }
    }
    canvas.width = width
    canvas.height = height
    ctx.drawImage(img, 0, 0, width, height)
    tempAvatar.value = canvas.toDataURL('image/jpeg', 0.7)
  }
  img.src = file.content
}

async function handleSpend() {
  const amount = parseInt(spendAmount.value)
  if (!amount || amount <= 0) { showToast('请输入有效积分'); return }
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
      showSpendModal.value = false; spendAmount.value = ''
    } else { showToast(data.error || '积分不足') }
  } catch (e) { showToast('网络错误') }
  finally { loading.value = false }
}

// 计算结算方案
function calculateSettlePlan() {
  if (!currentRoom.value || !currentRoom.value.members) return []
  
  const members = currentRoom.value.members
  // 分为正数和负数两组
  const negatives = members.filter(m => m.personalScore < 0).map(m => ({ ...m }))
  const positives = members.filter(m => m.personalScore > 0).map(m => ({ ...m }))
  
  // 按绝对值排序
  negatives.sort((a, b) => a.personalScore - b.personalScore) // 负数越小越先支付
  positives.sort((a, b) => b.personalScore - a.personalScore) // 正数越大越先接收
  
  const plan = []
  let i = 0, j = 0
  
  while (i < negatives.length && j < positives.length) {
    const debtor = negatives[i] // 欠钱的人（负数）
    const creditor = positives[j] // 收钱的人（正数）
    
    // 需要支付的金额（取绝对值）
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
    
    // 更新剩余金额
    debtor.personalScore += amount
    creditor.personalScore -= amount
    
    // 如果债务人还完了，移动到下一个
    if (Math.abs(debtor.personalScore) < 1) {
      i++
    }
    // 如果债权人收完了，移动到下一个
    if (creditor.personalScore < 1) {
      j++
    }
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

function quickSpend(n) { spendAmount.value = n.toString(); handleSpend() }

async function handleReclaim() {
  const amount = parseInt(reclaimAmount.value)
  if (!amount || amount <= 0) { showToast('请输入有效积分'); return }
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
      showReclaimModal.value = false; reclaimAmount.value = ''
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
      showReclaimModal.value = false; reclaimAmount.value = ''
    } else { showToast(data.error || '桌面积分不足') }
  } catch (e) { showToast('网络错误') }
  finally { loading.value = false }
}

function formatTime(timestamp) {
  if (!timestamp) return ''
  const date = new Date(timestamp)
  return `${date.getHours().toString().padStart(2,'0')}:${date.getMinutes().toString().padStart(2,'0')}:${date.getSeconds().toString().padStart(2,'0')}`
}

function getLogTagType(action) {
  if (action === '支出') return 'danger'
  if (action === '收回') return 'success'
  if (action === '加入') return 'primary'
  if (action === '返回') return 'warning'
  if (action === '离开') return 'default'
  return 'default'
}

function getLogActionText(action) {
  if (action === '加入') return '加入'
  if (action === '返回') return '返回'
  if (action === '离开') return '离开'
  return action
}

onMounted(async () => {
  await initUser()
  tempNickname.value = currentUser.value?.nickname || ''
  tempAvatar.value = currentUser.value?.avatar || ''
  const saved = localStorage.getItem('lastRoomCode')
  if (saved) lastRoomCode.value = saved
})

function saveLastRoom(roomCode) { localStorage.setItem('lastRoomCode', roomCode); lastRoomCode.value = roomCode }
async function goToLastRoom() { if (lastRoomCode.value) await goToRoom(lastRoomCode.value) }
// 离开房间（用户主动返回首页）
async function leaveRoom() {
  if (!currentRoom.value || !currentUser.value) return
  try {
    await fetch(`${API_BASE}/api/rooms/${currentRoom.value._id}/leave`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        odid: currentUser.value.odid,
        nickname: currentUser.value.nickname
      })
    })
  } catch (e) { /* ignore */ }
  currentRoom.value = null
  // 不清除 lastRoomCode，保留最后一个房间供快速返回
}

function goHome() { 
  leaveRoom()
}

// 刷新房间数据（应对WebSocket同步失败的情况）
async function refreshRoom() {
  if (!currentRoom.value) return
  refreshing.value = true
  try {
    const res = await fetch(`${API_BASE}/api/rooms/${currentRoom.value.roomCode}`)
    const data = await res.json()
    if (data.success) {
      currentRoom.value = data.data
      showToast('已刷新')
    }
  } catch (e) {
    showToast('刷新失败')
  } finally {
    refreshing.value = false
  }
}

async function goToRoom(roomCode) {
  try {
    // 先检查用户是否已在房间里，如果是则调用rejoin添加返回记录
    if (currentUser.value) {
      const checkRes = await fetch(`${API_BASE}/api/rooms/${roomCode}`)
      const checkData = await checkRes.json()
      if (checkData.success) {
        const isInRoom = checkData.data.members?.some(m => m.odid === currentUser.value.odid)
        if (!isInRoom) {
          // 不在房间里，调用rejoin接口
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
          // 已经在房间里，但可能本地信息有更新，同步最新信息到数据库
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
      // 用本地最新信息同步房间内的成员数据
      syncUserInfoInRoom()
      saveLastRoom(roomCode)
      socket.emit('joinRoom', roomCode) 
    }
    else { showToast('房间不存在或已过期'); localStorage.removeItem('lastRoomCode'); lastRoomCode.value = '' }
  } catch (e) { showToast('网络错误') }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@400;500;600;700&display=swap');
* { margin: 0; padding: 0; box-sizing: border-box; }
:root {
  --background: oklch(0.98 0.01 240);
  --surface: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --foreground-muted: oklch(0.556 0 0);
  --primary: oklch(0.6 0.15 250);
  --primary-foreground: oklch(1 0 0);
  --danger: oklch(0.6 0.18 25);
  --success: oklch(0.6 0.18 150);
  --shadow-color: oklch(0.145 0 0);
  --font-sans: 'Outfit', system-ui, -apple-system, sans-serif;
}
body { background: var(--background); font-family: var(--font-sans); -webkit-font-smoothing: antialiased; }
.app-container { min-height: 100vh; background: var(--background); }

.page-home {
  min-height: 100vh; display: flex; align-items: center; justify-content: center;
  background: radial-gradient(ellipse at 20% 20%, oklch(0.95 0.05 250) 0%, transparent 50%),
              radial-gradient(ellipse at 80% 80%, oklch(0.95 0.05 150) 0%, transparent 50%), var(--background);
  animation: gradientShift 8s ease-in-out infinite alternate;
}
@keyframes gradientShift { 0% { background-position: 0% 0%, 100% 100%; } 100% { background-position: 100% 100%, 0% 0%; } }
.home-content { width: 100%; padding: 24px; }

.user-info-card {
  display: flex;
  align-items: center;
  gap: 12px;
  background: var(--surface);
  padding: 12px 16px;
  border-radius: 14px;
  margin-bottom: 24px;
  box-shadow: 0 2px 12px oklch(var(--shadow-color) / 0.08);
  cursor: pointer;
  transition: transform 0.2s;
}
.user-info-card:active { transform: scale(0.98); }
.user-info-card .user-avatar { width: 44px; height: 44px; border-radius: 50%; box-shadow: 0 2px 8px oklch(var(--shadow-color) / 0.1); }
.user-info-card .user-name { flex: 1; font-size: 16px; font-weight: 600; color: var(--foreground); }
.user-info-card .edit-icon { font-size: 16px; color: var(--foreground-muted); }
.logo-area { text-align: center; margin-bottom: 48px; }
.logo { width: 120px; height: 120px; margin-bottom: 20px; animation: float 3s ease-in-out infinite; filter: drop-shadow(0 8px 24px oklch(0.6 0.15 250 / 0.3)); }
@keyframes float { 0%, 100% { transform: translateY(0); } 50% { transform: translateY(-8px); } }
.title { font-size: 32px; font-weight: 700; color: var(--foreground); letter-spacing: -0.02em; }
.btn-group { display: flex; flex-direction: column; gap: 14px; }
.last-room-btn { margin-bottom: 24px; animation: slideUp 0.4s ease-out; }
.action-btn { height: 56px; font-size: 16px; font-weight: 600; transition: all 0.2s; }
.action-btn:active { transform: scale(0.97); }

.page-room { min-height: 100vh; padding-bottom: 90px; background: var(--background); }
.room-header { background: linear-gradient(135deg, var(--primary) 0%, oklch(0.5 0.15 270) 100%); padding: 18px 20px; display: flex; align-items: center; justify-content: space-between; box-shadow: 0 4px 20px oklch(0.6 0.15 250 / 0.2); }
.room-header-left, .room-header-right { width: 44px; height: 44px; display: flex; align-items: center; justify-content: center; color: var(--primary-foreground); font-size: 22px; cursor: pointer; border-radius: 1rem; }
.room-header-left:active { background: oklch(1 0 0 / 0.1); }
.room-code { color: var(--primary-foreground); font-size: 17px; font-weight: 600; letter-spacing: 0.02em; }

.refresh-btn {
  font-size: 13px;
  font-weight: 600;
  background: oklch(1 0 0 / 0.2);
  border: 1px solid oklch(1 0 0 / 0.3);
}
.refresh-btn :deep(.van-button__text) {
  color: var(--primary-foreground);
}

.score-area { display: flex; align-items: center; justify-content: center; background: var(--surface); margin: 16px 12px; border-radius: 20px; padding: 24px 20px; box-shadow: 0 1px 3px oklch(var(--shadow-color) / 0.06), 0 8px 24px oklch(var(--shadow-color) / 0.08); border: 1px solid oklch(0.95 0.01 240); }
.score-item { flex: 1; text-align: center; }
.score-divider { width: 1px; height: 48px; background: linear-gradient(to bottom, transparent, oklch(0.92 0 0), transparent); }
.score-label { font-size: 13px; color: var(--foreground-muted); margin-bottom: 8px; font-weight: 500; text-transform: uppercase; letter-spacing: 0.05em; }
.score-value { font-size: 36px; font-weight: 700; color: var(--foreground); letter-spacing: -0.02em; }
.score-value.mine { background: linear-gradient(135deg, var(--primary) 0%, oklch(0.5 0.2 280) 100%); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }

.members-area, .logs-area { background: var(--surface); margin: 0 12px 12px; border-radius: 20px; padding: 16px; box-shadow: 0 1px 3px oklch(var(--shadow-color) / 0.06), 0 4px 12px oklch(var(--shadow-color) / 0.04); border: 1px solid oklch(0.95 0.01 240); }
.area-title { font-size: 13px; color: var(--foreground-muted); margin-bottom: 16px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.06em; }

.member-grid { display: flex; flex-wrap: wrap; gap: 8px; }
.member-card { width: calc(25% - 6px); aspect-ratio: 1; background: linear-gradient(145deg, oklch(0.98 0.02 240) 0%, oklch(0.95 0.02 240) 100%); border-radius: 14px; padding: 10px; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 3px; border: 1px solid oklch(0.92 0.02 240); transition: all 0.2s; }
.member-card:active { transform: scale(0.95); }
.member-card .member-avatar { width: 40px; height: 40px; border-radius: 50%; box-shadow: 0 2px 8px oklch(var(--shadow-color) / 0.1); }
.member-card .member-name { font-size: 12px; font-weight: 500; color: var(--foreground); text-align: center; max-width: 100%; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
.member-card .member-score { font-size: 11px; font-weight: 700; }
.member-score.positive { color: var(--success); }
.member-score.negative { color: var(--danger); }

.log-list { display: flex; flex-direction: column; gap: 10px; max-height: 300px; overflow-y: auto; }
.log-item { display: flex; align-items: center; gap: 10px; font-size: 14px; padding: 10px 12px; background: oklch(0.98 0.01 240); border-radius: 12px; }
.log-item:active { transform: scale(0.98); }
.log-text { color: var(--foreground-muted); flex: 1; font-weight: 450; }
.log-text strong { color: var(--foreground); font-weight: 600; }
.log-text.log-join { color: var(--primary); }
.log-text.log-return { color: #ff976a; }
.log-time { color: var(--foreground-muted); font-size: 11px; font-weight: 500; opacity: 0.7; }
.no-logs { text-align: center; color: var(--foreground-muted); padding: 32px 20px; font-size: 14px; opacity: 0.6; }

.room-actions { position: fixed; bottom: 0; left: 0; right: 0; display: flex; gap: 14px; padding: 16px 20px; background: oklch(1 0 0 / 0.85); backdrop-filter: blur(20px); box-shadow: 0 -4px 24px oklch(var(--shadow-color) / 0.08); border-top: 1px solid oklch(0.94 0.01 240); }
.room-actions .van-button { flex: 1; height: 50px; font-size: 15px; font-weight: 600; border-radius: 14px; transition: all 0.2s; }
.room-actions .van-button:active { transform: scale(0.97); }

.popup { padding: 28px 20px; background: var(--surface); border-radius: 24px 24px 0 0; }
.popup h3 { font-size: 20px; text-align: center; margin-bottom: 24px; font-weight: 700; color: var(--foreground); letter-spacing: -0.01em; }
.popup .hint { text-align: center; color: var(--foreground-muted); font-size: 14px; margin-bottom: 20px; font-weight: 500; }

.reclaim-header { display: flex; align-items: center; justify-content: center; position: relative; margin-bottom: 24px; }
.reclaim-header h3 { margin-bottom: 0; }
.reclaim-all-text { position: absolute; right: 0; top: 50%; transform: translateY(-50%); font-size: 14px; color: var(--primary); font-weight: 600; cursor: pointer; }

.area-title-wrap { display: flex; align-items: center; justify-content: space-between; margin-bottom: 16px; }
.area-title { font-size: 13px; color: var(--foreground-muted); font-weight: 600; text-transform: uppercase; letter-spacing: 0.06em; }
.settle-btn { font-size: 12px; color: var(--primary); font-weight: 600; cursor: pointer; padding: 4px 10px; background: oklch(0.6 0.15 250 / 0.1); border-radius: 8px; }

.settle-popup { max-height: 70vh; }
.settle-content { padding: 20px; min-width: 300px; }
.settle-content h3 { font-size: 18px; text-align: center; margin-bottom: 20px; font-weight: 700; }
.settle-list { display: flex; flex-direction: column; gap: 16px; }
.settle-item { display: flex; align-items: center; justify-content: space-between; gap: 12px; }
.settle-from, .settle-to { display: flex; flex-direction: column; align-items: center; gap: 4px; flex: 1; }
.settle-avatar { width: 40px; height: 40px; border-radius: 50%; }
.settle-name { font-size: 13px; font-weight: 500; color: var(--foreground); }
.settle-arrow { display: flex; flex-direction: column; align-items: center; gap: 2px; flex-shrink: 0; }
.settle-amount { font-size: 16px; font-weight: 700; color: var(--danger); }
.settle-arrow-icon { font-size: 20px; color: var(--foreground-muted); }
.settle-empty { text-align: center; color: var(--foreground-muted); padding: 20px; }

.avatar-upload-wrap { display: flex; flex-direction: column; align-items: center; margin: 24px 0; }
.avatar-upload { width: 96px; height: 96px; border-radius: 50%; overflow: hidden; box-shadow: 0 8px 32px oklch(var(--shadow-color) / 0.15); border: 3px solid var(--surface); transition: transform 0.2s; }
.avatar-upload:active { transform: scale(0.95); }
.preview-avatar { width: 100%; height: 100%; object-fit: cover; }
.avatar-tip { font-size: 13px; color: var(--foreground-muted); margin-top: 12px; font-weight: 500; }

.quick-numbers { display: flex; gap: 10px; margin: 16px 0 20px; }
.quick-num-btn { flex: 1; height: 48px; background: linear-gradient(145deg, oklch(0.97 0.02 240) 0%, oklch(0.93 0.03 240) 100%); border-radius: 14px; display: flex; align-items: center; justify-content: center; font-size: 18px; font-weight: 700; color: var(--foreground); cursor: pointer; border: 1px solid oklch(0.9 0.02 240); transition: all 0.15s; }
.quick-num-btn:active { transform: scale(0.92); background: var(--primary); color: var(--primary-foreground); }

@keyframes slideUp { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
.popup { animation: slideUp 0.3s ease-out; }
.members-area, .logs-area { animation: fadeIn 0.3s ease-out; }
</style>
