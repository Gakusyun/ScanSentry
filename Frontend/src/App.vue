<script setup lang="ts">
import { ref } from 'vue'
import Pagination from './components/Pagination.vue'

const SERVERS = [
  { label: 'ss.gkux.cn', host: 'https://ss.gkux.cn' },
  { label: 'ss.238806.xyz', host: 'https://ss.238806.xyz' },
]

const serverIdx = ref(0)
const base = () => SERVERS[serverIdx.value].host

type Tab = 'overview' | 'logs' | 'ips' | 'paths'
const activeTab = ref<Tab>('overview')

interface Overview { total_requests: number; unique_ips: number }
interface LogItem { id: number; client_ip: string; method: string; path: string; timestamp: string }
interface LogsResponse { total: number; page: number; page_size: number; items: LogItem[] }
interface IPItem { client_ip: string; location: string; isp: string; access_count: number }
interface IPResponse { total: number; page: number; page_size: number; items: IPItem[] }
interface PathItem { path: string; access_count: number }
interface PathResponse { total: number; page: number; page_size: number; items: PathItem[] }

const overview = ref<Overview | null>(null)
const logs = ref<LogsResponse | null>(null)
const ips = ref<IPResponse | null>(null)
const paths = ref<PathResponse | null>(null)
const loading = ref(false)
const error = ref('')

const logsPage = ref(1)
const logsPageSize = ref(20)
const ipsPage = ref(1)
const ipsPageSize = ref(20)
const pathsPage = ref(1)
const pathsPageSize = ref(20)

async function get<T>(path: string): Promise<T> {
  const res = await fetch(base() + path)
  if (!res.ok) throw new Error(`HTTP ${res.status}`)
  return res.json() as T
}

async function load() {
  loading.value = true
  error.value = ''
  try {
    if (activeTab.value === 'overview') {
      overview.value = await get<Overview>('/show/me/overview')
    } else if (activeTab.value === 'logs') {
      logs.value = await get<LogsResponse>(`/show/me/logs?page=${logsPage.value}&page_size=${logsPageSize.value}`)
    } else if (activeTab.value === 'ips') {
      ips.value = await get<IPResponse>(`/show/me/ip?page=${ipsPage.value}&page_size=${ipsPageSize.value}`)
    } else {
      paths.value = await get<PathResponse>(`/show/me/path?page=${pathsPage.value}&page_size=${pathsPageSize.value}`)
    }
  } catch (e: unknown) {
    error.value = e instanceof Error ? e.message : String(e)
  } finally {
    loading.value = false
  }
}

function tab(t: Tab) {
  activeTab.value = t
  load()
}

function formatTime(utc: string): string {
  const d = new Date(utc)
  return d.toLocaleString('zh-CN', { hour12: false })
}

load()
</script>

<template>
  <div id="app">
    <header>
      <select v-model="serverIdx" @change="load">
        <option v-for="(s, i) in SERVERS" :key="i" :value="i">{{ s.label }}</option>
      </select>
      <nav>
        <button :class="{ active: activeTab === 'overview' }" @click="tab('overview')">Overview</button>
        <button :class="{ active: activeTab === 'logs' }" @click="tab('logs')">Logs</button>
        <button :class="{ active: activeTab === 'ips' }" @click="tab('ips')">IPs</button>
        <button :class="{ active: activeTab === 'paths' }" @click="tab('paths')">Paths</button>
      </nav>
      <button class="refresh" @click="load" :disabled="loading">刷新</button>
    </header>
    <main>
      <div v-if="loading" class="loading">加载中...</div>
      <div v-else-if="error" class="error">
        <span>{{ error }}</span>
        <button @click="load">重试</button>
      </div>
      <template v-else>
        <section v-if="activeTab === 'overview' && overview">
          <div class="stat-grid">
            <div class="stat"><span class="lbl">总请求数</span><span class="val">{{ overview.total_requests.toLocaleString() }}</span></div>
            <div class="stat"><span class="lbl">独立 IP 数</span><span class="val">{{ overview.unique_ips.toLocaleString() }}</span></div>
          </div>
        </section>
        <section v-else-if="activeTab === 'logs' && logs">
          <div class="table-header">
            <span class="meta">共 {{ logs.total.toLocaleString() }} 条记录</span>
          </div>
          <table>
            <thead><tr><th>IP</th><th>Method</th><th>Path</th><th>Time</th></tr></thead>
            <tbody>
              <tr v-for="item in logs.items" :key="item.id">
                <td class="mono">{{ item.client_ip }}</td>
                <td class="mono">{{ item.method }}</td>
                <td class="mono">{{ item.path }}</td>
                <td class="mono">{{ formatTime(item.timestamp) }}</td>
              </tr>
            </tbody>
          </table>
          <Pagination
            v-model:page="logsPage"
            v-model:page-size="logsPageSize"
            :total="logs.total"
            @update:page="load"
            @update:page-size="load"
          />
        </section>
        <section v-else-if="activeTab === 'ips' && ips">
          <div class="table-header">
            <span class="meta">共 {{ ips.total.toLocaleString() }} 个 IP</span>
          </div>
          <table>
            <thead><tr><th>IP</th><th>Location</th><th>ISP</th><th>Access Count</th></tr></thead>
            <tbody>
              <tr v-for="item in ips.items" :key="item.client_ip">
                <td class="mono">{{ item.client_ip }}</td>
                <td>{{ item.location || '-' }}</td>
                <td>{{ item.isp || '-' }}</td>
                <td class="mono">{{ item.access_count.toLocaleString() }}</td>
              </tr>
            </tbody>
          </table>
          <Pagination
            v-model:page="ipsPage"
            v-model:page-size="ipsPageSize"
            :total="ips.total"
            @update:page="load"
            @update:page-size="load"
          />
        </section>
        <section v-else-if="activeTab === 'paths' && paths">
          <div class="table-header">
            <span class="meta">共 {{ paths.total.toLocaleString() }} 条路径</span>
          </div>
          <table>
            <thead><tr><th>Path</th><th>Access Count</th></tr></thead>
            <tbody>
              <tr v-for="item in paths.items" :key="item.path">
                <td class="mono">{{ item.path }}</td>
                <td class="mono">{{ item.access_count.toLocaleString() }}</td>
              </tr>
            </tbody>
          </table>
          <Pagination
            v-model:page="pathsPage"
            v-model:page-size="pathsPageSize"
            :total="paths.total"
            @update:page="load"
            @update:page-size="load"
          />
        </section>
      </template>
    </main>
  </div>
</template>
