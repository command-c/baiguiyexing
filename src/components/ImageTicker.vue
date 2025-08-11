<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue'

// 读取项目根目录下的 gallery 中的所有常见图片类型
// 从 src/components 到项目根是 ../../
const modules = import.meta.glob('../../gallery/*.{png,jpg,jpeg,gif,webp,avif}', {
  eager: true,
  import: 'default',
})

const images = Object.values(modules)

// 拆成两排，并各自做无缝拼接
const row1 = computed(() => images.filter((_, i) => i % 2 === 0))
const row2 = computed(() => images.filter((_, i) => i % 2 === 1))
const doubled1 = computed(() => row1.value.concat(row1.value))
const doubled2 = computed(() => row2.value.concat(row2.value))
// 使用 rAF 控制偏移，便于拖拽
const offset1 = ref(0)
const offset2 = ref(0)
const speed1 = ref(30) // px/s 左移
const speed2 = ref(22)
const track1 = ref(null)
const track2 = ref(null)
const rowWidth1 = ref(1)
const rowWidth2 = ref(1)

function measure() {
  const t1 = track1.value
  const t2 = track2.value
  rowWidth1.value = Math.max(1, Math.round((t1?.offsetWidth || 0) / 2))
  rowWidth2.value = Math.max(1, Math.round((t2?.offsetWidth || 0) / 2))
}

let raf = 0
let lastTs = 0
const dragging = ref(false)
const dragStartX = ref(0)
const startOffset1 = ref(0)
const startOffset2 = ref(0)
const didDrag = ref(false)

function step(ts) {
  if (!lastTs) lastTs = ts
  const dt = (ts - lastTs) / 1000
  lastTs = ts
  if (!dragging.value) {
    offset1.value -= speed1.value * dt
    offset2.value -= speed2.value * dt
  }
  raf = requestAnimationFrame(step)
}

function onPointerDown(e) {
  dragging.value = true
  didDrag.value = false
  dragStartX.value = e.clientX ?? (e.touches?.[0]?.clientX || 0)
  startOffset1.value = offset1.value
  startOffset2.value = offset2.value
  window.addEventListener('pointermove', onPointerMove, { passive: false })
  window.addEventListener('pointerup', onPointerUp, { passive: true })
  window.addEventListener('touchmove', onPointerMove, { passive: false })
  window.addEventListener('touchend', onPointerUp, { passive: true })
}

function onPointerMove(e) {
  const x = e.clientX ?? (e.touches?.[0]?.clientX || 0)
  const dx = x - dragStartX.value
  if (Math.abs(dx) > 3) didDrag.value = true
  offset1.value = startOffset1.value + dx
  offset2.value = startOffset2.value + dx
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
    <!-- 第一排 -->
    <div class="ticker__mask">
      <div ref="track1" class="ticker__track" :style="translateStyle(offset1, rowWidth1)">
        <figure v-for="(src, idx) in doubled1" :key="`r1-` + idx" class="ticker__item">
          <img :src="src" alt="gallery image" loading="lazy" @click="!didDrag ? openLightbox(src) : null" />
        </figure>
      </div>
    </div>

    <!-- 第二排 -->
    <div class="ticker__mask">
      <div ref="track2" class="ticker__track" :style="translateStyle(offset2, rowWidth2)">
        <figure v-for="(src, idx) in doubled2" :key="`r2-` + idx" class="ticker__item">
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
  padding: 0.5rem 0;
  display: grid;
  gap: 0.75rem;
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
  height: 140px;
  width: auto;
  display: block;
  border-radius: 12px;
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.25);
  object-fit: cover;
  cursor: zoom-in;
}

/* JS 驱动滚动，不再使用 CSS 关键帧 */

@media (min-width: 768px) {
  .ticker__item img { height: 180px; }
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
