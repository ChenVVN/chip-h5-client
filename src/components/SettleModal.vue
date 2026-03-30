<template>
  <van-popup v-model:show="visible" round closeable class="settle-popup">
    <div class="settle-content">
      <h3>结算方案</h3>
      <div class="settle-list" v-if="plan.length > 0">
        <div v-for="(item, idx) in plan" :key="idx" class="settle-item">
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
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  plan: {
    type: Array,
    default: () => []
  }
})

const emit = defineEmits(['update:show'])

const visible = ref(props.show)
const defaultAvatar = 'data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="35" r="25" fill="%23bdc3c7"/><circle cx="50" cy="100" r="40" fill="%23bdc3c7"/></svg>'

watch(() => props.show, (val) => {
  visible.value = val
})

watch(visible, (val) => {
  emit('update:show', val)
})
</script>

<style scoped>
/* Popup overlay */
:deep(.van-popup) {
  background: #20201f !important;
  border-radius: 32px 32px 0 0;
  max-height: 70vh;
}

:deep(.van-popup--bottom) {
  padding-bottom: env(safe-area-inset-bottom);
}

.settle-content {
  padding: 40px 24px 28px;
}

.settle-content h3 {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 24px;
  text-align: center;
  margin-bottom: 24px;
  font-weight: 700;
  color: #e5e2e1;
  letter-spacing: -0.02em;
}

.settle-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.settle-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 16px;
  background: rgba(14, 14, 14, 0.5);
  border-radius: 16px;
  border: 1px solid rgba(233, 193, 118, 0.1);
}

.settle-from,
.settle-to {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  flex: 1;
}

.settle-avatar {
  width: 48px;
  height: 48px;
  border-radius: 14px;
  object-fit: cover;
  border: 2px solid rgba(233, 193, 118, 0.2);
}

.settle-name {
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 13px;
  font-weight: 600;
  color: #e5e2e1;
}

.settle-arrow {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  flex-shrink: 0;
}

.settle-amount {
  font-family: 'Manrope', system-ui, sans-serif;
  font-size: 18px;
  font-weight: 700;
  color: #ffb4ab;
}

.settle-arrow-icon {
  font-size: 20px;
  color: #9a8f80;
}

.settle-empty {
  text-align: center;
  color: #9a8f80;
  padding: 32px 20px;
  font-size: 14px;
}
</style>