<template>
  <div class="desktop">
    <div class="menu-bar">
      <span class="apple-logo"></span>
      <span class="menu-item">Finder</span>
      <span class="menu-item">File</span>
      <span class="menu-item">Edit</span>
      <span class="menu-item">View</span>
      <span class="menu-item">Go</span>
      <span class="menu-item">Window</span>
      <span class="menu-item">Help</span>
      <span class="menu-item" @click="showImprintWindow">Imprint</span>
      <span class="clock">{{ date }} {{ time }}</span>
    </div>
    <div class="desktop-icons">
      <div class="icon">
        <img src="/img/folders/blue.png" alt="Blue Folder" />
        <span>Documents</span>
      </div>
      <div class="icon" @click="openGithubFinder">
        <img src="/img/folders/green.png" alt="Green Folder" />
        <span>Projects</span>
      </div>
    </div>
    <div class="dock">
      <div
          v-for="(icon, i) in dockIcons"
          :key="icon.alt + i"
          class="dock-icon-wrapper"
          @mouseenter="hoveredIcon = i"
          @mouseleave="hoveredIcon = null"
          style="position: relative;"
      >
        <div v-if="hoveredIcon === i" class="dock-tooltip">
          {{ icon.alt }}
        </div>
        <img
            class="dock-icon"
            :src="icon.src"
            :alt="icon.alt"
            @click="icon.alt === 'Imprint (Minimized)' ? handleImprintRestore() : openLink(icon.url, i)"
            style="user-select: none"
        />
      </div>
    </div>
    <ImprintWindow
        v-if="showImprint"
        :maximized="imprintMaximized"
        :position="imprintPosition"
        @close="handleImprintClose"
        @minimize="handleImprintMinimize"
        @maximize="handleImprintMaximize"
        @update:position="val => imprintPosition = val"
    />
    <GithubFinderWindow
        v-if="showGithubFinder"
        :maximized="githubFinderMaximized"
        :position="githubFinderPosition"
        @close="closeGithubFinder"
        @maximize="maximizeGithubFinder"
        @update:position="updateGithubFinderPosition"
    />

  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import ImprintWindow from './components/ImprintWindow.vue'
import GithubFinderWindow from './components/ProjectWindow.vue'

const time = ref('')
const date = ref('')
const hoveredIcon = ref<number|null>(null)
const showImprint = ref(false)
const imprintMinimized = ref(false)
const imprintMaximized = ref(false)
const imprintPosition = ref({ x: 0, y: 80 }) // safe default for SSR

function updateDateTime() {
  const now = new Date()
  time.value = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', hour12: false })
  date.value = now.toLocaleDateString([], { weekday: 'short', month: 'short', day: 'numeric' })
}

onMounted(() => {
  updateDateTime()
  setInterval(updateDateTime, 1000)
  // Only access window here
  imprintPosition.value = { x: window.innerWidth - 420, y: 80 }
})

function showImprintWindow() {
  showImprint.value = true
  imprintMinimized.value = false
}

const showGithubFinder = ref(false)
const githubFinderMaximized = ref(false)

const githubFinderPosition = ref({ x: 100, y: 100 })

onMounted(() => {
  githubFinderPosition.value = {
    x: window.innerWidth / 2 - 320,
    y: 100
  }
})


function openGithubFinder() {
  showGithubFinder.value = true
  githubFinderMaximized.value = false
}
function closeGithubFinder() {
  showGithubFinder.value = false
  githubFinderMaximized.value = false
}
function maximizeGithubFinder(val: boolean) {
  githubFinderMaximized.value = val
}
function updateGithubFinderPosition(pos: { x: number, y: number }) {
  githubFinderPosition.value = pos
}


const icons = [
  { src: '/img/apps/finder.png', alt: 'Finder', url: 'https://www.apple.com/macos/' },
  { src: '/img/apps/webstorm.png', alt: 'WebStorm', url: 'https://www.jetbrains.com/webstorm/' },
  { src: '/img/apps/pycharm.png', alt: 'PyCharm', url: 'https://www.jetbrains.com/pycharm/' },
  { src: '/img/apps/idea.png', alt: 'IntelliJ IDEA', url: 'https://www.jetbrains.com/idea/' },
  { src: '/img/apps/sublimetext.png', alt: 'Sublime Text', url: 'https://www.sublimetext.com/' },
  { src: '/img/apps/github.png', alt: 'GitHub', url: 'https://github.com/' },
  { src: '/img/apps/discord.png', alt: 'Discord', url: 'https://discord.com/' },
  { src: '/img/apps/email.png', alt: 'Email', url: 'mailto:' },
  { src: '/img/apps/littlesnitch.png', alt: 'Little Snitch', url: 'https://www.obdev.at/products/littlesnitch/index.html' },
  { src: '/img/apps/mongodb.png', alt: 'MongoDB', url: 'https://www.mongodb.com/' },
  { src: '/img/apps/termius.png', alt: 'Termius', url: 'https://termius.com/' },
  { src: '/img/apps/settings.png', alt: 'Imprint', url: '#' }
]

function openLink(url: string, i: number) {
  if (icons[i].alt === 'Imprint') {
    showImprint.value = true
    imprintMinimized.value = false
  } else {
    window.open(url, '_blank')
  }
}

const dockIcons = computed(() => {
  const base = [...icons]
  if (imprintMinimized.value) {
    base.push({ src: '/img/apps/settings.png', alt: 'Imprint (Minimized)', url: '#' })
  }
  return base
})

function handleImprintMinimize() {
  imprintMinimized.value = true
  showImprint.value = false
}
function handleImprintMaximize(val: boolean) {
  imprintMaximized.value = val
}
function handleImprintClose() {
  showImprint.value = false
  imprintMinimized.value = false
  imprintMaximized.value = false
}
function handleImprintRestore() {
  showImprint.value = true
  imprintMinimized.value = false
}
</script>

<style>
html, body {
  height: 100%;
  margin: 0;
  padding: 0;
  background: #181a1b;
}
</style>

<style scoped>
.desktop,
.menu-bar {
  font-family: 'Helvetica Neue', Helvetica, Arial, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}
.desktop {
  height: 100vh;
  width: 100vw;
  background: url('/img/wallpaper.jpg') center center/cover no-repeat, #181a1b;
  position: relative;
  overflow: hidden;
}
.menu-bar {
  height: 28px;
  background: rgba(32,32,32,0.55);
  backdrop-filter: blur(16px) saturate(160%);
  -webkit-backdrop-filter: blur(16px) saturate(160%);
  border-bottom: 1px solid rgba(255,255,255,0.08);
  display: flex;
  align-items: center;
  padding: 0 16px;
  font-size: 15px;
  color: #f3f3f3;
  box-shadow: 0 1px 4px rgba(0,0,0,0.25);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 10;
}
.apple-logo {
  font-size: 18px;
  margin-right: 12px;
  color: #f3f3f3;
}
.menu-item {
  margin-right: 16px;
  cursor: pointer;
  user-select: none;
  color: #f3f3f3;
}
.clock {
  margin-left: auto;
  font-variant-numeric: tabular-nums;
  color: #f3f3f3;
}
.desktop-icons {
  position: absolute;
  top: 60px;
  left: 40px;
  display: flex;
  flex-direction: column;
  gap: 32px;
}
.icon {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: #f3f3f3;
  text-shadow: 0 1px 2px #000a;
  font-size: 13px;
  cursor: pointer;
}
.icon img {
  width: 48px;
  margin-bottom: 4px;
  filter: drop-shadow(0 2px 4px #000c);
  max-width: 64px;
  max-height: 64px;
  object-fit: contain;
}
.dock {
  position: fixed;
  left: 50%;
  bottom: 18px;
  transform: translateX(-50%);
  background: rgba(24, 24, 24, 0.55);
  backdrop-filter: blur(16px) saturate(160%);
  -webkit-backdrop-filter: blur(16px) saturate(160%);
  border: 1px solid rgba(255,255,255,0.10);
  border-radius: 18px;
  padding: 8px 18px;
  display: flex;
  gap: 24px;
  box-shadow: 0 4px 24px #0008;
  z-index: 20;
}
.dock-icon-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
}
.dock-tooltip {
  position: absolute;
  bottom: 54px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(32,32,32,0.92);
  color: #fff;
  padding: 4px 12px;
  border-radius: 6px;
  font-size: 13px;
  white-space: nowrap;
  pointer-events: none;
  box-shadow: 0 2px 8px #0006;
  z-index: 100;
  opacity: 0.98;
  transition: opacity 0.12s;
}
.dock-icon {
  width: 44px;
  height: 44px;
  border-radius: 10px;
  transition: transform 0.15s;
  cursor: pointer;
  background: #23272a;
}
.dock-icon:hover {
  transform: scale(1.18);
  background: rgba(255,255,255,0.08);
}
</style>