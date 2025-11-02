<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount } from 'vue'

/* === CONFIG === */
const BASE = import.meta.env.BASE_URL || '/'
const IMAGES = ['story1.jpg', 'story2.jpg', 'story3.jpg', 'story4.jpg', 'story5.jpg', 'story6.jpg']

const stage = ref<HTMLDivElement | null>(null)
const rig = ref<HTMLDivElement | null>(null)
const front = ref<HTMLDivElement | null>(null)
const back = ref<HTMLDivElement | null>(null)
const btnPrev = ref<HTMLButtonElement | null>(null)
const btnNext = ref<HTMLButtonElement | null>(null)
const btnToggle = ref<HTMLButtonElement | null>(null)

/* === Estado (igual al vanilla) === */
let angle = 0,
  spinning = true,
  last: number | null = null
let rafSpin = 0,
  rafTilt = 0
const SPEED_SECONDS = 18
const degPerMs = 360 / (SPEED_SECONDS * 1000)

let cardsF: HTMLDivElement[] = []
let cardsB: HTMLDivElement[] = []
let n = 0
let theta = 0

/* separación entre tarjetas */
const AIR_PX = 90

function computeRadius() {
  if (!cardsF[0]) return 0
  const w = parseFloat(getComputedStyle(cardsF[0]).width) || 1
  const base = Math.round(w / 2 / Math.tan(Math.PI / n))
  return base + AIR_PX
}
function layout() {
  const radius = computeRadius()
  cardsF.forEach((p, i) => {
    p.style.setProperty('--angle', `${i * theta}deg`)
    p.style.setProperty('--radius', `${radius}px`)
  })
  cardsB.forEach((p, i) => {
    p.style.setProperty('--angle', `${i * theta}deg`)
    p.style.setProperty('--radius', `${radius}px`)
  })
}

function render(a: number) {
  if (!front.value || !back.value) return
  front.value.style.transform = `rotateY(${a}deg)`
  back.value.style.transform = `rotateY(${a}deg)`
}
function updateFront(a: number) {
  if (!n) return
  const idx = ((Math.round(-a / theta) % n) + n) % n
  cardsF.forEach((p) => p.classList.remove('is-front'))
  cardsF[idx]?.classList.add('is-front')
}

/* giro continuo */
function tick(ts: number) {
  if (last == null) last = ts
  const dt = ts - last
  last = ts
  if (spinning) {
    angle -= degPerMs * dt
    render(angle)
    updateFront(angle)
  }
  rafSpin = requestAnimationFrame(tick)
}

/* Look-At + rotación ligada */
const MAX_TILT_X = 8,
  MAX_TILT_Y = 14
let tiltX = 0,
  tiltY = 0,
  targetX = 0,
  targetY = 0,
  extraRot = 0
function setRigTilt(x: number, y: number) {
  const r = stage.value!.getBoundingClientRect()
  const nx = ((x - r.left) / r.width) * 2 - 1
  const ny = ((y - r.top) / r.height) * 2 - 1
  targetX = ny * -MAX_TILT_X
  targetY = nx * MAX_TILT_Y
  extraRot = nx * 10
}
function updateTilt() {
  if (!rig.value) return
  tiltX += (targetX - tiltX) * 0.1
  tiltY += (targetY - tiltY) * 0.1
  rig.value.style.transform = `rotateX(${tiltX}deg) rotateY(${tiltY}deg)`
  render(angle + extraRot)
  rafTilt = requestAnimationFrame(updateTilt)
}

/* Controles + drag */
function rotateSteps(steps: number) {
  angle += -steps * theta
  render(angle)
  updateFront(angle)
}
function updateToggleIcon() {
  if (!btnToggle.value) return
  btnToggle.value.textContent = '⏸'
  btnToggle.value.style.opacity = spinning ? '1' : '.5'
}
let dragging = false,
  startX = 0,
  startAngle = 0
function onDown(e: MouseEvent) {
  dragging = true
  startX = e.clientX
  startAngle = angle
  spinning = false
  updateToggleIcon()
}
function onMove(e: MouseEvent) {
  if (!dragging) return
  angle = startAngle + (e.clientX - startX) * -0.2
  render(angle)
  updateFront(angle)
}
function onUp() {
  dragging = false
}

function onResize() {
  layout()
}

/* Mount */
onMounted(() => {
  const $front = front.value!,
    $back = back.value!
  $front.innerHTML = ''
  $back.innerHTML = ''

  // crea 6 tarjetas (frente+reverso) como en el vanilla
  IMAGES.forEach((file) => {
    const f = document.createElement('div')
    f.className = 's2-card'

    const img = document.createElement('img')
    img.alt = ''
    img.src = `${BASE}image/${file}` // 👈 sin inventar @1x/@2x
    f.appendChild(img)

    front.value!.appendChild(f)

    const b = document.createElement('div')
    b.className = 's2-card-back'
    back.value!.appendChild(b)
  })

  cardsF = Array.from($front.querySelectorAll<HTMLDivElement>('.s2-card'))
  cardsB = Array.from($back.querySelectorAll<HTMLDivElement>('.s2-card-back'))
  n = cardsF.length
  theta = 360 / n

  // espera un frame para que el CSS (no scoped) ya esté aplicado y el width sea real
  requestAnimationFrame(() => {
    layout()
    updateFront(0)
    render(0)
    rafSpin = requestAnimationFrame(tick)
    rafTilt = requestAnimationFrame(updateTilt)
  })

  // eventos
  stage.value!.addEventListener('pointermove', (e) => setRigTilt(e.clientX, e.clientY))
  stage.value!.addEventListener('pointerleave', () => {
    targetX = 0
    targetY = 0
    extraRot = 0
  })
  front.value!.addEventListener('mousedown', onDown)
  window.addEventListener('mousemove', onMove)
  window.addEventListener('mouseup', onUp)
  window.addEventListener('resize', onResize, { passive: true })

  btnNext.value!.addEventListener('click', () => rotateSteps(1))
  btnPrev.value!.addEventListener('click', () => rotateSteps(-1))
  btnToggle.value!.addEventListener('click', () => {
    spinning = !spinning
    updateToggleIcon()
  })
  updateToggleIcon()

  if (matchMedia('(prefers-reduced-motion: reduce)').matches) {
    spinning = false
    updateToggleIcon()
  }
})

onBeforeUnmount(() => {
  cancelAnimationFrame(rafSpin)
  cancelAnimationFrame(rafTilt)
  window.removeEventListener('mousemove', onMove)
  window.removeEventListener('mouseup', onUp)
  window.removeEventListener('resize', onResize)
})
</script>

<template>
  <!-- Mantén .panel para que GSAP cuente este slide -->
  <div class="panel slide2">
    <div class="s2-grid">
      <!-- COLUMNA IZQUIERDA: CARRUSEL -->
      <div class="s2-left">
        <div class="s2-frame">
          <div class="s2-anchor">
            <div class="s2-stage" ref="stage" aria-label="Carrusel 3D">
              <div class="s2-rig" ref="rig">
                <div class="s2-carousel" ref="front"></div>
                <div class="s2-carousel" ref="back" aria-hidden="true"></div>
              </div>
            </div>

            <div class="s2-controls">
              <button class="s2-btn" ref="btnPrev" aria-label="Anterior">⟲</button>
              <button class="s2-btn" ref="btnToggle" aria-label="Play/Pause">⏸</button>
              <button class="s2-btn" ref="btnNext" aria-label="Siguiente">⟳</button>
            </div>
          </div>
        </div>
      </div>

      <!-- COLUMNA DERECHA: COPY -->
      <div class="s2-right">
        <div class="s2-copy">
          <h2 class="title">Lorem Ipsum is simply dummy</h2>
          <p class="subtitle">
            Lorem Ipsum is simply dummy text of the printing and typesetting industry.
          </p>
          <p class="body slot-text">
            Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an
            unknown printer took a galley of type and scrambled it to make a type specimen book. It
            has survived not only five centuries, but also the leap into electronic typesetting,
            remaining essentially unchanged.
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<!-- IMPORTANTE: SIN 'scoped' para que afecte a las tarjetas creadas por JS -->
<style>
/* Variables locales (equivalente al :root de tu demo) */
.slide2 {
  --w: clamp(130px, 14vw, 180px); /* ancho tarjeta */
  --h: calc(var(--w) * 16 / 9); /* 9:16 vertical → altura = 1.777 * ancho */
  --speed-s: 18;
  --third-x: calc(100vw / 3); /* tercio izquierdo */
  --center-y: 54vh; /* un poco más abajo */
}

/* Fondo del slide si quieres: .panel.s2 { background:#22c55e; } */

/* Layout del carrusel (igual que el original) */
.s2-frame {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
.s2-anchor {
  position: absolute;
  left: var(--third-x);
  top: var(--center-y);
  transform: translate(-50%, -50%);
  display: grid;
  row-gap: 14px;
  justify-items: center;
  scale: 0.9;
  transform-origin: center;
}
.s2-stage {
  width: min(92vw, 900px);
  height: min(60vh, 460px);
  perspective: 1200px;
  position: relative;
}
.s2-rig {
  position: absolute;
  inset: 0;
  transform-style: preserve-3d;
  will-change: transform;
}
.s2-carousel {
  position: absolute;
  inset: 0;
  transform-style: preserve-3d;
  will-change: transform;
}

/* Tarjetas (renombradas a .s2-card para no chocar con tu .panel de slides) */
.s2-card {
  position: absolute;
  top: 50%;
  left: 50%;
  width: var(--w);
  height: var(--h);
  border-radius: 14px;
  overflow: hidden;
  background: #fafafa;
  --angle: 0deg;
  --radius: 0px;
  /* una sola transform (Safari friendly) */
  transform: translate(-50%, -50%) rotateY(var(--angle)) translateZ(var(--radius));
  filter: drop-shadow(18px 24px 24px rgba(0, 0, 0, 0.3));
  transition:
    transform 0.45s cubic-bezier(0.2, 0.7, 0.3, 1),
    z-index 0.2s step-end,
    filter 0.2s ease;
}
.s2-card img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  backface-visibility: hidden;
}

.s2-card-back {
  position: absolute;
  top: 50%;
  left: 50%;
  width: var(--w);
  height: var(--h);
  border-radius: 14px;
  background: #fff;
  --angle: 0deg;
  --radius: 0px;
  transform: translate(-50%, -50%) rotateY(calc(var(--angle) + 180deg))
    translateZ(calc(var(--radius) - 0.5px));
}

/* Frontal (igual que tu demo) */
/* .s2-card.is-front {
  transform: translate(-50%, -50%) rotateY(var(--angle)) translateZ(var(--radius)) scale(1.5);
  z-index: 20;
  filter: drop-shadow(26px 36px 38px rgba(0, 0, 0, 0.38));
} */

/* DESPUÉS (pop sin escalar) */
.s2-card.is-front {
  transform: translate(-50%, -50%) rotateY(var(--angle)) /* acércala +120px (ajusta a gusto) */
    translateZ(calc(var(--radius) + 120px));
  z-index: 20;
  filter: drop-shadow(26px 36px 38px rgba(0, 0, 0, 0.38));
}

.s2-card img {
  /* ya tienes: width/height 100%, object-fit */
  image-rendering: -webkit-optimize-contrast; /* WebKit mejora nitidez */
  backface-visibility: hidden; /* evita artefactos en 3D */
  transform: translateZ(0); /* fuerza capa GPU (a veces ayuda) */
}

/* Controles glass */
.s2-controls {
  display: flex;
  gap: 0.6rem;
  align-items: center;
  justify-content: center;
  position: relative;
  z-index: 60;
}
.s2-btn {
  background: rgba(255, 255, 255, 0.35);
  border: 1px solid rgba(0, 0, 0, 0.12);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  color: #111;
  padding: 0.55rem 1rem;
  border-radius: 999px;
  cursor: pointer;
}
.s2-btn:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* Fondo blanco del slide 2 */
.panel.slide2 {
  background: #fff;
}

/* 1) Permite que el carrusel saque sombras fuera del slide */
.h-track > .panel.slide2,
.panel.slide2,
.s2-frame,
.s2-anchor,
.s2-stage {
  overflow: visible !important; /* <— clave */
}

/* 2) Un poco de “aire” abajo del slide para que no toque el borde del viewport */
.panel.slide2 {
  padding-bottom: 8vh;
}

/* 3) (Opcional) baja ligeramente la escala del front para evitar roces */
.s2-card.is-front {
  transform: translate(-50%, -50%) rotateY(var(--angle)) translateZ(var(--radius)) scale(1.38);
}

/* ===== Layout 2 columnas ===== */
.s2-grid {
  width: min(1200px, 92vw);
  margin: 0 auto;
  height: 100%;
  display: grid;
  grid-template-columns: 1fr 1fr; /* carrusel | texto */
  align-items: center;
  gap: clamp(20px, 4vw, 60px);
  overflow: visible;
}

/* Izquierda: mantenemos tu anclaje/posición del carrusel */
.s2-left {
  position: relative;
  overflow: visible;
}
.s2-frame,
.s2-anchor,
.s2-stage {
  overflow: visible;
} /* para que flote la sombra */

/* Derecha: bloque de texto */
.s2-right {
  display: grid;
  align-items: center;
}
.s2-copy {
  max-width: 520px;
  color: #0f172a; /* gris oscuro legible sobre fondo blanco */
}
.s2-copy .title {
  margin: 0 0 0.5rem 0;
  font-size: clamp(22px, 3.6vw, 34px);
  line-height: 1.15;
  color: #dc2626; /* rojo como en tu mock; cámbialo si quieres */
}
.s2-copy .subtitle {
  margin: 0 0 0.5rem 0;
  font-size: clamp(14px, 1.6vw, 16px);
  opacity: 0.85;
}
.s2-copy .body {
  margin: 0;
  font-size: clamp(13px, 1.4vw, 15px);
  line-height: 1.55;
  opacity: 0.9;
}

/* Responsive: apila en móvil */
@media (max-width: 860px) {
  .s2-grid {
    grid-template-columns: 1fr;
    align-content: center;
    justify-items: center;
    gap: 28px;
  }
  .s2-right {
    justify-items: center;
    text-align: center;
  }
  .s2-copy {
    max-width: 92vw;
  }
  /* el carrusel un pelín más arriba para que no corte abajo */
  .slide2 {
    --center-y: 52vh;
  }
}

/* Si sigue rozando el borde inferior en alguna resolución, sube aquí: */
.slide2 {
  --center-y: 54vh;
} /* ⬅️ ESTA es la línea que ajusta la altura del carrusel */

/* === MODO 2 COLUMNAS: centra el carrusel con grid (no absolute) === */
.s2-grid {
  width: min(1200px, 92vw);
  margin: 0 auto;
  height: 100%;
  display: grid;
  grid-template-columns: 1fr 1fr; /* carrusel | texto */
  align-items: center; /* ⬅ centra verticalmente ambas columnas */
  gap: clamp(20px, 4vw, 60px);
}

/* La columna izquierda actúa de “escenario”; centramos su contenido */
.s2-left {
  display: grid;
  place-items: center;
}

/* ⚠️ El ancla deja de ser absoluta en modo grid */
.s2-anchor {
  position: static; /* ⬅ antes absolute */
  left: auto;
  top: auto;
  transform: none;
  /* controles finos */
  --nudge-x: 0vw; /* 👈 mueve en X (negativo = más a la izquierda) */
  --nudge-y: 0vh; /* ya lo usábamos para subir/bajar */
  translate: var(--nudge-x) var(--nudge-y);
  overflow: visible;
  /* micro-ajuste vertical opcional */
  --nudge-y: 0vh; /* cambia a -2vh, -4vh si quieres subirlo un poco */
  translate: 0 var(--nudge-y);
}

/* El stage no recorta y flota correctamente */
.s2-frame,
.s2-stage {
  overflow: visible;
}

/* Dale algo de respiro al borde inferior del slide */
.panel.slide2 {
  padding-bottom: 6vh;
}

/* COPY a la derecha */
.s2-right {
  display: grid;
  justify-items: start;
}
.s2-copy {
  max-width: 520px;
}

/* Responsive: apila en móvil y recentra todo */
@media (max-width: 860px) {
  .s2-grid {
    grid-template-columns: 1fr;
    gap: 28px;
  }
  .s2-right {
    justify-items: center;
    text-align: center;
  }
  .s2-copy {
    max-width: 92vw;
  }
  .s2-anchor {
    --nudge-y: -2vh;
  } /* súbelo un pelín en pantallas pequeñas */
}

/* === RESPONSIVE: tamaño de tarjeta y posición === */

@media (max-width: 1200px) {
  .slide2 {
    --w: clamp(140px, 18vw, 200px);
    --center-y: 53vh;
  }
}
@media (max-width: 900px) {
  .slide2 {
    --w: clamp(130px, 22vw, 180px);
    --center-y: 51vh;
  }
}
@media (max-width: 640px) {
  .slide2 {
    --w: clamp(120px, 28vw, 160px);
    --center-y: 49vh;
  }
}

/* Alturas bajas de pantalla (laptops pequeñas, móviles apaisado) */
@media (max-height: 740px) {
  .slide2 {
    --center-y: 48vh;
  }
}
@media (max-height: 640px) {
  .slide2 {
    --center-y: 46vh;
  }
}

/* === RESPONSIVE: reduce el “pop” de la tarjeta frontal para que no se corte === */
@media (max-width: 900px), (max-height: 740px) {
  .s2-card.is-front {
    transform: translate(-50%, -50%) rotateY(var(--angle)) translateZ(var(--radius)) scale(1.32);
  }
}
@media (max-width: 640px), (max-height: 640px) {
  .s2-card.is-front {
    transform: translate(-50%, -50%) rotateY(var(--angle)) translateZ(var(--radius)) scale(1.22);
  }
}

/* opcional: si en algún breakpoint notas clipping, asegura que el stage no recorte */
.s2-stage {
  overflow: visible;
}
</style>
