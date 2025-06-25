<template>
  <div
      class="imprint-window"
      :class="{ maximized }"
      :style="windowStyle"
      @mousedown.stop
  >
    <div
        class="window-bar"
        @mousedown="startDrag"
        @dblclick="toggleMaximize"
    >
      <span class="window-dot red" @click="$emit('close')"></span>
      <span class="window-dot yellow" @click="$emit('minimize')"></span>
      <span class="window-dot green" @click="toggleMaximize"></span>
      <span class="window-title">Imprint</span>
    </div>
    <div class="window-content">
      <h2>Imprint</h2>
      <p>
        <strong>According to § 5 TMG</strong><br>
        Tjorge Färber<br>
        Stettiner Straße 3<br>
        25563 Wrist<br>
        Germany
      </p>
      <p>
        <strong>Contact</strong><br>
        <a href="mailto:tjorgefaerber@outlook.com">tjorgefaerber@outlook.com</a>
      </p>
      <p>
        <strong>Responsible for content (§ 55 Abs. 2 RStV)</strong><br>
        Tjorge Färber<br>
        Stettiner Straße 3<br>
        25563 Wrist<br>
        Germany
      </p>
    </div>
  </div>
</template>



<script setup lang="ts">
import { ref, computed, watch, onBeforeUnmount } from 'vue'

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
        : { top: pos.value.y + 'px', left: pos.value.x + 'px', width: '340px', height: 'auto' }
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
  padding: 20px 18px 18px 18px;
  color: #f3f3f3;
  font-size: 15px;
}
</style>