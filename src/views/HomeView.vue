<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref, nextTick } from 'vue'
import HorizontalSlides from '@/components/HorizontalSlides.vue'
import gsap from 'gsap'
import { ScrollToPlugin } from 'gsap/ScrollToPlugin'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollToPlugin, ScrollTrigger)

const skillsOpen = ref(false)
let skillsTimer: number | undefined

const contactOpen = ref(false)
let contactTimer: number | undefined

const resumeOpen = ref(false)

// ===== Spline =====
const showViewer = ref(false)
const viewerVisible = ref(false)
const placeholderVisible = ref(true)
let splineScriptLoaded = false
const TRANSITION_MS = 160

// ===== cursor rojo cuando está apagado =====
const cursorX = ref(0)
const cursorY = ref(0)
let targetX = 0
let targetY = 0
let currentX = 0
let currentY = 0
let rafId: number | null = null

function onMouseMove(e: MouseEvent) {
  // posición objetivo, desplazada un poco
  targetX = e.clientX + 16
  targetY = e.clientY + 10

  if (!rafId) animateCursor()
}

function animateCursor() {
  // suavizado: cuanto más bajo el factor, más lento sigue el puntero
  const ease = 0.12

  currentX += (targetX - currentX) * ease
  currentY += (targetY - currentY) * ease

  cursorX.value = currentX
  cursorY.value = currentY

  rafId = requestAnimationFrame(animateCursor)
}

// ===== Marca (gsusgil) =====
const brandVariants = [
  { weight: 300, color: '#ffd4d4' },
  { weight: 400, color: '#ffb3b3' },
  { weight: 500, color: '#ff8c8c' },
  { weight: 600, color: '#ff6b6b' },
  { weight: 700, color: '#ff4d4d' }, // base
  { weight: 800, color: '#ff3030' },
  { weight: 900, color: '#ff1111' },
]

const brandIndex = ref(4) // base = 700
const brandWeight = ref(brandVariants[4].weight)
const brandColor = ref(brandVariants[4].color)

let brandTimer: number | undefined
let brandKickTimer: number | undefined

function setBrand(idx: number) {
  brandIndex.value = idx
  const v = brandVariants[idx]
  brandWeight.value = v.weight
  brandColor.value = v.color
}

// drift corto entre 600–800 para que no parezca glitch
function brandDrift() {
  const current = brandIndex.value
  // nos movemos solo entre 3 valores: 600 (3), 700 (4), 800 (5)
  const allowed = [3, 4, 5]
  // quita el actual de la lista para no repetir
  const options = allowed.filter((i) => i !== current)
  const next = options[Math.floor(Math.random() * options.length)]
  setBrand(next)
}

function onBrandEnter() {
  // limpiamos todo
  clearInterval(brandTimer)
  clearTimeout(brandKickTimer)

  // 1) pequeño "kick" rápido
  setBrand(5) // 800 / rojo un poco más fuerte

  // 2) después de 140ms volvemos cerca del base
  brandKickTimer = window.setTimeout(() => {
    setBrand(4)

    // 3) y empezamos el drift más lento / natural
    brandTimer = window.setInterval(() => {
      brandDrift()
    }, 420) // este ritmo es más orgánico
  }, 140)
}

function onBrandLeave() {
  clearInterval(brandTimer)
  clearTimeout(brandKickTimer)
  setBrand(4) // back to base
}

// ===== Skills hover =====
function onSkillsEnter() {
  clearTimeout(skillsTimer)
  skillsTimer = window.setTimeout(() => (skillsOpen.value = true), 90)
}
function onSkillsLeave() {
  clearTimeout(skillsTimer)
  skillsTimer = window.setTimeout(() => (skillsOpen.value = false), 120)
}

// ===== Contact hover =====
function onContactEnter() {
  clearTimeout(contactTimer)
  contactTimer = window.setTimeout(() => (contactOpen.value = true), 90)
}
function onContactLeave() {
  clearTimeout(contactTimer)
  contactTimer = window.setTimeout(() => (contactOpen.value = false), 120)
}

// ===== Contact data =====
const CONTACT_EMAIL = 'tuemail@correo.com'
const copied = ref(false)

function copyEmail() {
  if (navigator?.clipboard?.writeText) {
    navigator.clipboard.writeText(CONTACT_EMAIL).then(
      () => {
        copied.value = true
        setTimeout(() => {
          copied.value = false
        }, 1200)
      },
      () => {
        alert('Copy this:\n' + CONTACT_EMAIL)
      },
    )
  } else {
    alert('Copy this:\n' + CONTACT_EMAIL)
  }
}

function openResume() {
  resumeOpen.value = true
  document.documentElement.style.overflow = 'hidden'
}
function closeResume() {
  resumeOpen.value = false
  document.documentElement.style.overflow = ''
}

function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape') {
    skillsOpen.value = false
    contactOpen.value = false
    closeResume()
  }
}
onMounted(() => {
  document.addEventListener('keydown', onKey)
  document.addEventListener('mousemove', onMouseMove)
})
onBeforeUnmount(() => {
  document.removeEventListener('keydown', onKey)
  document.removeEventListener('mousemove', onMouseMove)
  clearInterval(brandTimer)
  if (rafId) cancelAnimationFrame(rafId)
})

function ensureSplineLoaded() {
  if (splineScriptLoaded) return
  const id = 'spline-viewer-loader'
  if (!document.getElementById(id)) {
    const s = document.createElement('script')
    s.type = 'module'
    s.id = id
    s.src = 'https://unpkg.com/@splinetool/viewer@1.10.77/build/spline-viewer.js'
    s.onload = () => (splineScriptLoaded = true)
    document.head.appendChild(s)
  } else splineScriptLoaded = true
}

async function toggleSpline() {
  // ON →
  if (!showViewer.value) {
    ensureSplineLoaded()
    showViewer.value = true
    placeholderVisible.value = true
    viewerVisible.value = false
    await nextTick()
    requestAnimationFrame(() => {
      viewerVisible.value = true
      gsap.to('.brand-off', {
        opacity: 0,
        y: 10,
        duration: TRANSITION_MS / 1000,
        ease: 'power1.out',
      })
      setTimeout(() => (placeholderVisible.value = false), TRANSITION_MS)
    })
    return
  }

  // OFF →
  placeholderVisible.value = true
  gsap.fromTo(
    '.brand-off',
    { opacity: 0, y: -10 },
    { opacity: 1, y: 0, duration: TRANSITION_MS / 1000, ease: 'power1.out' },
  )
  viewerVisible.value = false
  setTimeout(() => {
    showViewer.value = false
  }, TRANSITION_MS)
}
</script>

<template>
  <section class="hero">
    <!-- cursor rojo SOLO cuando el spline está apagado -->
    <div
      v-if="!showViewer"
      class="cursor-dot"
      :style="{ left: cursorX + 'px', top: cursorY + 'px' }"
    ></div>

    <!-- Stack para cross-fade -->
    <div class="stack">
      <spline-viewer
        v-if="showViewer"
        id="spline"
        class="viewer"
        :class="{ 'is-visible': viewerVisible }"
        url="https://prod.spline.design/59dhmsvB8QAaBOBl/scene.splinecode"
      />
      <div v-show="placeholderVisible" class="spline-off viewer is-visible">
        <!-- título -->
        <span class="brand-off" @mouseenter="onBrandEnter" @mouseleave="onBrandLeave">
          <span class="brand-off__inner" :style="{ fontWeight: brandWeight, color: brandColor }">
            gsusgil
          </span>
        </span>
      </div>
    </div>

    <!-- UI -->
    <div class="ui">
      <div class="top-row">
        <!-- Skills -->
        <div class="slot slot--left" @mouseenter="onSkillsEnter" @mouseleave="onSkillsLeave">
          <button class="link strong" :aria-expanded="skillsOpen">Skills</button>
          <div class="skills panel" :class="{ open: skillsOpen }">
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

        <!-- Resume -->
        <div class="slot slot--center">
          <button class="link" @click="openResume">Resume</button>
        </div>

        <!-- Contact -->
        <div class="slot slot--right" @mouseenter="onContactEnter" @mouseleave="onContactLeave">
          <button class="link">Contact</button>
          <div class="contacts panel" :class="{ open: contactOpen }">
            <ul class="group">
              <li class="contact-item email-item" @click="copyEmail">
                gsusgildesigner@gmail.com
                <span v-if="copied" class="copied">Copied!</span>
              </li>
              <li>
                <a href="https://www.linkedin.com/in/gsus-gil/" target="_blank" rel="noopener"
                  >Jesus David Gil</a
                >
              </li>
              <li>
                <span class="placeholder">+34 685 656 681</span>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <!-- Switch -->
      <div class="bottom-row">
        <div class="spline-switch-wrapper" @click="toggleSpline">
          <div class="spline-switch" :class="{ on: showViewer }">
            <div class="spline-thumb"></div>
          </div>
          <!-- ✅ texto según estado -->
          <span class="spline-switch-label">
            {{ showViewer ? 'hit me again' : 'hit me' }}
          </span>
          <div class="spline-tooltip">
            <p v-if="!showViewer">HIT MEto load the spline</p>
            <p v-else>HIT ME AGAIN to turn it off & continue navigating</p>
          </div>
        </div>
      </div>
    </div>
  </section>

  <!-- Modal Resume -->
  <teleport to="body">
    <div v-if="resumeOpen" class="backdrop" @click="closeResume">
      <div class="modal" @click.stop>
        <button class="close" @click="closeResume" aria-label="Cerrar modal">×</button>
        <iframe class="doc" src="/public/documents/JesusGil_CV_GraphicDesigner.pdf" title="Resume"></iframe>

        <div class="download-wrap">
          <a href="/resume.pdf" download class="download-btn">Descargar PDF</a>
        </div>
      </div>
    </div>
  </teleport>

  <HorizontalSlides id="slides" />
</template>

<style scoped>
.hero {
  position: relative;
  height: 100vh;
  overflow: hidden;
}

.stack {
  position: absolute;
  inset: 0;
  background: #fff;
}

/* cursor rojo */
.cursor-dot {
  position: fixed;
  width: 10px;
  height: 10px;
  background: #ff4d4d;
  border-radius: 50%;
  pointer-events: none;
  transform: translate(-50%, -50%);
  z-index: 999;
}

/* Spline viewer */
.viewer {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  transition: opacity 160ms ease;
}
.viewer.is-visible {
  opacity: 1;
}
#spline {
  background: #fff;
  width: 100%;
  height: 100%;
}

/* Placeholder */
.spline-off {
  display: grid;
  place-items: center;
}

/* Marca */
.brand-off {
  font-family: var(--font-swiss, sans-serif);
  font-size: 2rem;
  letter-spacing: 0.7em;
  opacity: 1;
  transition:
    opacity 0.16s ease,
    transform 0.16s ease;
  cursor: pointer;
}
.brand-off__inner {
  display: inline-block;
  transition:
    font-weight 0.12s ease-out,
    color 0.22s ease-out,
    transform 0.16s ease;
}

.brand-off:hover .brand-off__inner {
  transform: translateY(-1px);
}

/* UI general */
.ui {
  position: absolute;
  inset: 0;
  z-index: 20;
  pointer-events: none;
  --side-pad: clamp(28px, 7vw, 120px);
  --top-off: 88px;
  --bot-off: 96px;
  --txt: #0f172a;
  --muted: rgba(15, 23, 42, 0.78);
  color: var(--txt);
}

/* tu spacing se queda */
.top-row {
  position: absolute;
  top: var(--top-off);
  left: 50%;
  transform: translateX(-50%);
  display: flex;
  align-items: flex-start;
  gap: 25rem;
}

.slot {
  position: relative;
  pointer-events: auto;
}

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

/* ========= dropdowns ========= */
/* base */
/* dropdowns alineados al botón */
.panel {
  position: absolute;
  top: calc(100% + 0.55rem);
  display: grid;
  gap: 1.1rem;
  opacity: 0;
  transform: translate(-50%, -8px); /* centrado ↓ */
  transition:
    opacity 0.28s cubic-bezier(0.22, 0.61, 0.36, 1),
    transform 0.28s cubic-bezier(0.22, 0.61, 0.36, 1);
  pointer-events: none;
  z-index: 30;
  left: 50%; /* por defecto, centro */
}

/* TODOS los slots → centro bajo el botón */
.slot--left .panel,
.slot--center .panel,
.slot--right .panel {
  left: 50%;
  right: auto;
  transform: translate(-50%, -8px);
}

.panel.open {
  opacity: 1;
  transform: translate(-50%, 0);
  pointer-events: auto;
}

/* lista base (skills + contact) */
.group {
  list-style: none;
  margin: 0;
  padding: 0;
  display: grid;
  gap: 0.35rem;
}
.group li {
  font-weight: 300;
  color: var(--muted);
  font-size: clamp(12px, 1.7vw, 15px);
  line-height: 1.4;
  opacity: 0;
  transform: translateY(-6px);
  transition:
    opacity 0.35s cubic-bezier(0.22, 0.61, 0.36, 1),
    transform 0.35s cubic-bezier(0.22, 0.61, 0.36, 1);
  --g: 0;
  --i: 0;
  transition-delay: calc(var(--g) * 140ms + var(--i) * 40ms);
}

/* entrada escalonada */
.panel.open .group li {
  opacity: 1;
  transform: translateY(0);
}

/* ===== CONTACTS (mismo estilo que skills) ===== */
.contacts.panel .group {
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
}

.contacts.panel .contact-item,
.contacts.panel li {
  display: inline-flex;
  align-items: center;
  gap: 0.45rem;
  white-space: nowrap;
  font-size: clamp(12px, 1.7vw, 15px);
  line-height: 1.4;
  color: var(--muted);
  font-weight: 300;
}

/* email → clicable y con mano */
.contacts.panel .email-item,
.contacts.panel .email-item * {
  cursor: pointer;
}
.contacts.panel .email-item:hover {
  opacity: 1;
  color: #0f172a;
  text-decoration: underline;
}

/* LinkedIn → clicable */
.contacts.panel a {
  cursor: pointer;
  text-decoration: none;
  color: var(--muted);
  opacity: 0.7;
  transition:
    opacity 0.15s ease,
    color 0.15s ease;
}
.contacts.panel a:hover {
  opacity: 1;
  color: #0f172a;
  text-decoration: underline;
}

/* Teléfono */
.contacts.panel .placeholder {
  opacity: 0.7;
  color: var(--muted);
}

/* badge "Copied!" */
.copied {
  font-size: 0.65rem;
  background: #ffffff;
  color: #000000;
  padding: 0.1rem 0.55rem;
  border-radius: 999px;
  border: 1px solid rgba(0, 0, 0, 0.08);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.06);
}

/* skills: dentro cursor normal */
:deep(.skills.panel),
:deep(.skills.panel *) {
  cursor: default;
}

/* bottom switch */
.bottom-row {
  position: absolute;
  left: 0;
  right: 0;
  bottom: var(--bot-off);
  display: grid;
  place-items: center;
  pointer-events: auto;
  isolation: isolate;
  z-index: 1;
}
.spline-switch-wrapper {
  position: relative;
  display: inline-flex;
  align-items: center;
  gap: 0.75rem;
  cursor: pointer;
}
.spline-switch {
  width: 48px;
  height: 26px;
  background: #cbd5e1;
  border-radius: 999px;
  position: relative;
  transition: background 0.18s ease;
}
.spline-switch.on {
  background: #8ff5b5;
}
.spline-thumb {
  position: absolute;
  top: 2px;
  left: 2px;
  width: 22px;
  height: 22px;
  border-radius: 50%;
  background: #fff;
  transition: transform 0.18s ease;
}
.spline-switch.on .spline-thumb {
  transform: translateX(22px);
}
.spline-switch-label {
  font-weight: 700;
  color: var(--muted);
  user-select: none;
  text-transform: lowercase;
}
.spline-tooltip {
  position: absolute;
  bottom: 130%;
  left: 50%;
  transform: translateX(-50%) translateY(4px);
  opacity: 0;
  background: #0f172a;
  color: #fff;
  font-size: 0.8rem;
  padding: 0.5rem 0.8rem;
  border-radius: 6px;
  text-align: center;
  line-height: 1.4;
  width: max-content;
  max-width: 220px;
  pointer-events: none;
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
  z-index: 2;
}
.spline-switch-wrapper:hover .spline-tooltip {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}

/* modal resume */
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
  display: flex;
  flex-direction: column;
}
.doc {
  width: 100%;
  height: 100%;
  border: 0;
  flex: 1 1 auto;
}
.close {
  position: absolute;
  top: 8px;
  right: 10px;
  background: transparent;
  border: 0;
  font-size: 28px;
  cursor: pointer;
  line-height: 1;
}
.download-wrap {
  flex: 0 0 auto;
  display: grid;
  place-items: center;
  padding: 0.8rem 1rem 1.1rem;
  background: #fff;
  border-top: 1px solid rgba(15, 23, 42, 0.06);
}
.download-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  background: #0f172a;
  color: #fff;
  padding: 0.45rem 1.05rem;
  border-radius: 999px;
  font-size: 0.85rem;
  font-weight: 500;
  text-decoration: none;
}
.download-btn:hover {
  opacity: 0.9;
}

/* responsive */
@media (max-width: 640px) {
  .ui {
    --top-off: 64px;
    --bot-off: 64px;
  }
  .top-row {
    gap: 2.5rem;
  }
  .modal {
    height: 80vh;
    padding-top: 0;
  }
}
</style>
