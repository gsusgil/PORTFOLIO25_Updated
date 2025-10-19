<script setup lang="ts">
import { ref, nextTick, onMounted, onBeforeUnmount } from 'vue'

const modalOpen = ref(false)
const modalTitle = ref('Título')
const modalBody = ref('Texto.')
const closeBtn = ref<HTMLButtonElement | null>(null)

function openModal(e: MouseEvent) {
  const el = e.currentTarget as HTMLElement | null
  modalTitle.value = el?.getAttribute('data-modal-title') || 'Detalle'
  modalBody.value = el?.getAttribute('data-modal-body') || 'Contenido no disponible.'
  modalOpen.value = true
  nextTick(() => closeBtn.value?.focus())
}
function closeModal() {
  modalOpen.value = false
}
function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape' && modalOpen.value) closeModal()
}

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
          <div class="card__tab">Resumen A</div>
          <div class="card__body">
            <h3 class="card__title">Título A</h3>
            <p class="card__p">Contenido de la card A. Aparece completo al subir.</p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Detalle A"
                data-modal-body="Este es el detalle A (h1 + p)."
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
          <div class="card__tab">Resumen B</div>
          <div class="card__body">
            <h3 class="card__title">Título B</h3>
            <p class="card__p">Contenido de la card B. Aparece completo al subir.</p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Detalle B"
                data-modal-body="Este es el detalle B (h1 + p)."
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
          <div class="card__tab">Resumen C</div>
          <div class="card__body">
            <h3 class="card__title">Título C</h3>
            <p class="card__p">Contenido de la card C. Aparece completo al subir.</p>
            <div class="card__actions">
              <button
                class="link"
                type="button"
                data-modal-title="Detalle C"
                data-modal-body="Este es el detalle C (h1 + p)."
                @click="openModal"
              >
                Ver más
              </button>
            </div>
          </div>
        </article>
      </div>
    </div>

    <!-- Modal -->
    <teleport to="body">
      <div class="modal-backdrop" :class="{ show: modalOpen }" @click.self="closeModal">
        <div class="modal" role="dialog" aria-modal="true" aria-labelledby="modal-title">
          <h1 id="modal-title">{{ modalTitle }}</h1>
          <p id="modal-body">{{ modalBody }}</p>
          <button class="close" ref="closeBtn" type="button" @click="closeModal">Cerrar</button>
        </div>
      </div>
    </teleport>
  </section>
</template>

<style>
/* ===== solo Slide3 ===== */
.panel.s3 {
  --tab-h: 64px;
  --radius: 14px;
  --w: 360px;
  --tab: #6b1f1c;
  --txt: #0f172a;
  --muted: rgba(15, 23, 42, 0.78);
}

/* Lienzo */
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

/* Carril */
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

/* ===== CARD (override total) ===== */
.panel.s3 .card {
  /* anular herencias globales peligrosas */
  padding: 0 !important;
  background: #fff !important;
  backdrop-filter: none !important;
  -webkit-backdrop-filter: none !important;
  border: 0 !important;
  box-shadow: 18px 24px 24px rgba(0, 0, 0, 0.08) !important;

  width: var(--w);
  height: min(75vh, 620px);
  overflow: hidden;
  pointer-events: auto;
  position: relative;

  border-top-left-radius: var(--radius);
  border-top-right-radius: var(--radius);

  /* estado inicial: SOLO pestaña visible */
  transform: translateY(calc(100% - var(--tab-h))) !important;
  transition: transform 420ms cubic-bezier(0.22, 1, 0.36, 1);
}
.panel.s3 .card:hover {
  /* levantar al hover (forzado) */
  transform: translateY(0) !important;
}
.panel.s3 .card::before {
  display: none !important;
} /* por si hay overlays globales */

/* Pestaña */
.panel.s3 .card__tab {
  height: var(--tab-h);
  background: var(--tab);
  color: #fff;
  border-top-left-radius: var(--radius);
  border-top-right-radius: var(--radius);
  display: flex;
  align-items: center;
  padding: 0 18px;
  font-weight: 700;
  letter-spacing: 0.2px;
  user-select: none;
  cursor: pointer;
  position: relative;
  z-index: 2;
  box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.15);
}

/* Cuerpo */
.panel.s3 .card__body {
  padding: 18px;
  display: grid;
  gap: 12px;
  position: relative;
  z-index: 1;
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

/* Modal */
.panel.s3 .modal-backdrop {
  position: fixed;
  inset: 0;
  display: none;
  align-items: center;
  justify-content: center;
  background: rgba(15, 23, 42, 0.6);
  padding: 24px;
  z-index: 1000;
}
.panel.s3 .modal-backdrop.show {
  display: flex;
}
.panel.s3 .modal {
  max-width: 680px;
  width: 100%;
  background: #fff;
  color: #0f172a;
  border-radius: 16px;
  box-shadow: 0 40px 90px rgba(0, 0, 0, 0.45);
  padding: 22px;
}
.panel.s3 .modal h1 {
  margin: 0 0 8px;
}
.panel.s3 .modal p {
  margin: 0;
  line-height: 1.7;
}
.panel.s3 .close {
  margin-top: 16px;
  display: inline-block;
  border: 0;
  background: rgba(255, 255, 255, 0.8);
  color: #0f172a;
  padding: 10px 14px;
  border-radius: 999px;
  font-weight: 700;
  cursor: pointer;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.06);
}

/* Responsive */
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
@media (prefers-reduced-motion: reduce) {
  .panel.s3 .card {
    transition: none;
  }
}
</style>
