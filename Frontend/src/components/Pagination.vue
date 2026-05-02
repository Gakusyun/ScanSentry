<script setup lang="ts">
import { computed } from 'vue'

const props = defineProps<{
  page: number
  total: number
  pageSize: number
}>()

const emit = defineEmits<{
  'update:page': [page: number]
  'update:pageSize': [size: number]
}>()

const totalPages = computed(() => Math.ceil(props.total / props.pageSize))

function prev() {
  if (props.page > 1) emit('update:page', props.page - 1)
}

function next() {
  if (props.page < totalPages.value) emit('update:page', props.page + 1)
}

function goToPage(e: Event) {
  const input = e.target as HTMLInputElement
  const p = parseInt(input.value)
  if (p >= 1 && p <= totalPages.value) {
    emit('update:page', p)
    input.value = ''
  }
}

function changePageSize(e: Event) {
  const select = e.target as HTMLSelectElement
  emit('update:pageSize', parseInt(select.value))
  emit('update:page', 1)
}
</script>

<template>
  <div class="pager">
    <button @click="prev" :disabled="page <= 1">上一页</button>
    <span class="page-info">
      {{ page }} / {{ totalPages }}
      <span class="jump">
        跳转：<input @keyup.enter="goToPage" type="number" min="1" :max="totalPages" />
        <button @click="goToPage">Go</button>
      </span>
    </span>
    <button @click="next" :disabled="page >= totalPages">下一页</button>
    <select :value="pageSize" @change="changePageSize" class="page-size">
      <option value="10">10 条/页</option>
      <option value="20">20 条/页</option>
      <option value="50">50 条/页</option>
      <option value="100">100 条/页</option>
    </select>
  </div>
</template>

<style scoped>
.pager {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  margin-top: 20px;
  padding: 12px 0;
}

.pager button {
  font: inherit;
  padding: 6px 14px;
  border: 1px solid var(--border);
  border-radius: 6px;
  background: var(--bg);
  color: var(--text);
  cursor: pointer;
  transition: all 0.15s;
}

.pager button:hover:not(:disabled) {
  border-color: var(--primary);
  color: var(--primary);
}

.pager button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.page-info {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 13px;
  color: var(--text-muted);
}

.jump {
  display: flex;
  align-items: center;
  gap: 4px;
}

.jump input {
  width: 60px;
  padding: 4px 8px;
  border: 1px solid var(--border);
  border-radius: 4px;
  background: var(--bg);
  color: var(--text);
  font: inherit;
  font-size: 13px;
  text-align: center;
}

.jump input::-webkit-inner-spin-button,
.jump input::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.jump button {
  padding: 4px 10px;
  font-size: 12px;
}

.page-size {
  font: inherit;
  padding: 4px 8px;
  border: 1px solid var(--border);
  border-radius: 4px;
  background: var(--bg);
  color: var(--text);
  font-size: 12px;
}
</style>
