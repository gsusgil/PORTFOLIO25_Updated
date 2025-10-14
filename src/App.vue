<template>
  <RouterView />
</template>

<script setup lang="ts"></script>

<style>
html, body, #app { height: 100%; margin: 0; }
:root {
  --bg: #0b0f14; --fg: #e5e7eb; --muted: #94a3b8; --accent: #38bdf8;
}
body { background: var(--bg); color: var(--fg); font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial; }
* { box-sizing: border-box; }
a { color: inherit; text-decoration: none; }
button { cursor: pointer; }
html {
  scroll-behavior: auto;   /* evita interferencias */
  overflow-y: scroll;      /* 👈 scrollbar siempre visible */
}

html, body {
  overscroll-behavior: none;   /* sin “rebote” al llegar a top/bottom */
}
</style>
