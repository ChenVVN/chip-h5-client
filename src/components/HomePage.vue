<template>
  <div class="page-home">
    <!-- Header -->
    <header class="home-header">
      <div class="logo-wrap">
        <span class="material-symbols-outlined logo-icon">database</span>
      </div>
      <h1 class="title">筹码平台</h1>
    </header>

    <main class="home-main">
      <!-- User Card -->
      <section class="user-section">
        <div class="user-card" @click="$emit('edit-profile')">
          <button class="edit-btn">
            <span class="material-symbols-outlined">edit</span>
          </button>
          <div class="avatar-wrap">
            <img :src="user?.avatar || defaultAvatar" class="avatar" />
          </div>
          <h1 class="nickname">{{ user?.nickname || '未设置昵称' }}</h1>
        </div>
      </section>

      <!-- Actions -->
      <div class="actions-wrap">
        <!-- Return to Room (if has last room) -->
        <button v-if="lastRoomCode" class="return-room-btn" @click="$emit('return-room')">
          <div class="return-room-content">
            <div class="icon-wrap">
              <span class="material-symbols-outlined">meeting_room</span>
            </div>
            <div class="text-wrap">
              <span class="room-code">返回房间 {{ lastRoomCode }}</span>
            </div>
            <span class="material-symbols-outlined arrow">arrow_forward_ios</span>
          </div>
        </button>

        <!-- Create/Join Buttons -->
        <div class="btn-grid">
          <button class="action-btn create" @click="$emit('create-room')">
            <span class="material-symbols-outlined">add_circle</span>
            <span>创建房间</span>
          </button>
          <button class="action-btn join" @click="$emit('show-join')">
            <span class="material-symbols-outlined">login</span>
            <span>加入房间</span>
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
defineProps({
  user: Object,
  lastRoomCode: String
})

defineEmits(['edit-profile', 'return-room', 'create-room', 'show-join'])

const defaultAvatar = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="35" r="25" fill="%23bdc3c7"/><circle cx="50" cy="100" r="40" fill="%23bdc3c7"/></svg>'
</script>

<style scoped>
.page-home {
  min-height: 100vh;
  background: #131313;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 24px;
}

.home-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 48px;
  margin-top: 20px;
}

.logo-wrap {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  background: #353535;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 0 15px rgba(233, 193, 118, 0.1);
  border: 1px solid rgba(233, 193, 118, 0.2);
  margin-bottom: 16px;
}

.logo-icon {
  font-size: 28px;
  color: #e9c176;
  font-variation-settings: 'FILL' 1;
}

.title {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 36px;
  font-weight: 800;
  color: #e9c176;
  letter-spacing: -0.02em;
  text-shadow: 0 0 15px rgba(233, 193, 118, 0.3);
}

.home-main {
  width: 100%;
  max-width: 400px;
}

.user-section {
  margin-bottom: 48px;
}

.user-card {
  position: relative;
  padding: 32px;
  border-radius: 24px;
  background: #1c1b1b;
  border: 1px solid rgba(233, 193, 118, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  transition: all 0.3s;
}

.user-card:hover {
  border-color: rgba(233, 193, 118, 0.3);
}

.edit-btn {
  position: absolute;
  top: 16px;
  right: 16px;
  width: 40px;
  height: 40px;
  border-radius: 12px;
  background: rgba(233, 193, 118, 0.1);
  color: #e9c176;
  display: flex;
  align-items: center;
  justify-content: center;
  border: none;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.edit-btn:hover {
  background: #e9c176;
  color: #261900;
}

.avatar-wrap {
  width: 128px;
  height: 128px;
  border-radius: 16px;
  overflow: hidden;
  background: #20201f;
  box-shadow: 0 20px 40px -12px rgba(0, 0, 0, 0.5);
  border: 4px solid #2a2a2a;
}

.avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.nickname {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 28px;
  font-weight: 800;
  color: #e5e2e1;
  letter-spacing: -0.02em;
}

.actions-wrap {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* Return Room Button with Gradient Border */
.return-room-btn {
  position: relative;
  width: 100%;
  padding: 2px;
  border-radius: 12px;
  background: linear-gradient(135deg, #e9c176 0%, #c5a059 100%);
  border: none;
  cursor: pointer;
  overflow: hidden;
}

.return-room-btn:active {
  transform: scale(0.98);
}

.return-room-content {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 20px 20px 20px 24px;
  background: #e9c176;
  border-radius: 10px;
  transition: all 0.2s;
}

.return-room-content .arrow {
  color: #0e0e0e;
  font-size: 18px;
  margin-left: auto;
}

.icon-wrap {
  width: 44px;
  height: 44px;
  border-radius: 10px;
  background: rgba(233, 193, 118, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.icon-wrap .material-symbols-outlined {
  font-size: 22px;
  color: #0e0e0e;
  font-variation-settings: 'FILL' 1;
}

.text-wrap {
  display: flex;
  flex-direction: column;
  text-align: left;
  flex: 1;
}

.text-wrap .room-code {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: #0e0e0e;
  text-shadow: 0 0 15px rgba(233, 193, 118, 0.3);
}

/* Action Buttons Grid */
.btn-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

.action-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 12px;
  padding: 24px;
  border-radius: 16px;
  background: #20201f;
  border: none;
  cursor: pointer;
  transition: all 0.2s;
  color: #e5e2e1;
}

.action-btn:hover {
  background: #2a2a2a;
}

.action-btn:active {
  transform: scale(0.95);
}

.action-btn .material-symbols-outlined {
  font-size: 24px;
  color: #e9c176;
}

.action-btn span:last-child {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 16px;
  font-weight: 700;
  color: #e9c176;
}

/* Material Symbols */
.material-symbols-outlined {
  font-variation-settings: 'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24;
}
</style>