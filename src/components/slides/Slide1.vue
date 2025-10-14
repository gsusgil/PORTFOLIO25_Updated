<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'

/* === CONFIG === */
const BASE = import.meta.env.BASE_URL || '/'

type ProjectItem = {
  src: string
  alt: string
  caption: string // descripción completa; se recorta a 100 chars en el overlay
}

const projects = ref<ProjectItem[]>([
  {
    src: `${BASE}image/project1.jpg`,
    alt: 'Editorial series — Hat portrait',
    caption:
      'Serie editorial en blanco y negro con énfasis en sombras duras y dirección de arte minimalista.',
  },
  {
    src: `${BASE}image/project2.jpg`,
    alt: 'Beauty close-up — Studio lighting',
    caption:
      'Retrato beauty con iluminación de estudio y retoque fino orientado a campañas de skin-care.',
  },
])

/* === MODAL STATE === */
const isOpen = ref(false)
const activeIndex = ref<number | null>(null)
const activeItem = computed(() =>
  activeIndex.value == null ? null : projects.value[activeIndex.value],
)
const caption100 = computed(() => {
  const txt = activeItem.value?.caption ?? ''
  return txt.length > 100 ? txt.slice(0, 100).trimEnd() + '…' : txt
})

function openModal(i: number) {
  activeIndex.value = i
  isOpen.value = true
  lockScroll(true)
}
function closeModal() {
  isOpen.value = false
  activeIndex.value = null
  lockScroll(false)
}
function onKey(e: KeyboardEvent) {
  if (!isOpen.value) return
  if (e.key === 'Escape') closeModal()
  if (e.key === 'ArrowRight' && activeIndex.value != null) {
    activeIndex.value = (activeIndex.value + 1) % projects.value.length
  }
  if (e.key === 'ArrowLeft' && activeIndex.value != null) {
    activeIndex.value = (activeIndex.value - 1 + projects.value.length) % projects.value.length
  }
}

/* Evita scroll de fondo cuando el modal está abierto */
function lockScroll(on: boolean) {
  const el = document.documentElement
  if (on) el.classList.add('no-scroll')
  else el.classList.remove('no-scroll')
}

onMounted(() => {
  window.addEventListener('keydown', onKey)
})
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKey)
  lockScroll(false)
})
</script>

<template>
  <!-- Mantén .panel para que GSAP cuente este slide -->
  <section class="panel slide1">
    <div class="s1-grid">
      <!-- Columna izquierda: título + media (2 imgs + video) -->
      <div class="s1-left">
        <h2 class="s1-title">Projects</h2>

        <div class="s1-media">
          <!-- 2 imágenes con hover-zoom -->
          <figure v-for="(item, i) in projects" :key="item.src" class="s1-card">
            <button
              class="s1-thumb"
              type="button"
              :aria-label="`Abrir imagen: ${item.alt}`"
              @click="openModal(i)"
            >
              <img :src="item.src" :alt="item.alt" />
            </button>
            <figcaption class="sr-only">{{ item.caption }}</figcaption>
          </figure>

          <!-- Video 16:9 -->
          <div class="s1-video">
            <video
              class="s1-video-el"
              :src="`${BASE}video/demo.mp4`"
              controls
              playsinline
              preload="metadata"
              :poster="`${BASE}image/video-poster.jpg`"
            ></video>
          </div>
        </div>
      </div>

      <!-- Columna derecha: texto explicativo -->
      <div class="s1-right">
        <div class="s1-copy">
          <h3 class="s1-subtitle">Dirección de arte y postproducción</h3>
          <p>
            Selección de trabajos personales y comerciales. Foco en retrato, moda y beauty con
            exploración de contraste, composición y narrativa visual. El video resume el flujo:
            pre-producción → shooting → retoque.
          </p>
        </div>
      </div>
    </div>

    <!-- MODAL / POPUP -->
    <teleport to="body">
      <div
        v-if="isOpen && activeItem"
        class="s1-modal"
        role="dialog"
        aria-modal="true"
        aria-label="Vista ampliada de proyecto"
        @click.self="closeModal"
      >
        <div class="s1-modal-box">
          <button class="s1-close" type="button" aria-label="Cerrar" @click="closeModal">✕</button>

          <div class="s1-modal-media">
            <img :src="activeItem.src" :alt="activeItem.alt" />
            <!-- Overlay gradiente blanco 25% desde abajo + texto -->
            <div class="s1-gradient">
              <p class="s1-caption">{{ caption100 }}</p>
            </div>
          </div>

          <div class="s1-modal-nav" aria-hidden="true">
            <button
              class="s1-nav-btn"
              @click.stop="activeIndex = (activeIndex! - 1 + projects.length) % projects.length"
              aria-label="Anterior"
            >
              ⟵
            </button>
            <button
              class="s1-nav-btn"
              @click.stop="activeIndex = (activeIndex! + 1) % projects.length"
              aria-label="Siguiente"
            >
              ⟶
            </button>
          </div>
        </div>
      </div>
    </teleport>
  </section>
</template>

<style>
/* ===== base / layout compatible con tu Slide2 ===== */
.panel.slide1 {
  background: #fff;
  overflow: visible;
}

/* grid 2 columnas como en Slide2 */
.s1-grid {
  width: min(1200px, 92vw);
  margin: 0 auto;
  min-height: 100%;
  display: grid;
  grid-template-columns: 1fr 1fr;
  align-items: center;
  gap: clamp(20px, 4vw, 60px);
}

/* ===== izquierda ===== */
.s1-left {
  display: grid;
  align-content: start;
  gap: clamp(12px, 1.6vw, 18px);
}
.s1-title {
  margin: 0;
  font-size: clamp(28px, 5vw, 54px);
  line-height: 1;
  letter-spacing: 0.01em;
  color: #0f172a;
}
.s1-media {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: clamp(12px, 1.6vw, 18px);
  align-items: start;
}

/* tarjetas de imagen */
.s1-card {
  margin: 0;
}
.s1-thumb {
  display: block;
  position: relative;
  width: 100%;
  aspect-ratio: 4 / 5; /* similar a tus shots verticales */
  border: none;
  padding: 0;
  cursor: zoom-in;
  border-radius: 14px;
  overflow: hidden;
  background: #f3f4f6;
  box-shadow: 18px 24px 24px rgba(0, 0, 0, 0.08);
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease;
  will-change: transform;
}
.s1-thumb img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transform: scale(1);
  transition: transform 0.45s cubic-bezier(0.2, 0.7, 0.3, 1);
  backface-visibility: hidden;
}
.s1-thumb:hover img,
.s1-thumb:focus-visible img {
  transform: scale(1.07); /* hover-zoom */
}
.s1-thumb:hover,
.s1-thumb:focus-visible {
  box-shadow: 26px 36px 38px rgba(0, 0, 0, 0.18);
  transform: translateY(-2px);
  outline: none;
}

/* video 16:9 */
.s1-video {
  grid-column: span 2;
}
.s1-video-el {
  width: 100%;
  aspect-ratio: 16 / 9;
  display: block;
  background: #000;
  border-radius: 14px;
  overflow: hidden;
  box-shadow: 18px 24px 24px rgba(0, 0, 0, 0.08);
}

/* ===== derecha ===== */
.s1-right {
  display: grid;
  justify-items: start;
}
.s1-copy {
  max-width: 520px;
  color: #0f172a;
}
.s1-subtitle {
  margin: 0 0 8px 0;
  font-size: clamp(18px, 2.6vw, 24px);
  color: #111827;
}
.s1-copy p {
  margin: 0;
  font-size: clamp(14px, 1.6vw, 16px);
  line-height: 1.55;
  opacity: 0.9;
}

/* ===== modal ===== */
.s1-modal {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.6);
  display: grid;
  place-items: center;
  z-index: 1000;
}
.s1-modal-box {
  position: relative;
  width: min(92vw, 1080px);
}
.s1-modal-media {
  position: relative;
  width: 100%;
  aspect-ratio: 16 / 10; /* caja flexible para la mayoría de retratos */
  background: #fff;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 40px 90px rgba(0, 0, 0, 0.45);
}
.s1-modal-media img {
  width: 100%;
  height: 100%;
  object-fit: contain; /* no recorta tu imagen en el modal */
  background: #fff;
}

/* degradado blanco 25% desde abajo */
.s1-gradient {
  position: absolute;
  inset: auto 0 0 0; /* pegado abajo */
  height: 25%;
  background: linear-gradient(
    to top,
    rgba(255, 255, 255, 0.95) 0%,
    rgba(255, 255, 255, 0.75) 60%,
    rgba(255, 255, 255, 0) 100%
  );
  display: flex;
  align-items: end;
  padding: 12px 16px;
}
.s1-caption {
  margin: 0;
  font-size: clamp(13px, 1.4vw, 15px);
  color: #111827;
}

/* navegación modal */
.s1-modal-nav {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: space-between;
  pointer-events: none;
}
.s1-nav-btn {
  pointer-events: auto;
  border: none;
  background: rgba(255, 255, 255, 0.35);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  width: 44px;
  height: 44px;
  border-radius: 999px;
  cursor: pointer;
  margin: 0 8px;
}
.s1-close {
  position: absolute;
  top: 8px;
  right: 8px;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 999px;
  background: rgba(255, 255, 255, 0.8);
  cursor: pointer;
}

/* accesibilidad simple */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  margin: -1px;
  clip: rect(0 0 0 0);
  overflow: hidden;
  white-space: nowrap;
  border: 0;
}

/* responsive: apila en móvil */
@media (max-width: 860px) {
  .s1-grid {
    grid-template-columns: 1fr;
    gap: 28px;
  }
  .s1-media {
    grid-template-columns: 1fr 1fr;
  }
  .s1-right {
    justify-items: center;
    text-align: center;
  }
  .s1-copy {
    max-width: 92vw;
  }
}

/* bloquea scroll al abrir modal */
html.no-scroll,
html.no-scroll body {
  overflow: hidden;
}
</style>
