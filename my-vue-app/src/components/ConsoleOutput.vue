<script setup>
import { computed } from 'vue'

const props = defineProps({
  result: {
    type: Object,
    default: () => ({ exit_code: 0, stdout: '', stderr: '' }),
  },
  executionTime: {
    type: Number,
    default: null,
  },
  showResult: {
    type: Boolean,
    default: false,
  },
})

const isSuccess = computed(() => props.result.exit_code === 0)
const output = computed(() => (isSuccess.value ? props.result.stdout : props.result.stderr))
</script>

<template>
  <div class="output-box">
    <div class="steps">
      <span>코드 실행 시작</span>
      <span>Initializing environment</span>
      <span>Installing packages</span>
      <span>Running code</span>
    </div>
    <template v-if="showResult">
      <div class="output-text" :class="isSuccess ? 'success' : 'error'">
        {{ output || '(출력 없음)' }}
      </div>
      <div v-if="executionTime !== null" class="exec-time">
        실행 시간: {{ executionTime }} ms
      </div>
    </template>
  </div>
</template>

<style scoped>
.output-box {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  width: 100%;
  box-sizing: border-box;
}

.steps {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 12px 16px 8px 16px;
  font-size: 13px;
  color: #bbb;
}

.output-text {
  font-size: 15px;
  line-height: 1.7;
  white-space: pre-wrap;
  word-break: break-word;
  padding: 12px 16px;
}

.exec-time {
  font-size: 13px;
  color: #999;
  padding: 0 16px 10px 16px;
}

.success {
  color: #1a1a1a;
}

.error {
  color: #c0392b;
}
</style>
