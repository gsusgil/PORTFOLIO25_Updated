<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)

/* Tus slides */
import Slide1 from './slides/Slide1.vue'
import Slide2 from './slides/Slide2.vue'
import Slide3 from './slides/Slide3.vue'
import Slide4 from './slides/Slide4.vue'

const section = ref<HTMLElement | null>(null)
const track = ref<HTMLElement | null>(null)
const spacer = ref<HTMLDivElement | null>(null)

let tween: gsap.core.Tween | null = null
let st: ScrollTrigger | null = null

function setVW() {
  // define --vw para layout estable
  document.documentElement.style.setProperty('--vw', `${window.innerWidth}px`)
}

/** px a scrollear = (panels-1) * sectionWidth */
function scrollLenPx(sec: HTMLElement, trk: HTMLElement) {
  const total = trk.querySelectorAll<HTMLElement>('.panel').length
  return Math.max(0, (total - 1) * sec.offsetWidth)
}

function kill() {
  st?.kill()
  tween?.kill()
  st = null
  tween = null
}

function init() {
  const sec = section.value!
  const trk = track.value!
  const spc = spacer.value!

  setVW()

  // 1) Calcula y fija el alto del spacer ANTES del tween (evita CLS)
  const len = scrollLenPx(sec, trk)
  spc.style.height = `${len}px`

  // fuerza un layout sync para que el alto quede comprometido antes del pin
  void sec.offsetWidth

  gsap.set(trk, { x: 0, force3D: true })

  // 2) Crea el tween con pinSpacing DESACTIVADO (ya reservamos el espacio)
  tween = gsap.to(trk, {
    x: () => `-${scrollLenPx(sec, trk)}px`,
    ease: 'none',
    force3D: true,
    invalidateOnRefresh: true,
    immediateRender: false,
    scrollTrigger: {
      trigger: sec,
      start: 'top top',
      end: () => `+=${scrollLenPx(sec, trk)}`,
      pin: true,
      pinSpacing: false, // 👈 clave: SIN spacer de GSAP
      scrub: 0.4,
      anticipatePin: 1.2,
      fastScrollEnd: true,
      // pinType: 'fixed',     // si te da salto raro en Safari, comenta/ajusta
      // markers: true,
    },
  })
  st = tween.scrollTrigger!

  // Triggers internos que ya tienes pueden vivir en cada Slide
}

function refresh() {
  const sec = section.value!
  const trk = track.value!
  const spc = spacer.value!
  setVW()
  spc.style.height = `${scrollLenPx(sec, trk)}px`
  ScrollTrigger.refresh()
}

onMounted(() => {
  // init en el siguiente frame para dejar que el DOM se pinte con el spacer
  requestAnimationFrame(init)
  const onResize = () => refresh()
  window.addEventListener('resize', onResize, { passive: true })
  onBeforeUnmount(() => window.removeEventListener('resize', onResize))
})

onBeforeUnmount(() => {
  kill()
})
</script>

<template>
  <!-- Sección horizontal fija -->
  <section class="h-section" ref="section" aria-label="Proyectos (horizontal)">
    <div class="h-track" ref="track">
      <Slide1 />
      <Slide2 />
      <Slide3 />
      <Slide4 />
    </div>
  </section>

  <!-- Spacer estático — reserva el scroll horizontal total -->
  <div class="h-spacer" ref="spacer" aria-hidden="true"></div>
</template>

<!-- Sin scoped: deja que los slides apliquen su propio estilo -->
<style>
.h-section {
  height: 100svh;
  height: 100lvh;
  height: 100vh;
  position: relative;
  overflow: hidden;
  z-index: 1;
  margin: 0;
  padding: 0;
  width: var(--vw, 100vw);
  contain: layout paint; /* reduce side-effects del pin */
}
.h-track {
  height: 100%;
  display: flex;
  will-change: transform;
  transform: translateZ(0);
}

/* Cada panel ocupa la anchura del viewport (estable gracias a --vw) */
.panel {
  flex: 0 0 var(--vw, 100vw);
  height: 100%;
  display: grid;
  place-items: center;
  padding: 2rem;
}

/* Spacer ocupa exactamente el scroll horizontal, así no hay "scroll extra" al final */
.h-spacer {
  height: 0;
} /* se setea por JS con px exactos */

/* —— Limpieza: NO definimos estilos .card globales aquí —— */
/* Si tienes esto en tu versión antigua, elimínalo:
.card { padding:24px; background:rgba(0,0,0,.22); backdrop-filter:blur(2px); }
*/
</style>
