<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue'

// 读取项目根目录下的 gallery 中的所有常见图片类型
// 从 src/components 到项目根是 ../../
const modules = import.meta.glob('../../gallery/*.{png,jpg,jpeg,gif,webp,avif,PNG,JPG,JPEG,GIF}', {
  eager: true,
  import: 'default',
})

const images = Object.values(modules)

// 改为 5 排
const ROWS = 5
const rows = computed(() => Array.from({ length: ROWS }, (_, r) => images.filter((_, i) => i % ROWS === r)))
const doubledRows = computed(() => rows.value.map(list => list.concat(list)))

// 使用 rAF 控制偏移，便于拖拽
const offsets = ref(Array(ROWS).fill(0)) // 每行偏移
// 每行速度（px/s），奇数行向右（负值），偶数行向左（正值）
const speeds = computed(() => [30, 26, 24, 28, 22].map((s, i) => (i % 2 === 1 ? -s : s)))
const tracks = ref(Array(ROWS).fill(null)) // 轨道元素引用
const rowWidths = ref(Array(ROWS).fill(1)) // 单轮宽度（每个 track 宽度的一半）

function measure() {
  rowWidths.value = tracks.value.map(t => Math.max(1, Math.round((t?.offsetWidth || 0) / 2)))
}

let raf = 0
let lastTs = 0
const dragging = ref(false)
const dragStartX = ref(0)
const startOffsets = ref(Array(ROWS).fill(0))
const didDrag = ref(false)

function step(ts) {
  if (!lastTs) lastTs = ts
  const dt = (ts - lastTs) / 1000
  lastTs = ts
  if (!dragging.value) {
    offsets.value = offsets.value.map((val, i) => val - ((speeds.value?.[i] || 0) * dt))
  }
  raf = requestAnimationFrame(step)
}

function onPointerDown(e) {
  dragging.value = true
  didDrag.value = false
  dragStartX.value = e.clientX ?? (e.touches?.[0]?.clientX || 0)
  startOffsets.value = [...offsets.value]
  window.addEventListener('pointermove', onPointerMove, { passive: false })
  window.addEventListener('pointerup', onPointerUp, { passive: true })
  window.addEventListener('touchmove', onPointerMove, { passive: false })
  window.addEventListener('touchend', onPointerUp, { passive: true })
}

function onPointerMove(e) {
  const x = e.clientX ?? (e.touches?.[0]?.clientX || 0)
  const dx = x - dragStartX.value
  if (Math.abs(dx) > 3) didDrag.value = true
  // 反向应用 dx，使得向右拖动时内容也向右移动
  offsets.value = startOffsets.value.map(v => v - dx)
  e.preventDefault()
}

function onPointerUp() {
  dragging.value = false
  window.removeEventListener('pointermove', onPointerMove)
  window.removeEventListener('pointerup', onPointerUp)
  window.removeEventListener('touchmove', onPointerMove)
  window.removeEventListener('touchend', onPointerUp)
}

onMounted(() => {
  measure()
  setTimeout(measure, 400)
  window.addEventListener('resize', measure)
  raf = requestAnimationFrame(step)
})

onUnmounted(() => {
  cancelAnimationFrame(raf)
  window.removeEventListener('resize', measure)
  onPointerUp()
})

function translateStyle(offset, width) {
  const w = width || 1
  let m = offset % w
  if (m < 0) m += w
  const x = -m
  return { transform: `translateX(${x}px)` }
}

// 简易灯箱状态
const lightboxOpen = ref(false)
const lightboxSrc = ref('')

function openLightbox(src) {
  lightboxSrc.value = src
  lightboxOpen.value = true
  document.documentElement.style.overflow = 'hidden'
}

function closeLightbox() {
  lightboxOpen.value = false
  lightboxSrc.value = ''
  document.documentElement.style.overflow = ''
}
</script>

<template>
  <section
    class="ticker"
    :class="{ dragging }"
    aria-label="image ticker gallery"
    @pointerdown="onPointerDown"
    @touchstart.passive="onPointerDown"
  >
    <div class="ticker__mask" v-for="(list, r) in doubledRows" :key="'row-'+r">
      <div
        class="ticker__track"
        :ref="el => (tracks[r] = el)"
        :style="translateStyle(offsets[r], rowWidths[r])"
      >
        <figure v-for="(src, idx) in list" :key="'row-' + r + '-' + idx" class="ticker__item">
          <img :src="src" alt="gallery image" loading="lazy" @click="!didDrag ? openLightbox(src) : null" />
        </figure>
      </div>
    </div>
    
    <!-- 灯箱 -->
    <div v-if="lightboxOpen" class="lightbox" role="dialog" aria-modal="true" @click.self="closeLightbox">
      <button class="lightbox__close" aria-label="Close" @click="closeLightbox">×</button>
      <img class="lightbox__img" :src="lightboxSrc" alt="preview" />
    </div>
  </section>
</template>

<style scoped>
.ticker {
  width: 100%;
  overflow: hidden;
  padding: 0.5rem 0.25rem;
  display: grid;
  gap: 0.5rem;
  cursor: grab;
  user-select: none;
}

.ticker.dragging { cursor: grabbing; }

.ticker__mask {
  mask-image: linear-gradient(to right, transparent, black 10%, black 90%, transparent);
  -webkit-mask-image: linear-gradient(to right, transparent, black 10%, black 90%, transparent);
}

.ticker__track {
  display: flex;
  width: max-content;
  gap: 1rem;
}

.ticker__item {
  margin: 0;
}

.ticker__item img {
  height: 110px;
  width: auto;
  display: block;
  border-radius: 12px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.25);
  object-fit: cover;
  cursor: zoom-in;
}

/* JS 驱动滚动，不再使用 CSS 关键帧 */

@media (min-width: 768px) {
  .ticker__item img { height: 140px; }
}

/* 灯箱样式 */
.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.8);
  display: grid;
  place-items: center;
  z-index: 1000;
  padding: 2rem;
  animation: lbFade 240ms ease-out both;
}

.lightbox__img {
  width: 50vw;
  height: auto;
  border-radius: 14px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.6);
  animation: lbZoom 280ms ease-out both;
  cursor: zoom-out;
}

.lightbox__close {
  position: absolute;
  top: 12px;
  right: 16px;
  font-size: 2rem;
  line-height: 1;
  color: #fff;
  background: transparent;
  border: none;
  cursor: pointer;
}

@keyframes lbFade {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes lbZoom {
  from {
    opacity: 0;
    transform: scale(0.85);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
</style>
