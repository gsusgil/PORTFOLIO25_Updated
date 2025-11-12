<script setup lang="ts">
import { ref, nextTick, onMounted, onBeforeUnmount } from 'vue'

/* ===== Modal PDF (solo para la card C) ===== */
const modalOpen = ref(false)
const modalTitle = ref('Detalle')
const modalPdf = ref('/resume.pdf')
const closeBtn = ref<HTMLButtonElement | null>(null)

function openModal(e: MouseEvent) {
  const el = e.currentTarget as HTMLElement | null
  const title = el?.getAttribute('data-modal-title') || 'Detalle'
  const pdfAttr = (el?.getAttribute('data-modal-pdf') || '/resume.pdf').trim()
  const openBlank = el?.getAttribute('data-open-blank') === 'true'

  // A y B → abrir en pestaña nueva
  if (openBlank) {
    window.open(pdfAttr, '_blank', 'noopener')
    return
  }

  // C → modal
  modalTitle.value = title
  modalPdf.value = pdfAttr
  modalOpen.value = true
  document.documentElement.style.overflow = 'hidden'
  nextTick(() => closeBtn.value?.focus())
}

function closeModal() {
  modalOpen.value = false
  document.documentElement.style.overflow = ''
}

function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape' && modalOpen.value) closeModal()
}

/* ===== Fondo dinámico con la imagen de la card en hover ===== */
const stageEl = ref<HTMLDivElement | null>(null)
const cardsActive = ref(false)
let hoverCount = 0
let touchTimer: number | null = null

function setStageBgFromCard(card?: HTMLElement | null) {
  const url = card?.querySelector('img')?.getAttribute('src') || ''
  if (!stageEl.value) return
  if (url) {
    stageEl.value.style.setProperty('--bg-url', `url("${url}")`)
    stageEl.value.classList.add('has-bg')
  } else {
    stageEl.value.classList.remove('has-bg')
    stageEl.value.style.removeProperty('--bg-url')
  }
}

function onCardEnter(e?: MouseEvent) {
  hoverCount++
  cardsActive.value = true
  setStageBgFromCard(e?.currentTarget as HTMLElement | null)
}
function onCardLeave() {
  hoverCount = Math.max(0, hoverCount - 1)
  if (!hoverCount) {
    cardsActive.value = false
    setStageBgFromCard(null)
  }
}
function onCardTouch(e: TouchEvent) {
  cardsActive.value = true
  setStageBgFromCard(e.currentTarget as HTMLElement | null)
  if (touchTimer) clearTimeout(touchTimer)
  touchTimer = window.setTimeout(() => {
    cardsActive.value = false
    setStageBgFromCard(null)
  }, 1200)
}

onMounted(() => window.addEventListener('keydown', onKey))
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKey)
  if (touchTimer) clearTimeout(touchTimer)
  document.documentElement.style.overflow = ''
})
</script>

<template>
  <section id="slide-3" class="panel s3" style="padding: 0">
    <div class="stage" ref="stageEl" :class="{ 'cards-active': cardsActive }">
      <div class="deck">
        <!-- A (blank) -->
        <article
          class="card"
          @mouseenter="onCardEnter($event)"
          @mouseleave="onCardLeave"
          @touchstart.passive="onCardTouch"
        >
          <div class="card__tab card__tab--img">
            <img src="/image/revista_mosaic.png" alt="Mosaic cover" />
          </div>

          <div class="card__body">
            <h3 class="card__title">Design and Art Direction for MOSAIC</h3>
            <p class="card__p">
              ZIGURAT’s magazine focused on artificial intelligence applied to architecture.
            </p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="MOSAIC Magazine"
                data-modal-pdf="/documents/mosaic_mail_web_v2.pdf"
                data-open-blank="true"
                @click="openModal"
              >
                more here
              </button>
            </div>
          </div>
        </article>

        <!-- B (blank) -->
        <article
          class="card"
          @mouseenter="onCardEnter($event)"
          @mouseleave="onCardLeave"
          @touchstart.passive="onCardTouch"
        >
          <div class="card__tab card__tab--img">
            <img src="/image/mails%26%26websitev2_img.png" alt="Mails & website" />
          </div>
          <div class="card__body">
            <h3 class="card__title">Email Campaigns & Landing Page Design</h3>
            <p class="card__p">
              Coordinated email and landing page design that ensures brand consistency and a
              seamless user journey.
            </p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Editorial B"
                data-modal-pdf="/documents/modal_mail_web_v2.pdf"
                data-open-blank="true"
                @click="openModal"
              >
                more here
              </button>
            </div>
          </div>
        </article>

        <!-- C (modal interno) -->
        <article
          class="card"
          @mouseenter="onCardEnter($event)"
          @mouseleave="onCardLeave"
          @touchstart.passive="onCardTouch"
        >
          <div class="card__tab">
            <img src="/image/paula_img.png" alt="Brand Manual" />
          </div>
          <div class="card__body">
            <h3 class="card__title">Brand Identity Guidelines</h3>
            <p class="card__p">
              Comprehensive brand manual detailing the visual identity, logo applications, and usage
              guidelines for consistent communication.
            </p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Brand Identity Guidelines"
                data-modal-pdf="/documents/manual-Paula-Belil.pdf"
                @click="openModal"
              >
                more here
              </button>
            </div>
          </div>
        </article>
      </div>
    </div>

    <!-- Modal PDF (solo C) -->
    <teleport to="body">
      <div v-if="modalOpen" class="backdrop" @click.self="closeModal">
        <div class="modal" @click.stop>
          <button class="close" ref="closeBtn" type="button" @click="closeModal" aria-label="Close">
            ×
          </button>
          <iframe
            class="doc"
            :src="`${modalPdf}#toolbar=0&navpanes=0&scrollbar=0`"
            :title="modalTitle"
          ></iframe>
        </div>
      </div>
    </teleport>
  </section>
</template>

<style>
.panel.s3 {
  --radius: 14px;
  --w: 360px;
  --txt: #0f172a;
  --muted: rgba(15, 23, 42, 0.78);
  --peek-h: 240px;
}

/* ===== Stage con fondo dinámico (blur 50% opacidad) ===== */
.panel.s3 .stage {
  position: relative;
  height: 100vh;
  width: 100%;
  background: #fff;
  overflow: hidden;
  padding: clamp(8px, 2vw, 24px);
  color: var(--txt);
  --bg-url: ''; /* la setea JS al hover */
}
.panel.s3 .stage::before {
  content: '';
  position: absolute;
  inset: 0;
  z-index: 0; /* debajo de texto y cards */
  background: var(--bg-url) center / cover no-repeat;
  filter: blur(14px);
  opacity: 0;
  transition:
    opacity 300ms ease,
    filter 300ms ease;
  transform: translateZ(0);
}
.panel.s3 .stage.has-bg::before {
  opacity: 0.5; /* 50% */
}

/* Texto grande del fondo */
.panel.s3 .stage::after {
  content: 'GRAPHIC DESIGN';
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  font-weight: 800;
  font-size: clamp(56px, 18vw, 150px);
  letter-spacing: 0.06em;
  color: #0f172a;
  opacity: 0.06;
  transition:
    opacity 0.34s ease,
    filter 0.34s ease;
  pointer-events: none;
  z-index: 1; /* sobre el fondo blur, bajo las cards */
}
.panel.s3 .stage.cards-active::after {
  opacity: 0;
  filter: blur(10px);
}

/* Las cards por encima de todo */
.panel.s3 .deck {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  gap: 40px;
  padding: 0 24px 8px;
  pointer-events: none;
  z-index: 2;
}

.panel.s3 .card {
  padding: 0 !important;
  background: #fff !important;
  border: 0 !important;
  box-shadow: -20px 24px 24px rgba(0, 0, 0, 0.08) !important;
  width: var(--w);
  height: min(65vh, 520px);
  overflow: hidden;
  pointer-events: auto;
  position: relative;
  border-top-left-radius: var(--radius);
  border-top-right-radius: var(--radius);
  transform: translateY(calc(100% - var(--peek-h))) !important;
  transition: transform 420ms cubic-bezier(0.22, 1, 0.36, 1);
}
.panel.s3 .card:hover {
  transform: translateY(0) !important;
}

/* header-img con misma altura */
.panel.s3 .card__tab {
  height: var(--peek-h);
  border-top-left-radius: var(--radius);
  border-top-right-radius: var(--radius);
  overflow: hidden;
  display: block;
  line-height: 0;
  background: #eee;
}
.panel.s3 .card__tab--img {
  background: transparent;
}
.panel.s3 .card__tab img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  object-position: center;
  display: block;
  transition: transform 0.35s ease;
}
.panel.s3 .card:hover .card__tab img {
  transform: scale(1.05);
}

/* cuerpo */
.panel.s3 .card__body {
  padding: 18px;
  display: grid;
  gap: 12px;
  background: #fff;
  min-height: 180px;
}
.panel.s3 .card__title {
  margin: 0;
  font-size: clamp(18px, 2.8vw, 22px);
  color: var(--txt);
}
.panel.s3 .card__p {
  margin: 0;
  line-height: 1.65;
  color: var(--muted);
}
.panel.s3 .card__actions {
  margin-top: auto;
}
.panel.s3 .link {
  color: #0f172a;
  text-decoration: underline;
  font-weight: 600;
  background: none;
  border: 0;
  cursor: pointer;
}

/* Modal */
.backdrop {
  position: fixed;
  inset: 0;
  z-index: 2000;
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
  overflow: auto;
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

/* responsive */
@media (max-width: 1100px) {
  .panel.s3 {
    --w: 320px;
  }
  .panel.s3 .deck {
    gap: 18px;
  }
}
@media (max-width: 980px) {
  .panel.s3 {
    --w: 300px;
  }
  .panel.s3 .deck {
    gap: 12px;
  }
}
@media (max-width: 920px) {
  .panel.s3 {
    --w: 280px;
  }
  .panel.s3 .deck {
    justify-content: space-evenly;
  }
}
</style>
