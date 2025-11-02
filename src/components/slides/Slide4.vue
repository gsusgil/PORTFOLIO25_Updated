<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

/* Rotación del longtail */
const words = ['designer', 'dev', 'graphic', 'visual', 'web', 'versatil']
const idx = ref(0)
const current = computed(() => words[idx.value])

let rotTimer: number | undefined
onMounted(() => {
  rotTimer = window.setInterval(() => (idx.value = (idx.value + 1) % words.length), 1000)
})
onBeforeUnmount(() => rotTimer && clearInterval(rotTimer))

/* Tooltip + copiar (unificado con HomeView) */
const EMAIL = 'gsusgildesigner@gmail.com'
const showTip = ref(false)
const showToast = ref(false)
let toastTimer: number | undefined

async function copyEmail() {
  try {
    await navigator.clipboard.writeText(EMAIL)
  } catch {
    const ta = document.createElement('textarea')
    ta.value = EMAIL
    ta.style.position = 'fixed'
    ta.style.opacity = '0'
    document.body.appendChild(ta)
    ta.select()
    try {
      document.execCommand('copy')
    } catch {}
    document.body.removeChild(ta)
  }
  showToast.value = true
  clearTimeout(toastTimer)
  toastTimer = window.setTimeout(() => (showToast.value = false), 1200)
}
</script>

<template>
  <section id="slide-4" class="panel s4">
    <div class="s4-center">
      <div class="col">
        <!-- línea principal -->
        <span class="brandline" aria-live="polite">
          <span class="brand">gsusgil</span>
          <span class="longtail">
            <transition name="fade" mode="out-in">
              <span class="word" :key="current">{{ current }}</span>
            </transition>
          </span>
        </span>

        <!-- bloque relativo: let’s talk + tooltip + toast -->
        <div class="talk-wrap">
          <button
            type="button"
            class="talk link-like"
            aria-describedby="talk-tip"
            @mouseenter="showTip = true"
            @mouseleave="showTip = false"
            @focus="showTip = true"
            @blur="showTip = false"
            @click="copyEmail"
          >
            let’s talk
            <span v-show="showTip" class="tooltip" id="talk-tip" aria-hidden="true">
              {{ EMAIL }}
            </span>
          </button>

          <div v-show="showToast" class="toast" aria-live="polite">Copied!</div>
        </div>
      </div>
    </div>
  </section>
</template>

<style scoped>
/* ===== Base ===== */
.panel.s4 {
  background: #fff;
  color: #0f172a;
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    sans-serif;
  height: 100%;
  width: 100%;
}
.s4-center {
  height: 100%;
  width: 100%;
  display: grid;
  place-items: center;
}
.col {
  display: grid;
  justify-items: center;
  text-align: center;
  gap: 3.3rem;
}

/* ===== Línea principal ===== */
.brandline {
  display: inline-flex;
  align-items: baseline;
  gap: 0.6rem;
  line-height: 1.1;
  letter-spacing: 0.2px;
  user-select: none;
}
.brand {
  font-weight: 700;
  font-size: clamp(26px, 3.2vw, 34px);
  color: #374151;
  text-transform: lowercase;
}
.longtail {
  min-width: 7ch;
  text-align: left;
  font-weight: 600;
  font-size: clamp(18px, 2.6vw, 26px);
  color: #ff4d4d; /* las palabras rotativas heredan el rojo vibrante */
  opacity: 1;
}

/* animación entre palabras */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.22s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

/* ===== Botón tipo link ===== */
.link-like {
  background: transparent;
  border: 0;
  padding: 0;
  cursor: pointer;
  font-weight: 600;
  letter-spacing: 0.15px;
  color: #0f172a;
}
.link-like:hover {
  opacity: 0.85;
}

/* ===== Tooltip ===== */
.talk-wrap {
  position: relative;
  display: grid;
  justify-items: center;
}
.tooltip {
  position: absolute;
  top: calc(100% + 10px);
  left: 50%;
  transform: translateX(-50%) translateY(4px);
  background: #0f172a;
  color: #fff;
  font-size: 0.72rem;
  padding: 0.45rem 0.7rem;
  border-radius: 6px;
  line-height: 1.35;
  white-space: nowrap;
  pointer-events: none;
  opacity: 0;
  transition:
    opacity 0.18s ease,
    transform 0.18s ease;
  z-index: 2;
  box-shadow: 0 10px 22px rgba(15, 23, 42, 0.15);
}
.talk-wrap .talk:hover .tooltip,
.talk-wrap .talk:focus-visible + .tooltip,
.tooltip[aria-hidden='false'] {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}
.tooltip::after {
  content: '';
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  border-width: 6px;
  border-style: solid;
  border-color: transparent transparent #0f172a transparent;
}

/* ===== Toast (mismo estilo HomeView: píldora blanca, sombra suave) ===== */
.toast {
  position: absolute;
  top: calc(100% + 50px);
  left: 50%;
  transform: translateX(-50%);
  background: #ffffff;
  color: #000;
  border: 1px solid rgba(0, 0, 0, 0.08);
  border-radius: 999px;
  padding: 0.28rem 0.85rem 0.38rem;
  font-size: 0.68rem;
  font-weight: 500;
  box-shadow: 0 12px 26px rgba(15, 23, 42, 0.08);
  pointer-events: none;
  white-space: nowrap;
  z-index: 3;
}

/* focus accesible */
.talk:focus-visible {
  outline: 2px solid rgba(225, 6, 0, 0.35);
  outline-offset: 4px;
  border-radius: 6px;
}
</style>
