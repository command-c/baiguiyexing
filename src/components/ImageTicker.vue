<script setup>
import { computed, ref } from 'vue'

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
const duration1 = computed(() => `${Math.max(row1.value.length * 3, 20)}s`)
const duration2 = computed(() => `${Math.max(row2.value.length * 3, 20)}s`)

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
  <section class="ticker" aria-label="image ticker gallery">
    <!-- 第一排 -->
    <div class="ticker__mask">
      <div class="ticker__track" :style="{ animationDuration: duration1 }">
        <figure v-for="(src, idx) in doubled1" :key="`r1-` + idx" class="ticker__item">
          <img :src="src" alt="gallery image" loading="lazy" @click="openLightbox(src)" />
        </figure>
      </div>
    </div>

    <!-- 第二排 -->
    <div class="ticker__mask">
      <div class="ticker__track" :style="{ animationDuration: duration2 }">
        <figure v-for="(src, idx) in doubled2" :key="`r2-` + idx" class="ticker__item">
          <img :src="src" alt="gallery image" loading="lazy" @click="openLightbox(src)" />
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
}

.ticker__mask {
  mask-image: linear-gradient(to right, transparent, black 10%, black 90%, transparent);
  -webkit-mask-image: linear-gradient(to right, transparent, black 10%, black 90%, transparent);
}

.ticker__track {
  display: flex;
  width: max-content;
  gap: 1rem;
  animation-name: scrollX;
  animation-timing-function: linear;
  animation-iteration-count: infinite;
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

@keyframes scrollX {
  from { transform: translateX(0); }
  to   { transform: translateX(-50%); }
}

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
  max-width: 50vw;
  max-height: 50vh;
  width: auto;
  height: auto;
  border-radius: 14px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.6);
  animation: lbZoom 280ms ease-out both;
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
