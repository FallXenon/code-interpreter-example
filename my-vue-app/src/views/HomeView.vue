<script setup>
import { ref, reactive } from 'vue'
import ConsoleOutput from '../components/ConsoleOutput.vue'

const rawInput = ref(`{
  "success": true,
  "output": "ConsoleOutput(exit_code=1, stderr='/sandbox/f76f07fcefaa49b9ba15ae3e4122db10.js:1\\\\nconsole.lo(\\"Hello World\\")\\\\n ^\\\\n\\\\nTypeError: console.lo is not a function\\\\n at Object.<anonymous> (/sandbox/f76f07fcefaa49b9ba15ae3e4122db10.js:1:9)\\\\n', stdout='')",
  "error": null,
  "execution_time": 0.10472531989216805,
  "timestamp": "2026-05-08T10:41:06.672275"
}`)

const parseError = ref('')
const hasResult = ref(false)

const result = reactive({
  exit_code: 0,
  stdout: '',
  stderr: '',
})

const meta = reactive({
  execution_time: null,
  timestamp: null,
})

function parseConsoleOutput(str) {
  const trimmed = str.trim()
  if (!trimmed.startsWith('ConsoleOutput(')) {
    throw new Error("output 필드가 'ConsoleOutput('으로 시작해야 합니다.")
  }

  const inner = trimmed.slice('ConsoleOutput('.length, -1)

  const exitMatch = inner.match(/exit_code\s*=\s*(-?\d+)/)
  if (!exitMatch) throw new Error('exit_code를 찾을 수 없습니다.')

  function extractField(src, key) {
    const re = new RegExp(key + `\\s*=\\s*'((?:[^'\\\\]|\\\\.)*)'`)
    const reDouble = new RegExp(key + `\\s*=\\s*"((?:[^"\\\\]|\\\\.)*)"`)
    const m = src.match(re) || src.match(reDouble)
    if (!m) return ''
    return m[1]
      .replace(/\\n/g, '\n')
      .replace(/\\t/g, '\t')
      .replace(/\\\\/g, '\\')
      .replace(/\\'/g, "'")
      .replace(/\\"/g, '"')
  }

  return {
    exit_code: parseInt(exitMatch[1], 10),
    stdout: extractField(inner, 'stdout'),
    stderr: extractField(inner, 'stderr'),
  }
}

function formatTimestamp(ts) {
  if (!ts) return ''
  const d = new Date(ts)
  if (isNaN(d)) return ts
  return d.toLocaleString('ko-KR', {
    year: 'numeric', month: '2-digit', day: '2-digit',
    hour: '2-digit', minute: '2-digit', second: '2-digit',
    hour12: false,
  })
}

function apply() {
  parseError.value = ''

  if (!rawInput.value.trim()) {
    hasResult.value = false
    return
  }

  try {
    const json = JSON.parse(rawInput.value)
    if (!json.output) {
      hasResult.value = false
      return
    }

    const parsed = parseConsoleOutput(json.output)
    result.exit_code = parsed.exit_code
    result.stdout = parsed.stdout
    result.stderr = parsed.stderr

    meta.execution_time = json.execution_time != null
      ? json.execution_time
      : null
    meta.timestamp = json.timestamp || null
    hasResult.value = true
  } catch (e) {
    hasResult.value = false
  }
}

apply()
</script>

<template>
  <main class="page">
    <h1 class="title">Console Output Viewer</h1>

    <div class="input-section">
      <label class="input-label">JSON 값 붙여넣기</label>
      <textarea
        v-model="rawInput"
        class="raw-input"
        rows="8"
        placeholder='{ "output": "ConsoleOutput(...)", "execution_time": 0.1, "timestamp": "..." }'
        spellcheck="false"
        @keydown.ctrl.enter="apply"
        @keydown.meta.enter="apply"
      ></textarea>
      <div class="input-footer">
        <span class="hint">Ctrl+Enter 또는 버튼으로 적용</span>
        <button class="btn apply" @click="apply">적용</button>
      </div>
      <div v-if="parseError" class="parse-error">파싱 오류: {{ parseError }}</div>
    </div>

    <div class="output-section">
      <ConsoleOutput :result="result" :execution-time="meta.execution_time" :show-result="hasResult" />
    </div>
  </main>
</template>

<style scoped>
.page {
  width: 760px;
  margin: 0 auto;
  padding: 40px 20px;
  box-sizing: border-box;
}

.title {
  font-size: 24px;
  font-weight: 700;
  margin-bottom: 28px;
}

.input-section {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 32px;
}

.input-label {
  font-size: 14px;
  font-weight: 600;
  color: #444;
}

.raw-input {
  width: 100%;
  padding: 12px;
  font-family: 'Courier New', Courier, monospace;
  font-size: 13px;
  border: 1px solid #ccc;
  border-radius: 8px;
  resize: vertical;
  box-sizing: border-box;
  line-height: 1.6;
}

.raw-input:focus {
  outline: none;
  border-color: #3b5bdb;
  box-shadow: 0 0 0 3px rgba(59, 91, 219, 0.15);
}

.input-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.hint {
  font-size: 12px;
  color: #999;
}

.btn.apply {
  padding: 8px 24px;
  background-color: #3b5bdb;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
}

.btn.apply:hover {
  opacity: 0.85;
}

.parse-error {
  padding: 10px 14px;
  background-color: #fff0f0;
  border: 1px solid #e74c3c;
  border-radius: 6px;
  color: #c0392b;
  font-size: 13px;
}

.output-section {
  display: flex;
  flex-direction: column;
}
</style>
