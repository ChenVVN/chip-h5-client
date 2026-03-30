<template>
  <van-popup v-model:show="visible" round position="bottom">
    <div class="popup">
      <div class="reclaim-header">
        <h3>收回积分</h3>
        <span class="reclaim-all-text" @click="handleReclaimAll">全收</span>
      </div>
      <p class="hint">桌面积分: {{ deskScore }}</p>
      <div class="input-wrap">
        <van-field v-model="amount" type="digit" maxlength="5" placeholder="请输入积分数量" />
      </div>
      <van-button type="success" size="large" round @click="handleReclaim" block :loading="loading">确认收回</van-button>
    </div>
  </van-popup>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  loading: Boolean,
  deskScore: Number
})

const emit = defineEmits(['update:show', 'reclaim', 'reclaim-all'])

const visible = ref(props.show)
const amount = ref('')

watch(() => props.show, (val) => {
  visible.value = val
  if (!val) amount.value = ''
})

watch(visible, (val) => {
  emit('update:show', val)
})

function handleReclaim() {
  const val = parseInt(amount.value)
  if (!val || val <= 0) return
  emit('reclaim', val)
}

function handleReclaimAll() {
  emit('reclaim-all')
}
</script>

<style scoped>
.popup {
  padding: 32px 24px 28px;
  background: #20201f;
  border-radius: 32px 32px 0 0;
}

.reclaim-header {
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  margin-bottom: 8px;
}

.reclaim-header h3 {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 24px;
  text-align: center;
  font-weight: 700;
  color: #e5e2e1;
  letter-spacing: -0.02em;
}

.reclaim-all-text {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 14px;
  color: #e9c176;
  font-weight: 700;
  cursor: pointer;
}

.reclaim-all-text:active {
  opacity: 0.8;
  transform: translateY(-50%) scale(0.95);
}

.hint {
  text-align: center;
  color: #d1c5b4;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 12px;
  margin-bottom: 16px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
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

/* Confirm button - gradient */
:deep(.van-button--success) {
  background: linear-gradient(135deg, #98d3ba 0%, #e9c176 100%) !important;
  border: none !important;
  height: 64px;
  border-radius: 16px;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: #261900;
  letter-spacing: 0.02em;
  box-shadow: 0 8px 24px rgba(152, 211, 186, 0.2);
}

:deep(.van-button--success:active) {
  transform: scale(0.98);
}
</style>