<template>
  <van-popup v-model:show="visible" round position="bottom">
    <div class="popup">
      <h3>加入房间</h3>
      <van-field v-model="code" type="digit" maxlength="6" placeholder="请输入6位房间号" />
      <van-button type="primary" size="large" round @click="handleJoin" block :loading="loading">加入</van-button>
    </div>
  </van-popup>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  loading: Boolean
})

const emit = defineEmits(['update:show', 'join'])

const visible = ref(props.show)
const code = ref('')

watch(() => props.show, (val) => {
  visible.value = val
})

watch(visible, (val) => {
  emit('update:show', val)
})

function handleJoin() {
  if (!code.value || code.value.length !== 6) return
  emit('join', code.value)
}
</script>

<style scoped>
.popup {
  padding: 28px 20px;
  background: #20201f;
  border-radius: 24px 24px 0 0;
}

.popup h3 {
  font-size: 20px;
  text-align: center;
  margin-bottom: 24px;
  font-weight: 700;
  color: #e5e2e1;
  letter-spacing: -0.01em;
}

/* Input field */
:deep(.van-field) {
  background: #20201f !important;
  border-radius: 12px;
  padding: 16px 20px;
}

:deep(.van-field__control) {
  color: #e5e2e1 !important;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 500;
  padding: 0 5px;
}

:deep(.van-field__control::placeholder) {
  color: rgba(154, 143, 128, 0.5) !important;
}

:deep(.van-field__control) {
  padding-right: 60px !important;
}
:deep(.van-field__body) {
      border-bottom: 1px #5d5d5d solid;
}
</style>