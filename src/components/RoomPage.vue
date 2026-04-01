<template>
  <div class="page-room">
    <!-- Room Header -->
    <div class="room-header">
      <div class="header-left" @click="$emit('go-home')">
        <span class="material-symbols-outlined">arrow_back</span>
      </div>
      <div class="room-code">房间号 {{ room?.roomCode }}</div>
      <div class="header-right">
        <button
          class="refresh-btn"
          @click="$emit('refresh')"
          :disabled="refreshing"
        >
          <span v-if="!refreshing">刷新</span>
          <span v-else>...</span>
        </button>
      </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
      <!-- Left Sidebar: Members -->
      <aside class="members-sidebar">
      <div class="member-title">成员</div>
        <div class="member-list">
          <div
            v-for="member in room?.members"
            :key="member.odid"
            class="member-item"
          >
            <div class="member-avatar-wrap">
              <img :src="member.avatar || defaultAvatar" class="member-avatar" />
            </div>
            <span class="member-name">{{ member.nickname }}</span>
            <span class="member-score" :class="member.personalScore >= 0 ? 'positive' : 'negative'">
              {{ member.personalScore >= 0 ? '+' : '' }}{{ member.personalScore }}
            </span>
          </div>
        </div>
      </aside>

      <!-- Right Content -->
      <main class="right-content">
        <!-- Score Cards -->
       <div class="score-block">
          <div class="desk-score">
            <div class="score-label">
              <div>桌面积分</div>
              <button class="reclaim-all-btn" @click="$emit('reclaim-all')">全收</button>
            </div>
            <div class="score-value">{{ room?.deskScore || 0 }}</div>
          </div>
          <div class="mine-score">
            <div class="score-label">我的积分</div>
            <div class="score-value">{{ myScore }}</div>
          </div>
        </div>

        <!-- Logs Area -->
        <div class="logs-area">
          <div class="area-title-row">
            <div class="area-title">房间动态</div>
            <button class="settle-btn" @click="$emit('show-settle')">结算方案</button>
          </div>
          <div class="log-list" v-if="room?.logs?.length > 0">
            <div v-for="(log, index) in room.logs" :key="log.timestamp || index" class="log-item" :class="{ 'log-new': index === 0 && room.logs.length > 0 }">
              <div class="log-left">
                <span class="material-symbols-outlined log-icon" :class="getLogIconClass(log.action)">{{ getLogIcon(log.action) }}</span>
                <span class="log-text" :class="getLogTextClass(log.action)">
                  <strong>{{ log.nickname }}</strong>
                  <template v-if="log.action === '支出'">支出了</template>
                  <template v-else-if="log.action === '收回'">收回了</template>
                  <template v-else-if="log.action === '加入'">加入了房间</template>
                  <template v-else-if="log.action === '返回'">返回了房间</template>
                  <template v-else-if="log.action === '离开'">离开了房间</template>
                  <span v-if="log.action === '支出'" class="amount-spend">{{ log.amount }}</span>
                  <span v-else-if="log.action === '收回'" class="amount-reclaim">{{ log.amount }}</span>
                  <template v-if="log.action === '支出' || log.action === '收回'"> 积分</template>
                </span>
              </div>
              <span class="log-time">{{ formatTime(log.timestamp) }}</span>
            </div>
          </div>
          <div v-else class="no-logs">暂无记录</div>
        </div>
      </main>
    </div>

    <!-- Bottom Actions -->
    <div class="room-actions">
      <van-button type="danger" round @click="$emit('show-spend')">支出</van-button>
      <van-button type="success" round @click="$emit('show-reclaim')">收回</van-button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  room: Object,
  myScore: Number,
  refreshing: Boolean
})

defineEmits(['go-home', 'refresh', 'show-settle', 'show-spend', 'show-reclaim', 'reclaim-all'])

const defaultAvatar = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="35" r="25" fill="%23bdc3c7"/><circle cx="50" cy="100" r="40" fill="%23bdc3c7"/></svg>'

function formatTime(timestamp) {
  if (!timestamp) return ''
  const date = new Date(timestamp)
  return `${date.getHours().toString().padStart(2,'0')}:${date.getMinutes().toString().padStart(2,'0')}:${date.getSeconds().toString().padStart(2,'0')}`
}

function getLogIcon(action) {
  if (action === '支出') return 'payments'
  if (action === '收回') return 'add_circle'
  if (action === '加入') return 'person_add'
  if (action === '返回') return 'login'
  if (action === '离开') return 'logout'
  return 'chat'
}

function getLogIconClass(action) {
  if (action === '支出') return 'icon-spend'
  if (action === '收回') return 'icon-reclaim'
  if (action === '加入') return 'icon-join'
  if (action === '返回') return 'icon-return'
  if (action === '离开') return 'icon-leave'
  return ''
}

function getLogTextClass(action) {
  if (action === '支出') return 'log-spend'
  if (action === '收回') return 'log-reclaim'
  if (action === '加入') return 'log-join'
  if (action === '返回') return 'log-return'
  if (action === '离开') return 'log-leave'
  return ''
}
</script>

<style scoped>
.page-room {
  height: 100vh;
  height: 100dvh;
  padding-bottom: 90px;
  background: #131313;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* Room Header */
.room-header {
  background: #131313;
  padding: 12px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid rgba(233, 193, 118, 0.1);
  flex-shrink: 0;
}

.header-left {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #e9c176;
  font-size: 18px;
  cursor: pointer;
  border-radius: 8px;
}

.header-left:active {
  background: rgba(233, 193, 118, 0.1);
}

.header-right {
  width: 50px;
  display: flex;
  justify-content: flex-end;
}

.room-code {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: #e9c176;
  letter-spacing: -0.02em;
}

.refresh-btn {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 10px;
  font-weight: 600;
  color: #e9c176;
  background: rgba(233, 193, 118, 0.1);
  border: 1px solid rgba(233, 193, 118, 0.2);
  min-width: 44px;
  height: 26px;
  border-radius: 14px;
  cursor: pointer;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.refresh-btn:active {
  background: rgba(233, 193, 118, 0.2);
}

/* Main Content */
.main-content {
  flex: 1;
  display: flex;
  overflow: hidden;
}

/* Members Sidebar (Left) */
.members-sidebar {
  width: 100px;
  background: #1c1b1b;
  padding: 8px 12px;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}

.member-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  align-items: center;
}

.member-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 8px 8px;
  background: #232323;
  border-radius: 12px;
  width: 100%;
    border: 1px solid #343030;
    box-shadow: 5px 4px 3px 0px;
}

.member-avatar-wrap {
  position: relative;
}

.member-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  object-fit: cover;
  border: 2px solid rgba(233, 193, 118, 0.3);
}

.online-dot {
  position: absolute;
  bottom: 0;
  right: 0;
  width: 10px;
  height: 10px;
  background: #98d3ba;
  border-radius: 50%;
  border: 2px solid #1c1b1b;
}

.member-name {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 10px;
  font-weight: 600;
  color: #ffffff;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.member-score {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 11px;
  font-weight: 700;
}

.member-score.positive {
  color: #98d3ba;
}

.member-score.negative {
  color: #ffb4ab;
}

/* Right Content */
.right-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 16px;
  gap: 16px;
  overflow-y: auto;
}

/* Score Cards */
.score-cards {
  display: flex;
  gap: 12px;
}

.score-block {
  background: #3d3a36;
  padding: 20px 24px;
  border-radius: 16px;
  border: 1px solid rgba(255, 220, 130, 0.3);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  .desk-score {
    border-bottom: 1px solid rgba(255, 220, 130, 0.2);
    padding-bottom: 12px;
    margin-bottom: 12px;
    .score-label {
      display: flex;
      justify-content: space-between;
      align-items: center;
      color: #e8d9b8;
      font-size: 12px;
      text-transform: uppercase;
      letter-spacing: 0.08em;
    }
    .score-value {
      color: #ffdd88;
      font-size: 44px;
      font-weight: 700;
      margin-top: 6px;
      text-shadow: 0 2px 10px rgba(255, 210, 100, 0.3);
    }
  }
  .mine-score {
    .score-label {
      color: #e8d9b8;
      font-size: 12px;
      text-transform: uppercase;
      letter-spacing: 0.08em;
    }
    .score-value {
      color: #6ee7a0;
      font-size: 38px;
      font-weight: 700;
      text-shadow: 0 2px 10px rgba(110, 231, 160, 0.25);
    }
  }
}

.score-card {
  flex: 1;
  background: #1c1b1b;
  border-radius: 20px;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.score-card.mine {
  background: #ffdea5;
}

.score-card.mine .score-label {
  color: rgba(65, 45, 0, 0.6);
}

.score-card.mine .score-value {
  color: #261900;
  text-shadow: none;
}

.score-label {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 10px;
  font-weight: 600;
  color: #d1c5b4;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.score-value {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 32px;
  font-weight: 800;
  color: #e9c176;
  letter-spacing: -0.02em;
}

.reclaim-all-btn {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 10px;
  font-weight: 700;
  color: #98d3ba;
  background: rgba(152, 211, 186, 0.15);
  border: 1px solid rgba(152, 211, 186, 0.4);
  padding: 6px 14px;
  border-radius: 12px;
  cursor: pointer;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  transition: all 150ms ease-out;
}

.reclaim-all-btn:active {
  background: rgba(152, 211, 186, 0.3);
  transform: scale(0.96);
}

/* Logs Area */
.logs-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
}

.area-title-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 12px;
}
.member-title {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: #e5e2e1;
  margin-bottom:8px;
}

.area-title {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: #e5e2e1;
}

.settle-btn {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 11px;
  font-weight: 600;
  color: #e9c176;
  cursor: pointer;
  padding: 6px 12px;
  background: rgba(233, 193, 118, 0.1);
  border-radius: 16px;
  border: 1px solid rgba(233, 193, 118, 0.2);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.settle-btn:active {
  background: rgba(233, 193, 118, 0.2);
}

.log-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  flex: 1;
  overflow-y: auto;
max-height: 600px;
}

.log-item {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
  font-size: 13px;
  padding: 12px;
  background: #1c1b1b;
  border-radius: 12px;
}

.log-left {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  flex: 1;
  flex-wrap: wrap;
  min-width: 0;
}

.log-icon {
  font-size: 18px;
  flex-shrink: 0;
}

.log-icon.icon-spend { color: #ffb4ab; }
.log-icon.icon-reclaim { color: #98d3ba; }
.log-icon.icon-join { color: #5c9ef5; }
.log-icon.icon-return { color: #5c9ef5; }
.log-icon.icon-leave { color: #9a8f80; }

.log-text {
  color: #d1c5b4;
  flex: 1;
  display: flex;
  align-items: flex-start;
  gap: 4px;
  flex-wrap: wrap;
  word-break: break-word;
}

.log-text strong {
  color: #e5e2e1;
  font-weight: 600;
}

.amount-spend {
  color: #ffb4ab;
  font-weight: 700;
  font-family: 'Manrope', system-ui, sans-serif;
}

.amount-reclaim {
  color: #98d3ba;
  font-weight: 700;
  font-family: 'Manrope', system-ui, sans-serif;
}

.log-text.log-join,
.log-text.log-return,
.log-text.log-leave {
  color: #9a8f80;
}

.log-time {
  font-family: 'Inter', system-ui, sans-serif;
  color: #9a8f80;
  font-size: 11px;
  flex-shrink: 0;
}

.no-logs {
  text-align: center;
  color: #9a8f80;
  padding: 32px;
}

/* Room Actions - Bottom navigation */
.room-actions {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  justify-content: space-around;
  align-items: center;
  padding: 16px 32px 24px;
  background: rgba(14, 14, 14, 0.95);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border-top: 1px solid rgba(233, 193, 118, 0.1);
}

.room-actions .van-button {
  flex: 1;
  max-width: 160px;
  height: 52px;
  border-radius: 26px;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 14px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

:deep(.van-button--danger) {
  background: #e9c176 !important;
  border: none !important;
  color: #412d00 !important;
}

:deep(.van-button--danger:active) {
  transform: scale(0.95);
}

:deep(.van-button--success) {
  background: #1c1b1b !important;
  border: 1px solid #98d3ba !important;
  color: #98d3ba !important;
}

:deep(.van-button--success:active) {
  transform: scale(0.95);
}

/* 新日志动画 */
@keyframes logFadeIn {
  0% {
    opacity: 0;
    transform: translateY(-10px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

.log-item.log-new {
  animation: logFadeIn 0.4s ease-out forwards;
}

/* Material Symbols */
.material-symbols-outlined {
  font-variation-settings: 'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24;
}
</style>