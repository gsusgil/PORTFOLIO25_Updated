<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

/**
 * Props:
 * - duration: ms que permanece visible el loader (default 3000)
 * - fadeMs:   ms del desvanecido (default 500)
 * - message:  texto bajo los puntos (opcional)
 */
const props = withDefaults(defineProps<{
  duration?: number
  fadeMs?: number
  message?: string
}>(), {
  duration: 3000,
  fadeMs: 500,
  message: 'Cargando…'
})

const visible = ref(true)
let timer: number | null = null

onMounted(() => {
  // Oculta el overlay tras "duration"
  timer = window.setTimeout(() => { visible.value = false }, props.duration)
})

onBeforeUnmount(() => {
  if (timer) { clearTimeout(timer) }
})
</script>

<template>
  <!-- Teleport para asegurarnos de cubrir todo sin tocar tu layout -->
  <teleport to="body">
    <transition name="fade">
      <div
        v-if="visible"
        class="loader"
        role="status"
        aria-live="polite"
        aria-busy="true"
      >
        <div class="dots">
          <span class="dot"></span>
          <span class="dot"></span>
          <span class="dot"></span>
        </div>
        <p class="msg">{{ message }}</p>
      </div>
    </transition>
  </teleport>
</template>

<style scoped>
/* Overlay a pantalla completa */
.loader {
  position: fixed; inset: 0;
  display: grid; place-items: center;
  background: #0b0f14;  /* mismo fondo de tu sitio */
  color: #e5e7eb;
  z-index: 9999;
}

/* Puntos animados */
.dots { display: inline-flex; gap: 8px; align-items: center; }
.dot {
  width: 10px; height: 10px; border-radius: 50%;
  background: #38bdf8;
  animation: bounce .9s infinite ease-in-out;
}
.dot:nth-child(2) { animation-delay: .15s; }
.dot:nth-child(3) { animation-delay: .30s; }

.msg { margin-top: .75rem; text-align: center; color: #94a3b8; }

/* Fade in/out controlado por <transition> */
.fade-enter-active, .fade-leave-active {
  transition: opacity var(--fade, 500ms) ease;
}
.fade-enter-from, .fade-leave-to { opacity: 0; }

/* Keyframes */
@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}
</style>

<!-- CSS var para controlar el fade sin JS extra -->
<style>
/* Se define fuera de scoped para que la transición coja el valor */
.fade-enter-active, .fade-leave-active { --fade: 500ms; }
</style>
