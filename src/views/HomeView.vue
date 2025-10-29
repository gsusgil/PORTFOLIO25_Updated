<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref, nextTick } from 'vue'
import HorizontalSlides from '@/components/HorizontalSlides.vue'
import gsap from 'gsap'
import { ScrollToPlugin } from 'gsap/ScrollToPlugin'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
gsap.registerPlugin(ScrollToPlugin, ScrollTrigger)

const skillsOpen = ref(false)
let skillsTimer: number | undefined
const resumeOpen = ref(false)

// ===== Spline =====
const showViewer = ref(false)
const viewerVisible = ref(false)
const placeholderVisible = ref(true)
let splineScriptLoaded = false
const TRANSITION_MS = 160

function onSkillsEnter() {
  clearTimeout(skillsTimer)
  skillsTimer = window.setTimeout(() => (skillsOpen.value = true), 90)
}
function onSkillsLeave() {
  clearTimeout(skillsTimer)
  skillsTimer = window.setTimeout(() => (skillsOpen.value = false), 120)
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
    closeResume()
  }
}
onMounted(() => document.addEventListener('keydown', onKey))
onBeforeUnmount(() => document.removeEventListener('keydown', onKey))

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
      // Fade out placeholder con GSAP para evitar solapamiento
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
        <span class="brand-off">gsusgil</span>
      </div>
    </div>

    <!-- UI -->
    <div class="ui">
      <div class="top-row">
        <div class="slot slot--left" @mouseenter="onSkillsEnter" @mouseleave="onSkillsLeave">
          <button class="link strong" :aria-expanded="skillsOpen">Skills</button>
          <div class="skills" :class="{ open: skillsOpen }">
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

      <!-- Switch -->
      <div class="bottom-row">
        <div class="spline-switch-wrapper" @click="toggleSpline">
          <div class="spline-switch" :class="{ on: showViewer }">
            <div class="spline-thumb"></div>
          </div>
          <span class="spline-switch-label">{{ showViewer ? 'ON' : 'OFF' }}</span>
          <div class="spline-tooltip">
            <p v-if="!showViewer">Click me and discover one new function</p>
            <p v-else>Off please shut me down for continue navigate.</p>
          </div>
        </div>
      </div>
    </div>
  </section>

  <!-- Modal Resume -->
  <teleport to="body">
    <div v-if="resumeOpen" class="backdrop">
      <div class="modal">
        <button class="close" @click="closeResume" aria-label="Close">×</button>
        <iframe class="doc" src="/resume.pdf" title="Resume"></iframe>
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

/* Fade cross */
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
.brand-off {
  color: #e10600;
  font-family: var(--font-swiss, sans-serif);
  font-size: 2rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  opacity: 1;
  transition:
    opacity 0.16s ease,
    transform 0.16s ease;
}

/* UI */
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
.top-row {
  position: absolute;
  left: 0;
  right: 0;
  top: var(--top-off);
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  align-items: start;
  padding: 0 var(--side-pad);
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

/* Skills */
.skills {
  position: absolute;
  top: 2.1rem;
  left: 0;
  display: grid;
  gap: 1.1rem;
  opacity: 0;
  transform: translateY(-8px);
  transition:
    opacity 0.28s cubic-bezier(0.22, 0.61, 0.36, 1),
    transform 0.28s cubic-bezier(0.22, 0.61, 0.36, 1);
  pointer-events: none;
}
.slot--left:hover .skills,
.skills.open {
  opacity: 1;
  transform: translateY(0);
  pointer-events: auto;
}
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
.skills.open .group li,
.slot--left:hover .group li {
  opacity: 1;
  transform: translateY(0);
}
.skills .group:nth-child(1) li {
  --g: 0;
}
.skills .group:nth-child(2) li {
  --g: 1;
}
.skills .group:nth-child(3) li {
  --g: 2;
}
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

/* Bottom */
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

/* Switch */
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
  background: #0ea5e9;
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
