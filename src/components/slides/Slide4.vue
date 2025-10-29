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

/* Tooltip + copiar */
const EMAIL = 'gsusgildesigner@gmailcom'
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
  toastTimer = window.setTimeout(() => (showToast.value = false), 1600)
}
</script>

<template>
  <section id="slide-4" class="panel s4" style="padding: 0">
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

        <!-- bloque relativo: let’s talk + tooltip + toast (sin empujar nada) -->
        <div class="talk-wrap">
          <span
            class="talk"
            role="button"
            tabindex="0"
            aria-describedby="talk-tip"
            @mouseenter="showTip = true"
            @mouseleave="showTip = false"
            @focus="showTip = true"
            @blur="showTip = false"
            @click="copyEmail"
            @keydown.enter.prevent="copyEmail"
          >
            let’s talk
            <span v-show="showTip" class="tooltip" id="talk-tip" aria-hidden="true">
              {{ EMAIL }}
            </span>
          </span>

          <!-- toast absoluto (no modifica layout) -->
          <div class="toast" v-show="showToast" aria-live="polite">Email copied — your move.</div>
        </div>
      </div>
    </div>
  </section>
</template>

<style>
/* Base del sitio */
.panel.s4 {
  background: #fff;
  color: #374151; /* gris plomo */
  font-family:
    -apple-system, BlinkMacSystemFont, 'Helvetica Neue', Helvetica, Arial, ui-sans-serif, system-ui,
    sans-serif;
}

/* Centro matemático */
.panel.s4 .s4-center {
  height: 100%;
  width: 100%;
  display: grid;
  place-items: center;
}

/* Columna con MUCHO más espacio entre bloques */
.panel.s4 .col {
  display: grid;
  justify-items: center;
  text-align: center;
  gap: 56px; /* ← más aire entre “gsusgil …” y “let’s talk” */
}

/* Línea principal */
.panel.s4 .brandline {
  display: inline-flex;
  align-items: baseline;
  gap: 0.6rem;
  line-height: 1.15;
  letter-spacing: 0.2px;
  user-select: none;
}
.panel.s4 .brand {
  font-weight: 400; /* regular */
  font-size: clamp(24px, 3.2vw, 32px);
  color: #374151;
}
.panel.s4 .longtail {
  min-width: 7ch;
  text-align: left;
  font-weight: 600; /* semibold */
  opacity: 0.95;
  font-size: clamp(18px, 2.6vw, 28px);
  color: #374151;
}

/* Fade entre palabras */
.panel.s4 .fade-enter-active,
.panel.s4 .fade-leave-active {
  transition: opacity 0.22s ease;
}
.panel.s4 .fade-enter-from,
.panel.s4 .fade-leave-to {
  opacity: 0;
}

/* Let’s talk */
.panel.s4 .talk-wrap {
  position: relative; /* ancla tooltip + toast sin mover layout */
  display: grid;
  justify-items: center;
}
.panel.s4 .talk {
  position: relative;
  display: inline-block;
  font-size: clamp(14px, 1.8vw, 16px);
  font-weight: 500;
  color: #374151;
  line-height: 1.2;
  user-select: none;
  cursor: pointer; /* pointer como botón */
}
.panel.s4 .talk:hover {
  opacity: 0.9;
}
.panel.s4 .talk:focus-visible {
  outline: 2px solid rgba(55, 65, 81, 0.35);
  outline-offset: 4px;
  border-radius: 4px;
}

/* Tooltip (debajo del “let’s talk”) */
.panel.s4 .tooltip {
  position: absolute;
  top: calc(100% + 10px);
  left: 50%;
  transform: translateX(-50%);
  white-space: nowrap;
  background: #111827;
  color: #fff;
  border-radius: 8px;
  padding: 6px 10px;
  font-size: 12px;
  font-weight: 600;
  box-shadow: 0 10px 22px rgba(0, 0, 0, 0.18);
  pointer-events: none;
  opacity: 0;
  transition:
    opacity 0.18s ease,
    transform 0.18s ease;
}
.panel.s4 .talk:hover .tooltip,
.panel.s4 .talk:focus .tooltip {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}
.panel.s4 .tooltip::after {
  content: '';
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  border-width: 6px;
  border-style: solid;
  border-color: transparent transparent #111827 transparent;
}

/* Toast glass — absoluto, UNA sola línea */
.panel.s4 .toast {
  position: absolute;
  top: calc(100% + 48px); /* debajo del tooltip; ajusta si lo quieres más lejos */
  left: 50%;
  transform: translateX(-50%);
  background: rgba(255, 255, 255, 0.78);
  -webkit-backdrop-filter: blur(8px);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(0, 0, 0, 0.08);
  color: #111827;
  padding: 8px 12px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 700;
  box-shadow: 0 18px 30px rgba(0, 0, 0, 0.12);
  pointer-events: none;
  display: inline-flex; /* ← no columnas raras */
  align-items: center;
  white-space: nowrap; /* ← UNA sola línea siempre */
  line-height: 1; /* compacto */
}
</style>
