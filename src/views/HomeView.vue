<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref } from 'vue'
import HorizontalSlides from '@/components/HorizontalSlides.vue'

// 👇 Añadido: GSAP + ScrollToPlugin para scroll suave sin pelearse con el pin
import gsap from 'gsap'
import { ScrollToPlugin } from 'gsap/ScrollToPlugin'
gsap.registerPlugin(ScrollToPlugin, ScrollTrigger)
import { ScrollTrigger } from 'gsap/ScrollTrigger'

const skillsOpen = ref(false)
let skillsTimer: number | undefined
const resumeOpen = ref(false)

function onSkillsEnter() {
  clearTimeout(skillsTimer)
  skillsTimer = window.setTimeout(() => (skillsOpen.value = true), 90) // hover-intent ~90ms
}
function onSkillsLeave() {
  clearTimeout(skillsTimer)
  skillsTimer = window.setTimeout(() => (skillsOpen.value = false), 120) // cierra suave
}
function openResume() {
  resumeOpen.value = true
  document.documentElement.style.overflow = 'hidden'
}
function closeResume() {
  resumeOpen.value = false
  document.documentElement.style.overflow = ''
}

// 👇 Reemplazado: animación suave con GSAP ScrollTo (NO nativo)
function toProjects() {
  const el = document.getElementById('slides')
  if (!el) return
  gsap.killTweensOf(window)
  gsap.to(window, {
    duration: 0.9,
    ease: 'power2.out',
    scrollTo: { y: el, autoKill: true, offsetY: 2 }, // 👈 2px fuera del borde
    overwrite: 'auto',
  })
}

function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape') {
    skillsOpen.value = false
    closeResume()
  }
}

function toggleSkills() {
  skillsOpen.value = !skillsOpen.value
}

onMounted(() => {
  const id = 'spline-viewer-loader'
  if (!document.getElementById(id)) {
    const s = document.createElement('script')
    s.type = 'module'
    s.id = id
    s.src = 'https://unpkg.com/@splinetool/viewer@1.10.77/build/spline-viewer.js'
    document.head.appendChild(s)
  }
  document.addEventListener('keydown', onKey)
})
onBeforeUnmount(() => document.removeEventListener('keydown', onKey))
</script>

<template>
  <section class="hero">
    <spline-viewer id="spline" url="https://prod.spline.design/59dhmsvB8QAaBOBl/scene.splinecode" />

    <!-- 👇 UI SOLO DEL HOME -->
    <div class="ui">
      <div class="top-row">
        <div class="slot slot--left" @mouseenter="onSkillsEnter" @mouseleave="onSkillsLeave">
          <button class="link strong" @click="toggleSkills" :aria-expanded="skillsOpen">
            Skills
          </button>
          <div class="skills" :class="{ open: skillsOpen }" role="menu" aria-label="Habilidades">
            <ul class="group">
              <li>Photoshop</li>
              <li>Illustrator</li>
              <li>Indesign</li>
              <li>After Effects</li>
              <li>Figma</li>
              <li>Spline</li>
            </ul>
            <ul class="group">
              <li>Html</li>
              <li>Css</li>
              <li>Javascript</li>
            </ul>
            <ul class="group">
              <li>Hubspot</li>
            </ul>
          </div>
        </div>

        <div class="slot slot--center">
          <button class="link" @click="openResume">Resume</button>
        </div>

        <div class="slot slot--right">
          <button class="link">Contact</button>
        </div>
      </div>

      <div class="bottom-row">
        <button class="projects" @click="toProjects" aria-label="Ver proyectos">
          <span>Projects</span>
          <svg class="chev" viewBox="0 0 24 24" width="28" height="28" aria-hidden="true">
            <path
              d="M6 9l6 6 6-6"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </button>
      </div>
    </div>
    <!-- 👆 UI SOLO DEL HOME -->
  </section>

  <teleport to="body">
    <!-- modal resume igual -->
  </teleport>

  <HorizontalSlides id="slides" />
</template>

<style scoped>
/* ===== Fondo ===== */
.hero {
  position: relative; /* importante para que la UI se ancle aquí */
  height: 100svh;
  height: 100lvh;
  height: 100vh;
  overflow: hidden;
}
#spline {
  display: block;
  width: 100%;
  height: 100%;
  /* 👇 opcional: ayuda a evitar reproyección cuando vuelves desde el pin */
  backface-visibility: hidden;
  contain: paint;
  transform: translateZ(0);
}

/* Antes estaba fixed. Debe ser absolute para que se vaya al scrollear. */
.ui {
  position: absolute; /* <— aquí el cambio */
  inset: 0;
  z-index: 20;
  pointer-events: none;
  /* márgenes */
  --side-pad: clamp(28px, 7vw, 120px);
  --top-off: 88px;
  --bot-off: 96px;
  --txt: #0f172a;
  --muted: rgba(15, 23, 42, 0.78);
  color: var(--txt);
}

/* 3 textos arriba */
.top-row {
  position: absolute;
  left: 0;
  right: 0;
  top: var(--top-off, 88px);
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: start;
  padding: 0 var(--side-pad, 80px);
}
.slot {
  position: relative;
  pointer-events: auto;
}
.slot--left {
  justify-self: start;
}
.slot--center {
  justify-self: center;
}
.slot--right {
  justify-self: end;
}

/* Botones de texto */
.link {
  background: transparent;
  border: 0;
  padding: 0;
  cursor: pointer;
  letter-spacing: 0.2px;
  font-weight: 600;
}
.link.strong {
  font-weight: 700;
}
.link:hover {
  opacity: 0.85;
}

/* Panel Skills: easing tipo “springy”, sin fondo */
/* Panel Skills */
.skills {
  position: absolute;
  top: 2.1rem;
  left: 0;
  display: grid;
  gap: 1.1rem;
  opacity: 0;
  transform: translateY(-8px); /* cae desde ARRIBA */
  transition:
    opacity 0.28s cubic-bezier(0.22, 0.61, 0.36, 1),
    transform 0.28s cubic-bezier(0.22, 0.61, 0.36, 1);
  pointer-events: none;
  will-change: transform, opacity;
}
.slot--left:hover .skills,
.skills.open {
  opacity: 1;
  transform: translateY(0);
  pointer-events: auto;
}

/* Grupos + stagger top→bottom (entre grupos e ítems) */
.group {
  list-style: none;
  margin: 0;
  padding: 0;
  display: grid;
  gap: 0.35rem;
}

/* Valores base para delay usando CSS vars */
.group li {
  /* animación por ítem: fade + caída corta */
  font-weight: 300;
  color: var(--muted);
  font-size: clamp(12px, 1.7vw, 15px);
  line-height: 1.4;

  opacity: 0;
  transform: translateY(-6px); /* cae desde ARRIBA */
  transition:
    opacity 0.35s cubic-bezier(0.22, 0.61, 0.36, 1),
    transform 0.35s cubic-bezier(0.22, 0.61, 0.36, 1);

  /* índice de grupo e índice de item -> delay escalonado */
  --g: 0; /* lo seteo abajo por grupo */
  --i: 0; /* lo seteo abajo por item  */
  transition-delay: calc(var(--g) * 140ms + var(--i) * 40ms);
}

/* Activación */
.skills.open .group li,
.slot--left:hover .group li {
  opacity: 1;
  transform: translateY(0);
}

/* Índice por grupo (top→bottom) */
.skills .group:nth-child(1) li {
  --g: 0;
} /* primer bloque (arriba) */
.skills .group:nth-child(2) li {
  --g: 1;
}
.skills .group:nth-child(3) li {
  --g: 2;
}

/* Índice por item dentro del grupo (top→bottom) */
.skills .group li:nth-child(1) {
  --i: 0;
}
.skills .group li:nth-child(2) {
  --i: 1;
}
.skills .group li:nth-child(3) {
  --i: 2;
}
.skills .group li:nth-child(4) {
  --i: 3;
}
.skills .group li:nth-child(5) {
  --i: 4;
}
.skills .group li:nth-child(6) {
  --i: 5;
}
.skills .group li:nth-child(7) {
  --i: 6;
}
.skills .group li:nth-child(8) {
  --i: 7;
}
.skills .group li:nth-child(9) {
  --i: 8;
}
.skills .group li:nth-child(10) {
  --i: 9;
}

/* Texto abajo centrado */
.bottom-row {
  position: absolute;
  left: 0;
  right: 0;
  bottom: var(--bot-off, 96px);
  display: grid;
  place-items: center;
}
.projects {
  pointer-events: auto;
  background: transparent;
  border: 0;
  padding: 0;
  display: grid;
  place-items: center;
  gap: 0.35rem;
  color: var(--muted);
  font-weight: 700;
  cursor: pointer;
}
.projects:hover .chev {
  transform: translateY(2px);
  transition: transform 0.15s ease;
}

/* Modal */
.backdrop {
  position: fixed;
  inset: 0;
  z-index: 30;
  background: rgba(255, 255, 255, 0.7);
  display: grid;
  place-items: center;
  padding: 2rem;
}
.modal {
  position: relative;
  width: min(1000px, 92vw);
  height: min(80vh, 900px);
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 18px 60px rgba(0, 0, 0, 0.18);
  overflow: hidden;
}
.doc {
  width: 100%;
  height: 100%;
  border: 0;
}
.close {
  position: absolute;
  top: 8px;
  right: 10px;
  background: transparent;
  border: 0;
  font-size: 28px;
  cursor: pointer;
}

@media (max-width: 640px) {
  .ui {
    --top-off: 64px;
    --bot-off: 64px;
  }
}
</style>
