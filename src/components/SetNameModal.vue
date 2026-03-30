<template>
  <van-popup v-model:show="visible" round position="bottom">
    <div class="popup">
      <h3>设置昵称</h3>
      <van-field v-model="nickname" placeholder="请输入昵称" maxlength="12" />
      <div class="avatar-upload-wrap">
        <van-uploader :after-read="onAvatarRead" accept="image/*">
          <div class="avatar-upload">
            <img :src="avatar || defaultAvatar" class="preview-avatar" />
          </div>
        </van-uploader>
        <span class="avatar-tip">点击更换头像</span>
      </div>
      <van-button
        type="primary"
        size="large"
        round
        @click="handleConfirm"
        :loading="loading"
        block
      >
        保存
      </van-button>
    </div>
  </van-popup>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  loading: Boolean,
  initialNickname: String,
  initialAvatar: String
})

const emit = defineEmits(['update:show', 'confirm'])

const visible = ref(props.show)
const nickname = ref(props.initialNickname || '')
const avatar = ref(props.initialAvatar || '')
const defaultAvatar = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="35" r="25" fill="%23bdc3c7"/><circle cx="50" cy="100" r="40" fill="%23bdc3c7"/></svg>'

watch(() => props.show, (val) => {
  visible.value = val
  if (val) {
    nickname.value = props.initialNickname || ''
    avatar.value = props.initialAvatar || ''
  }
})

watch(visible, (val) => {
  emit('update:show', val)
})

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
    avatar.value = canvas.toDataURL('image/jpeg', 0.7)
  }
  img.src = file.content
}

function handleConfirm() {
  const name = nickname.value.trim()
  if (!name) return
  emit('confirm', { nickname: name, avatar: avatar.value })
}
</script>

<style scoped>
.popup {
  padding: 40px 24px 28px;
  background: #20201f;
  border-radius: 32px 32px 0 0;
}

.popup h3 {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 24px;
  text-align: center;
  margin-bottom: 32px;
  font-weight: 700;
  color: #e5e2e1;
  letter-spacing: -0.02em;
}

.avatar-upload-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 32px;
}

.avatar-upload {
  width: 96px;
  height: 96px;
  border-radius: 50%;
  padding: 3px;
  background: linear-gradient(135deg, rgba(233, 193, 118, 0.4), rgba(152, 211, 186, 0.4));
  overflow: hidden;
  transition: transform 0.2s;
  cursor: pointer;
}

.avatar-upload:active {
  transform: scale(0.95);
}

.avatar-upload-inner {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  overflow: hidden;
  background: #20201f;
  border: 2px solid #20201f;
}

.preview-avatar {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.avatar-tip {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 12px;
  color: #d1c5b4;
  margin-top: 12px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

/* Input field */
:deep(.van-field) {
  background: #20201f !important;
  border-radius: 12px;
  padding: 16px 20px;
  margin-bottom: 24px;
}

:deep(.van-field__control) {
  color: #e5e2e1 !important;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 18px;
  text-align: center;
}

:deep(.van-field__control::placeholder) {
  color: rgba(154, 143, 128, 0.5) !important;
}

/* Save button */
:deep(.van-button--primary) {
  background: #e9c176 !important;
  border: none !important;
  height: 56px;
  border-radius: 16px;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 16px;
  font-weight: 700;
  color: #412d00;
  letter-spacing: 0.02em;
  box-shadow: 0 8px 24px rgba(233, 193, 118, 0.15);
}

:deep(.van-button--primary:active) {
  transform: scale(0.98);
}
</style>