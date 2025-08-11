<script setup>
import { onMounted, onUnmounted } from 'vue'
import ImageTicker from '@/components/ImageTicker.vue'

// 使用 import 方式让 Vite 处理位于项目根目录的背景图
const bgUrl = new URL('../background.jpg', import.meta.url).href

let prev = {}
onMounted(() => {
  const b = document.body.style
  prev = {
    backgroundImage: b.backgroundImage,
    backgroundSize: b.backgroundSize,
    backgroundPosition: b.backgroundPosition,
    backgroundRepeat: b.backgroundRepeat,
    backgroundAttachment: b.backgroundAttachment,
    backgroundColor: b.backgroundColor,
  }
  b.backgroundImage = `url(${bgUrl})`
  b.backgroundSize = 'cover'
  b.backgroundPosition = 'center'
  b.backgroundRepeat = 'no-repeat'
  b.backgroundAttachment = 'fixed'
  b.backgroundColor = '#000'
})

onUnmounted(() => {
  const b = document.body.style
  b.backgroundImage = prev.backgroundImage || ''
  b.backgroundSize = prev.backgroundSize || ''
  b.backgroundPosition = prev.backgroundPosition || ''
  b.backgroundRepeat = prev.backgroundRepeat || ''
  b.backgroundAttachment = prev.backgroundAttachment || ''
  b.backgroundColor = prev.backgroundColor || ''
})
</script>

<template>
  <div class="page">
  <header class="hero" role="banner">
      <div class="hero__overlay">
        <h1 class="hero__title">百鬼夜行™️</h1>
  <p class="hero__subtitle">Portfolio Exhibition</p>
      </div>
    </header>

    <main class="content">
      <ImageTicker />
    </main>
  </div>
</template>

<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.hero {
  position: relative;
  height: 42vh;
  min-height: 240px;
  background-size: cover;
  background-position: center;
}

.hero__overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to bottom, rgba(0,0,0,0.45), rgba(0,0,0,0.35), rgba(0,0,0,0.55));
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-end;
  padding: 2rem 1rem;
}

.hero__title {
  color: silver;
  font-size: 8rem;
  line-height: 1.1;
  letter-spacing: 0.08em;
  text-shadow: 0 6px 18px rgba(0,0,0,0.5);
  font-weight: 800;
  font-family: "Brush Script MT", "Snell Roundhand", "Zapfino", "Segoe Script", "Lucida Handwriting", cursive;
  animation: titleFadeUp 3s ease-out both;
  will-change: transform, opacity;
}

/* @media (min-width: 768px) {
  .hero__title { font-size: 4.5rem; }
} */

.hero__subtitle {
  margin-top: 1.35rem;
  color: rgba(255,255,255,0.92);
  font-size: 4rem;
  line-height: 1.2;
  letter-spacing: 0.06em;
  text-shadow: 0 4px 14px rgba(0,0,0,0.45);
  font-weight: 500;
  font-family: "Brush Script MT", "Snell Roundhand", "Zapfino", "Segoe Script", "Lucida Handwriting", cursive;
  animation: subtitleFadeUp 2.2s ease-out 0.2s both;
}

/* @media (min-width: 768px) {
  .hero__subtitle { font-size: 1.75rem; }
} */

.content {
  padding: 1rem;
}

@keyframes titleFadeUp {
  from {
    opacity: 0;
    transform: translateY(24px) scale(0.98);
  }
  60% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

@keyframes subtitleFadeUp {
  from {
    opacity: 0;
    transform: translateY(18px) scale(0.99);
  }
  60% {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}
</style>
