<template>
  <div
      class="imprint-window"
      :class="{ maximized }"
      :style="windowStyle"
      @mousedown.stop
  >
    <div class="window-bar" @mousedown="startDrag" @dblclick="toggleMaximize">
      <span class="window-dot red" @click="$emit('close')"></span>
      <span class="window-dot yellow" @click="$emit('minimize')"></span>
      <span class="window-dot green" @click="toggleMaximize"></span>
      <span class="window-title">My Projects</span>
    </div>
    <div class="window-content finder-view">
      <aside class="sidebar">
        <ul>
          <li
              v-for="repo in repos"
              :key="repo.name"
              :class="{ active: current === repo.name }"
              @click="select(repo.name)"
          >
            <img src="/img/folders/blue.png" class="folder-icon" alt="folder" />
            <div class="repo-info" @click.stop="openRepo(repo.html_url)">
              <strong>{{ repo.name }}</strong>
              <small>{{ formatDate(repo.created_at) }}</small>
              <p>{{ repo.description }}</p>
            </div>
          </li>
        </ul>
      </aside>
      <section class="details" v-if="currentRepo">
        <img :src="currentRepo.image" alt="Project preview" />
      </section>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, watch, onBeforeUnmount } from 'vue'
import axios from 'axios'

const props = defineProps<{
  maximized?: boolean
  position?: { x: number, y: number }
}>()
const emit = defineEmits(['close', 'minimize', 'maximize', 'update:position'])

const maximized = ref(props.maximized ?? false)
const pos = ref(props.position ?? { x: 0, y: 80 })
const dragging = ref(false)
const dragOffset = ref({ x: 0, y: 0 })

const windowStyle = computed(() =>
    maximized.value
        ? { top: '0', left: '0', width: '100vw', height: '100vh' }
        : { top: pos.value.y + 'px', left: pos.value.x + 'px', width: '960px', height: '480px' }
)

function startDrag(e: MouseEvent) {
  if (maximized.value) return
  dragging.value = true
  dragOffset.value = {
    x: e.clientX - pos.value.x,
    y: e.clientY - pos.value.y
  }
  document.addEventListener('mousemove', onDrag)
  document.addEventListener('mouseup', stopDrag)
}

function onDrag(e: MouseEvent) {
  if (!dragging.value) return
  pos.value = {
    x: Math.max(0, Math.min(window.innerWidth - 340, e.clientX - dragOffset.value.x)),
    y: Math.max(0, Math.min(window.innerHeight - 60, e.clientY - dragOffset.value.y))
  }
  emit('update:position', pos.value)
}

function stopDrag() {
  dragging.value = false
  document.removeEventListener('mousemove', onDrag)
  document.removeEventListener('mouseup', stopDrag)
}

function toggleMaximize() {
  maximized.value = !maximized.value
  emit('maximize', maximized.value)
}

watch(() => props.maximized, v => maximized.value = v ?? false)
watch(() => props.position, v => { if (v) pos.value = v })

onBeforeUnmount(() => stopDrag())

interface Repo {
  name: string
  description: string | null
  image: string
  html_url: string
  created_at: string
}

const repos = ref<Repo[]>([])
const current = ref<string | null>(null)
const currentRepo = ref<Repo | null>(null)

function select(name: string) {
  current.value = name
  currentRepo.value = repos.value.find(r => r.name === name) || null
}

function openRepo(url: string) {
  window.open(url, '_blank')
}

function formatDate(dateStr: string) {
  return new Date(dateStr).toLocaleDateString()
}

async function fetchRepos() {
  const res = await axios.get('https://api.github.com/users/BlackDevCoding/repos')
  repos.value = res.data
      .filter((r: { fork: boolean }) => !r.fork)
      .map((r: any): Repo => ({
        name: r.name,
        description: r.description,
        image: `https://raw.githubusercontent.com/BlackDevCoding/${r.name}/main/preview.png`,
        html_url: r.html_url,
        created_at: r.created_at
      }))
  if (repos.value.length) select(repos.value[0].name)
}

onMounted(fetchRepos)
</script>

<style scoped>
.imprint-window {
  position: fixed;
  background: #23272a;
  border-radius: 12px;
  box-shadow: 0 8px 32px #000a;
  z-index: 200;
  border: 1px solid #444;
  overflow: hidden;
  animation: popin 0.18s cubic-bezier(.4,2.2,.2,1);
  min-width: 240px;
  min-height: 120px;
  transition: all 0.18s cubic-bezier(.4,2.2,.2,1);
}
.imprint-window.maximized {
  border-radius: 0;
  width: 100vw !important;
  height: 100vh !important;
  left: 0 !important;
  top: 0 !important;
}
@keyframes popin {
  from { transform: scale(0.92); opacity: 0; }
  to { transform: scale(1); opacity: 1; }
}
.window-bar {
  display: flex;
  align-items: center;
  height: 32px;
  background: #181a1b;
  border-bottom: 1px solid #333;
  padding: 0 12px;
  cursor: grab;
  user-select: none;
}
.window-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 6px;
  display: inline-block;
}
.red { background: #ff5f56; cursor: pointer; }
.yellow { background: #ffbd2e; cursor: pointer; }
.green { background: #27c93f; cursor: pointer; }
.window-title {
  margin-left: 12px;
  color: #eee;
  font-size: 15px;
  font-weight: 500;
  flex: 1;
}
.window-content {
  height: calc(100% - 32px);
  display: flex;
  color: #f3f3f3;
  font-size: 14px;
}
.finder-view {
  display: flex;
  height: 100%;
}
.sidebar {
  width: 70%;
  background: #1e2124;
  border-right: 1px solid #444;
  overflow-y: auto;
}
.sidebar ul {
  list-style: none;
  padding: 0;
  margin: 0;
}
.sidebar li {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  padding: 10px;
  cursor: pointer;
  border-bottom: 1px solid #333;
}
.sidebar li.active {
  background: #404552;
}
.folder-icon {
  width: 32px;
  height: 32px;
}
.repo-info {
  flex: 1;
  cursor: pointer;
}
.repo-info strong {
  color: #fff;
  font-size: 14px;
}
.repo-info small {
  color: #aaa;
  font-size: 12px;
  display: block;
}
.repo-info p {
  color: #ccc;
  font-size: 13px;
  margin: 4px 0 0;
}
.details {
  width: 30%;
  padding: 16px;
  overflow-y: auto;
  background: #2a2d30;
  display: flex;
  align-items: center;
  justify-content: center;
}
.details img {
  max-width: 100%;
  max-height: 100%;
  border-radius: 8px;
  background: #333;
}
</style>
