<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollTrigger)

import Slide1 from './slides/Slide1.vue'
import Slide2 from './slides/Slide2.vue'
import Slide3 from './slides/Slide3.vue'
import Slide4 from './slides/Slide4.vue'

const section = ref<HTMLElement | null>(null)
const track = ref<HTMLElement | null>(null)

let tween: gsap.core.Tween | null = null
let st: ScrollTrigger | null = null

const refresh = () => ScrollTrigger.refresh()

onMounted(() => {
  const sec = section.value!
  const trk = track.value!

  // Si usas --vw en CSS, define la var
  const setVW = () => document.documentElement.style.setProperty('--vw', `${window.innerWidth}px`)
  setVW()

  gsap.set(trk, { x: 0, force3D: true })

  const panels = Array.from(trk.querySelectorAll<HTMLElement>('.panel'))
  const total = panels.length

  const scrollLen = () => Math.max(0, (total - 1) * sec.offsetWidth)

  // SNAP (sin tipos raros). Si no hay snap, usa undefined (no false).
  const snapCfg =
    total > 1
      ? ({
          snapTo: (v: number) => {
            const step = 1 / (total - 1)
            const SAFE = 0.08
            return v < SAFE || v > 1 - SAFE ? v : gsap.utils.snap(step)(v)
          },
          duration: { min: 0.08, max: 0.16 },
          delay: 0,
          ease: 'power1.out',
          inertia: false,
          directional: true,
        } as any)
      : undefined

  tween = gsap.to(trk, {
    x: () => `-${scrollLen()}px`,
    ease: 'none',
    force3D: true,
    invalidateOnRefresh: true,
    immediateRender: false,
    scrollTrigger: {
      trigger: sec,
      start: 'top top',
      end: () => `+=${scrollLen()}`,
      pin: true,
      pinSpacing: true,
      pinReparent: true, // si al volver a Home ves salto, ponlo en false
      scrub: 1,
      anticipatePin: 1,
      fastScrollEnd: true,
      snap: snapCfg,
      // markers: true,
    },
  })
  st = tween.scrollTrigger!

  // ---- Animaciones por slide ----
  const innerTriggers: ScrollTrigger[] = []
  panels.forEach((panel) => {
    const tl = gsap.timeline({
      defaults: { ease: 'power2.out' },
      scrollTrigger: {
        trigger: panel,
        containerAnimation: tween!,
        start: 'left center',
        end: 'right center',
        toggleActions: 'play none none reverse',
      },
    })

    tl.from(panel.querySelector('.title'), { y: 40, autoAlpha: 0, duration: 0.6 })
      .from(panel.querySelector('.subtitle'), { y: 20, autoAlpha: 0, duration: 0.45 }, '-=0.25')
      .from(
        panel.querySelectorAll(
          '.card, .features li, .media-placeholder, .button, .slot-media, .slot-text',
        ),
        { y: 20, autoAlpha: 0, duration: 0.4, stagger: 0.08 },
        '-=0.2',
      )

    if (tl.scrollTrigger) innerTriggers.push(tl.scrollTrigger)
  })

  gsap.from('#slide-3 .card', {
    y: 30,
    autoAlpha: 0,
    duration: 0.35,
    stagger: 0.06,
    scrollTrigger: {
      trigger: '#slide-3',
      containerAnimation: tween!,
      start: 'left center',
      toggleActions: 'play none none reverse',
    },
  })
  gsap.from('#slide-4 .button', {
    y: 18,
    scale: 0.96,
    autoAlpha: 0,
    duration: 0.4,
    scrollTrigger: {
      trigger: '#slide-4',
      containerAnimation: tween!,
      start: 'left center',
      toggleActions: 'play none none reverse',
    },
  })

  requestAnimationFrame(() => ScrollTrigger.refresh())

  const onResize = () => {
    setVW()
    refresh()
  }
  window.addEventListener('resize', onResize, { passive: true })

  onBeforeUnmount(() => {
    innerTriggers.forEach((t) => t.kill())
    window.removeEventListener('resize', onResize)
  })
})

onBeforeUnmount(() => {
  st?.kill()
  tween?.kill()
})
</script>

<template>
  <section class="h-section" ref="section" aria-label="Proyectos (horizontal)">
    <div class="h-track" ref="track">
      <!-- 👇 Ahora cada slide es un componente, pero sigue siendo .panel dentro de .h-track -->
      <Slide1 />
      <Slide2 />
      <Slide3 />
      <Slide4 />
    </div>
  </section>
</template>

<!-- 👇 Quita "scoped" para que el estilo afecte a los hijos .panel -->
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
}

.h-track {
  height: 100%;
  display: flex;
  will-change: transform;
  transform: translateZ(0);
}

.panel {
  flex: 0 0 var(--vw, 100vw);
  height: 100%;
  display: grid;
  place-items: center;
  color: #fff;
  padding: 2rem;
}

.inner {
  width: min(1100px, 92vw);
  margin: 0 auto;
  display: grid;
  gap: 1rem;
  text-align: center;
}
.title {
  margin: 0;
  font-size: clamp(28px, 5.5vw, 54px);
}
.subtitle {
  margin: 0;
  font-size: clamp(14px, 2.4vw, 18px);
  opacity: 0.9;
}
.media-placeholder {
  height: clamp(180px, 40vh, 360px);
  border: 2px dashed rgba(255, 255, 255, 0.35);
  border-radius: 16px;
  display: grid;
  place-items: center;
  opacity: 0.7;
}
.features {
  list-style: none;
  padding: 0;
  margin: 0.25rem 0 0;
  display: grid;
  gap: 0.25rem;
  opacity: 0.95;
}
.grid {
  display: grid;
  grid-template-columns: repeat(3, minmax(0, 1fr));
  gap: 12px;
}
.card {
  padding: 24px;
  border-radius: 12px;
  background: rgba(0, 0, 0, 0.22);
  backdrop-filter: blur(2px);
}
.button {
  margin: 0.25rem auto 0;
  padding: 0.9rem 1.25rem;
  border-radius: 0.6rem;
  border: 0;
  background: #111827;
  color: #fff;
  font-weight: 700;
  text-decoration: none;
  display: inline-block;
}
.button:hover {
  filter: brightness(1.1);
}

.slot-media,
.slot-text {
  display: contents;
}

.s1 {
  background: #0ea5e9;
}
.s2 {
  background: #fff;
}
.s3 {
  background: #f97316;
}
.s4 {
  background: #7c3aed;
}
</style>
