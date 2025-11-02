<script setup lang="ts">
import { ref, nextTick, onMounted, onBeforeUnmount } from 'vue'

const modalOpen = ref(false)
const modalTitle = ref('Detalle')
const modalPdf = ref('/resume.pdf')
const closeBtn = ref<HTMLButtonElement | null>(null)

function openModal(e: MouseEvent) {
  const el = e.currentTarget as HTMLElement | null
  modalTitle.value = el?.getAttribute('data-modal-title') || 'Detalle'
  const pdfAttr = el?.getAttribute('data-modal-pdf')
  modalPdf.value = pdfAttr && pdfAttr.trim() !== '' ? pdfAttr : '/resume.pdf'

  modalOpen.value = true
  // seguimos bloqueando el body para que no se mueva el fondo
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

/* hover logic para apagar "DIGITAL" */
const cardsActive = ref(false)
let hoverCount = 0
let touchTimer: number | null = null

function onCardEnter() {
  hoverCount++
  cardsActive.value = true
}
function onCardLeave() {
  hoverCount = Math.max(0, hoverCount - 1)
  if (!hoverCount) cardsActive.value = false
}
function onCardTouch() {
  cardsActive.value = true
  if (touchTimer) clearTimeout(touchTimer)
  touchTimer = window.setTimeout(() => (cardsActive.value = false), 1200)
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
    <div class="stage" :class="{ 'cards-active': cardsActive }">
      <div class="deck">
        <!-- A -->
        <article
          class="card"
          @mouseenter="onCardEnter"
          @mouseleave="onCardLeave"
          @touchstart.passive="onCardTouch"
        >
          <div class="card__tab">
            <img
              src="https://images.unsplash.com/photo-1595152772835-219674b2a8a6?auto=format&fit=crop&w=960&h=360&q=80&fm=png"
              alt="Modelo A"
            />
          </div>
          <div class="card__body">
            <h3 class="card__title">Editorial A</h3>
            <p class="card__p">Campaña retrato – enfoque en composición y luz natural.</p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Editorial A"
                data-modal-pdf="/resume.pdf"
                @click="openModal"
              >
                Ver más
              </button>
            </div>
          </div>
        </article>

        <!-- B -->
        <article
          class="card"
          @mouseenter="onCardEnter"
          @mouseleave="onCardLeave"
          @touchstart.passive="onCardTouch"
        >
          <div class="card__tab card__tab--img">
            <img src="/image/Frame 1427.png" alt="Modelo B" />
          </div>
          <div class="card__body">
            <h3 class="card__title">Editorial B</h3>
            <p class="card__p">Producción de moda con contraste de color y textura.</p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Editorial B"
                data-modal-pdf="/pdf/editorial-b.pdf"
                @click="openModal"
              >
                Ver más
              </button>
            </div>
          </div>
        </article>

        <!-- C -->
        <article
          class="card"
          @mouseenter="onCardEnter"
          @mouseleave="onCardLeave"
          @touchstart.passive="onCardTouch"
        >
          <div class="card__tab">
            <img
              src="https://images.unsplash.com/photo-1503341455253-b2e723bb3dbb?auto=format&fit=crop&w=960&h=360&q=80&fm=png"
              alt="Modelo C"
            />
          </div>
          <div class="card__body">
            <h3 class="card__title">Editorial C</h3>
            <p class="card__p">Fotografía artística de retrato – luz dura y fondo oscuro.</p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Editorial C"
                data-modal-pdf="/resume.pdf"
                @click="openModal"
              >
                Ver más
              </button>
            </div>
          </div>
        </article>
      </div>
    </div>

    <!-- Modal PDF -->
    <teleport to="body">
      <div v-if="modalOpen" class="backdrop" @click.self="closeModal">
        <div class="modal" @click.stop>
          <button class="close" ref="closeBtn" type="button" @click="closeModal" aria-label="Close">
            ×
          </button>
          <!-- aquí queremos scroll vertical -->
          <iframe class="doc" :src="modalPdf" :title="modalTitle"></iframe>
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

.panel.s3 .stage {
  position: relative;
  height: 100vh;
  width: 100%;
  background: #fff;
  overflow: hidden;
  padding: clamp(8px, 2vw, 24px);
  color: var(--txt);
}
.panel.s3 .stage::after {
  content: 'DIGITAL';
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  font-weight: 800;
  font-size: clamp(56px, 18vw, 220px);
  letter-spacing: 0.06em;
  color: #0f172a;
  opacity: 0.06;
  transition:
    opacity 0.34s ease,
    filter 0.34s ease;
  pointer-events: none;
  z-index: 0;
}
.panel.s3 .stage.cards-active::after {
  opacity: 0;
  filter: blur(10px);
}

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
  z-index: 1;
}

.panel.s3 .card {
  padding: 0 !important;
  background: #fff !important;
  border: 0 !important;
  box-shadow: 18px 24px 24px rgba(0, 0, 0, 0.08) !important;
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

/* header-img con misma altura en todas */
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
  object-position: center center;
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
  color: #0a66c2;
  text-decoration: underline;
  font-weight: 600;
  background: none;
  border: 0;
  cursor: pointer;
}

/* ===== Modal (igual al de home pero sin botón de descarga) ===== */
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
  overflow: auto; /* 👈 aquí activamos scroll dentro */
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
