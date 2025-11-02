<template>
  <section class="panel bento-full">
    <div class="grid">
      <!-- 1. título -->
      <div class="cell title">
        <div class="title-inner">
          <span class="line">PRO</span>
          <span class="line">JECTS</span>
          <span class="num">25</span>
        </div>
      </div>

      <!-- 2. imagen 1 -->
      <div class="cell img" @mousemove="onSlotMove" @mouseleave="onSlotLeave">
        <button class="img-btn" type="button" @click="openImageModal(0)">
          <img :src="images[0].src" :alt="images[0].alt" />
        </button>
      </div>

      <!-- 3. imagen 2 (la de arriba a la derecha, la que queremos TAPAR) -->
      <div class="cell img second" @mousemove="onSlotMove" @mouseleave="onSlotLeave">
        <button class="img-btn" type="button" @click="openImageModal(1)">
          <img :src="images[1].src" :alt="images[1].alt" />
        </button>
      </div>

      <!-- 4. video -->
      <div class="cell video">
        <video
          class="video-el"
          src="https://videos.pexels.com/video-files/4828611/4828611-hd_1920_1080_30fps.mp4"
          autoplay
          muted
          loop
          playsinline
        ></video>
        <div class="video-label">Showreel</div>
      </div>

      <!-- 5. TEXTO que dispara el overlay -->
      <div class="cell text" @mouseenter="showFlyout = true" @mouseleave="showFlyout = false">
        <div class="text-content">
          <h3>Dirección de arte</h3>
          <p>Retrato · moda · beauty. Control de luz, composición y post.</p>
          <p class="hint">Hover para ver más</p>
        </div>

        <!-- overlay ABSOLUTO sobre el GRID -->
        <div class="text-flyout" :class="{ 'is-open': showFlyout }">
          <h2>Dirección de arte</h2>
          <p>
            Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed id bibendum orci. Donec
            vitae leo id odio consectetur porta. Cras euismod justo non mi commodo, nec lacinia
            risus molestie.
          </p>
          <p>
            Phasellus sollicitudin lorem vitae velit efficitur, vel cursus ipsum maximus. Vivamus id
            nisi ultrices, posuere enim non, tincidunt magna. Nullam vitae feugiat tortor.
          </p>
          <p>
            Donec fringilla sem vitae turpis malesuada, nec consectetur sapien bibendum. Integer
            viverra, mi eu varius gravida, justo dolor tempor lectus.
          </p>
        </div>
      </div>
    </div>

    <!-- MODAL IMG -->
    <teleport to="body">
      <div v-if="isImageModalOpen && activeImage" class="img-modal" @click.self="closeImageModal">
        <div class="img-box">
          <button class="close" @click="closeImageModal" aria-label="Cerrar">×</button>
          <img :src="activeImage.src" :alt="activeImage.alt" />
        </div>
      </div>
    </teleport>
  </section>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const images = ref([
  {
    src: 'https://images.unsplash.com/photo-1516592673392-7fc07e52bff7?w=1200&auto=format&fit=crop&q=80',
    alt: 'Retrato editorial',
  },
  {
    src: 'https://images.unsplash.com/photo-1513364776144-60967b0f800f?w=1200&auto=format&fit=crop&q=80',
    alt: 'Beauty studio',
  },
])

/* modal imagen */
const isImageModalOpen = ref(false)
const activeIndex = ref<number | null>(null)
const activeImage = computed(() =>
  activeIndex.value == null ? null : images.value[activeIndex.value],
)
function openImageModal(i: number) {
  activeIndex.value = i
  isImageModalOpen.value = true
  document.documentElement.classList.add('no-scroll')
}
function closeImageModal() {
  isImageModalOpen.value = false
  activeIndex.value = null
  document.documentElement.classList.remove('no-scroll')
}

/* flyout de texto (solo hover) */
const showFlyout = ref(false)

/* ESC solo para el modal de imagen */
function onKey(e: KeyboardEvent) {
  if (e.key === 'Escape' && isImageModalOpen.value) {
    closeImageModal()
  }
  if (isImageModalOpen.value && activeIndex.value != null) {
    if (e.key === 'ArrowRight') activeIndex.value = (activeIndex.value + 1) % images.value.length
    if (e.key === 'ArrowLeft')
      activeIndex.value = (activeIndex.value - 1 + images.value.length) % images.value.length
  }
}
onMounted(() => window.addEventListener('keydown', onKey))
onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKey)
  document.documentElement.classList.remove('no-scroll')
})

/* parallax mouse */
function onSlotMove(e: MouseEvent) {
  const box = e.currentTarget as HTMLElement
  const img = box.querySelector('img') as HTMLElement | null
  if (!img) return
  const r = box.getBoundingClientRect()
  const nx = (e.clientX - r.left) / r.width - 0.5
  const ny = (e.clientY - r.top) / r.height - 0.5
  img.style.transform = `translate(${nx * 14}px, ${ny * 14}px) scale(1.4)`
}
function onSlotLeave(e: MouseEvent) {
  const box = e.currentTarget as HTMLElement
  const img = box.querySelector('img') as HTMLElement | null
  if (img) img.style.transform = ''
}
</script>

<style scoped>
.bento-full {
  width: 100vw;
  height: 100vh;
  background: #fff;
  display: flex;
  margin: 0;
  padding: 0;
}
.grid {
  position: relative; /* para anclar el overlay dentro del grid */
  overflow: visible; /* dejar salir el overlay hacia arriba */
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(2, 1fr);
  width: 100%;
  height: 100%;
}
.cell {
  position: relative;
  /* OJO: antes teníamos overflow: hidden; eso estaba cortando el overlay */
  overflow: visible;
  background: #fff;
}

/* título */
.title {
  background: #e5e5e5;
  display: flex;
  align-items: center;
  justify-content: center;
}
.title-inner {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}
.line {
  font-weight: 900;
  font-size: clamp(56px, 7vw, 100px);
  line-height: 0.9;
  color: #000;
}
.num {
  font-weight: 800;
  font-size: clamp(26px, 2.5vw, 40px);
  margin-top: 6px;
  color: #000;
}

/* imágenes */
.img {
  background: #ddd5d5;
  overflow: hidden; /* solo las imágenes sí recortan */
}
.img.second {
  background: #d3a9a9;
}
.img-btn {
  width: 100%;
  height: 100%;
  border: 0;
  background: transparent;
  cursor: zoom-in;
  display: block;
  overflow: hidden;
}
.img-btn img {
  width: 115%;
  height: 115%;
  object-fit: cover;
  transform: scale(1.08);
  transition: transform 0.35s ease-out;
}
.img-btn:hover img,
.img-btn:focus-visible img {
  transform: scale(1.5);
}

/* video */
.video {
  grid-column: 1 / 3;
  grid-row: 2 / 3;
  background: #6bcc94;
  overflow: hidden;
}
.video-el {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}
.video-label {
  position: absolute;
  top: 16px;
  left: 16px;
  background: rgba(255, 255, 255, 0.85);
  padding: 4px 10px;
  border-radius: 999px;
  font-size: 0.7rem;
  font-weight: 600;
}

/* ===== TEXTO ===== */
.text {
  grid-column: 3 / 4;
  grid-row: 2 / 3;
  background: #fff;
  padding: clamp(20px, 3vw, 36px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  cursor: pointer;
  /* muestro que puede superponerse */
  z-index: 1;
}
.text-content h3 {
  margin: 0 0 6px;
  font-size: clamp(20px, 2.4vw, 32px);
}
.text-content p {
  margin: 0;
  font-size: clamp(13px, 1.4vw, 16px);
  color: rgba(0, 0, 0, 0.6);
}
.hint {
  font-size: 0.7rem;
  margin-top: 10px;
  opacity: 0.45;
}

/* overlay que tapa la imagen de arriba derecha */
.text-flyout {
  position: absolute;
  left: 0;
  top: -100%; /* empieza justo sobre la imagen */
  width: 100%;
  height: 200%; /* tapa imagen (arriba) + texto (abajo) */
  background: #fff;
  padding: clamp(20px, 2.6vw, 32px);
  display: flex;
  flex-direction: column;
  gap: 10px;
  opacity: 0;
  pointer-events: none;
  transform: translateY(10px);
  transition:
    opacity 0.22s ease,
    transform 0.22s ease;
  box-shadow: 0 26px 50px rgba(0, 0, 0, 0.12);
  z-index: 5; /* por encima de la imagen de arriba */
}
.text-flyout.is-open {
  opacity: 1;
  pointer-events: auto;
  transform: translateY(0);
}

/* modal img */
.img-modal {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.7);
  display: grid;
  place-items: center;
  z-index: 1000;
}
.img-box {
  position: relative;
  max-width: 85vw;
  max-height: 85vh;
  background: #fff;
  border-radius: 10px;
  overflow: hidden;
}
.img-box img {
  display: block;
  max-width: 100%;
  max-height: 85vh;
  object-fit: contain;
  background: #fff;
}
.close {
  position: absolute;
  top: 6px;
  right: 10px;
  width: 34px;
  height: 34px;
  border: 0;
  background: rgba(255, 255, 255, 0.9);
  border-radius: 50%;
  font-size: 22px;
  cursor: pointer;
}

/* responsive */
@media (max-width: 900px) {
  .grid {
    grid-template-columns: 1fr;
    grid-template-rows: repeat(5, minmax(160px, 1fr));
  }
  .text-flyout {
    top: 0;
    height: 100%;
  }
}
</style>
