<template>
  <van-popup v-model:show="visible" round position="bottom">
    <div class="popup">
      <h3>支出积分</h3>
      <p class="hint">我的积分: {{ myScore }}</p>
      <div class="input-wrap">
        <van-field v-model="amount" type="digit" maxlength="5" placeholder="请输入积分数量" />
      </div>
      <div class="quick-numbers">
        <button v-for="n in 5" :key="n" class="quick-num-btn" @click="quickSpend(n)">{{ n }}</button>
      </div>
      <van-button type="danger" size="large" round @click="handleSpend" block :loading="loading">确认支出</van-button>
    </div>
  </van-popup>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  loading: Boolean,
  myScore: Number
})

const emit = defineEmits(['update:show', 'spend'])

const visible = ref(props.show)
const amount = ref('')

watch(() => props.show, (val) => {
  visible.value = val
  if (!val) amount.value = ''
})

watch(visible, (val) => {
  emit('update:show', val)
})

function quickSpend(n) {
  amount.value = n.toString()
  handleSpend()
}

function handleSpend() {
  const val = parseInt(amount.value)
  if (!val || val <= 0) return
  emit('spend', val)
}
</script>

<style scoped>
.popup {
  padding: 32px 24px 28px;
  background: #20201f;
  border-radius: 32px 32px 0 0;
}

.popup h3 {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 24px;
  text-align: center;
  margin-bottom: 8px;
  font-weight: 700;
  color: #e5e2e1;
  letter-spacing: -0.02em;
}

.hint {
  text-align: center;
  color: #e9c176;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 12px;
  margin-bottom: 24px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

/* Input field */
:deep(.van-field) {
  background: #20201f !important;
  border-radius: 12px;
  padding: 16px 20px;
  height: 64px;
}

:deep(.van-field__control) {
  color: #e5e2e1 !important;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 500;
  padding: 0 5px;
}
:deep(.van-field__body) {
      border-bottom: 1px #5d5d5d solid;
}

:deep(.van-field__control::placeholder) {
  color: rgba(154, 143, 128, 0.5) !important;
}

/* Quick numbers - horizontal */
.quick-numbers {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  margin: 16px 0 24px;
}

.quick-num-btn {
  flex: 1;
  height: 48px;
  background: #353535;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: #e5e2e1;
  cursor: pointer;
  border: none;
  transition: all 0.15s;
}

.quick-num-btn:hover {
  background: #393939;
}

.quick-num-btn:active {
  transform: scale(0.9);
  background: #e9c176;
  color: #261900;
}

/* Confirm button */
:deep(.van-button--danger) {
  background: #e9c176 !important;
  border: none !important;
  height: 64px;
  border-radius: 16px;
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: #412d00;
  letter-spacing: 0.02em;
  box-shadow: 0 8px 24px rgba(233, 193, 118, 0.15);
}

:deep(.van-button--danger:active) {
  transform: scale(0.98);
}
</style>