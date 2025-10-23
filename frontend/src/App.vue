<template>
  <div class="page">
    <!-- wobbling white sheet sized to viewport -->
    <div class="tiltwrap">
      <header class="header">
        <div class="brand" @click="open('https://github.com/TSherpa10/my-website')">
          my portfolio
        </div>
      </header>

      <main class="main">
        <section class="hero">
          <figure class="avatar" aria-label="profile image placeholder">
            <img :src="imageSrc" alt="tashi sherpa portrait placeholder" />
          </figure>

          <div class="intro">
            <h1 class="name">Tashi Sherpa</h1>
            <p class="blurb">{{ hobbies }} = <span class="me-text">me</span></p>

            <div class="cta-row">
              <Button
                label="linkedin"
                icon="pi pi-linkedin"
                class="p-button-outlined"
                @click="open(links.linkedin)"
              />
              <Button
                label="github"
                icon="pi pi-github"
                class="p-button-outlined"
                @click="open(links.github)"
              />
              <Button
                label="game dev"
                icon="pi pi-sparkles"
                class="p-button-text"
                @click="open(links.gamedev)"
              />
            </div>
          </div>
        </section>

        <section class="section">
          <h2>software experience</h2>
          <ul class="xp" role="list">
            <li v-for="item in experience" :key="item.company" class="xp-item">
              <div class="xp-left">
                <div class="xp-company" @click="open(item.link)">{{ item.company }}</div>
                <div class="xp-meta">{{ item.period }}</div>
              </div>
              <div class="xp-text">{{ item.oneLine }}</div>
            </li>
          </ul>
        </section>
      </main>

      <footer class="footer">
        <nav class="foot-links" aria-label="social links">
          <a :href="links.linkedin" target="_blank" rel="noreferrer noopener">linkedin</a>
          <a :href="links.github" target="_blank" rel="noreferrer noopener">github</a>
          <a :href="links.gamedev" target="_blank" rel="noreferrer noopener">game dev</a>
        </nav>
        <div class="brand">© {{ year }} tashi sherpa</div>
      </footer>
    </div>

    <!-- black-blob cursor -->
    <div class="cursor-layer" aria-hidden="true">
      <div class="cursor-wrap">
        <div class="cursor-bubble"></div>
      </div>
    </div>
    <div class="laser-dot" aria-hidden="true"></div>
  </div>
</template>

<script lang="ts" setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import Button from 'primevue/button'

interface Links {
  linkedin?: string
  github?: string
  gamedev?: string
}
interface ExperienceItem {
  company: string
  period: string
  oneLine: string
  link?: string
}

const imageSrc = ref(
  'https://media.licdn.com/dms/image/v2/D4E03AQGfWDjhgOlNpg/profile-displayphoto-crop_800_800/B4EZoCjxODIMAI-/0/1760979522831?e=1762992000&v=beta&t=Ga7gq9y2CY1vt51js5CkFvNwbpijXUcwUWHlZkLHK64',
)
const hobbies = ref('sports + cooking + photography + nature + game dev')
const links = ref<Links>({
  linkedin: 'https://www.linkedin.com/in/tsherpa10/',
  github: 'https://github.com/TSherpa10',
  gamedev: 'https://notoshi.itch.io/',
})

const experience = ref<ExperienceItem[]>([
  {
    company: 'rokt',
    period: 'nov.2025 — present',
    oneLine: 'cdp platform team',
    link: 'https://www.rokt.com/',
  },
  {
    company: 'jhu applied physics lab',
    period: 'aug.2025 — oct.2025',
    oneLine: 'autonomous sytems — full-stack, devsecops',
    link: 'https://www.jhuapl.edu/',
  },
  {
    company: 'leidos',
    period: 'jun.2024 — nov.2024',
    oneLine: 'simulation software — internal tooling, a/b testing, e2e system design',
    link: 'https://www.leidos.com/',
  },
  {
    company: 'nytimes',
    period: 'jun.2023 — aug.2023',
    oneLine: 'live news team - full-stack, drove engagement to millions daily',
    link: 'https://www.nytimes.com/',
  },
])

function open(href?: string) {
  if (href && href !== '#') window.open(href, '_blank', 'noopener')
}
const year = computed(() => new Date().getFullYear())

/* ---------------- Cursor / tilt controller (hardened) ---------------- */
let aborter: AbortController | null = null
let rafId = 0

const state = {
  w: 0,
  h: 0,
  x: 0,
  y: 0, // pointer px
  nx: 0,
  ny: 0, // normalized -1..1
  visible: false,
  disabled: false,
}

function setVars(root: HTMLElement, x: number, y: number, nx: number, ny: number) {
  root.style.setProperty('--x', `${x}px`)
  root.style.setProperty('--y', `${y}px`)

  const maxTilt = 4
  const rx = (-ny * maxTilt).toFixed(3)
  const ry = (nx * maxTilt).toFixed(3)
  const tMax = 10
  const tx = (nx * tMax).toFixed(2)
  const ty = (ny * tMax).toFixed(2)

  root.style.setProperty('--rx', `${rx}deg`)
  root.style.setProperty('--ry', `${ry}deg`)
  root.style.setProperty('--tx', `${tx}px`)
  root.style.setProperty('--ty', `${ty}px`)

  const theta = (Math.atan2(ny, nx) * 180) / Math.PI
  root.style.setProperty('--theta', `${theta.toFixed(2)}deg`)
  root.style.setProperty('--sx', (1 + Math.abs(nx) * 0.25).toFixed(3))
  root.style.setProperty('--sy', (1 + Math.abs(ny) * 0.18).toFixed(3))

  // disable invert over avatar
  const hit = document.elementFromPoint(x, y) as HTMLElement | null
  document.documentElement.classList.toggle('over-avatar', !!hit && !!hit.closest('.avatar'))
}

function tick() {
  rafId = 0
  if (state.disabled) return
  setVars(document.documentElement, state.x, state.y, state.nx, state.ny)
}

function schedule() {
  if (!rafId) rafId = requestAnimationFrame(tick)
}

function updateFromPointer(clientX: number, clientY: number) {
  state.x = clientX
  state.y = clientY
  state.nx = (clientX / state.w) * 2 - 1
  state.ny = (clientY / state.h) * 2 - 1
  schedule()
}

function initCenter() {
  const cx = Math.round(state.w / 2)
  const cy = Math.round(state.h / 2)
  state.x = cx
  state.y = cy
  state.nx = 0
  state.ny = 0
  setVars(document.documentElement, cx, cy, 0, 0)
}

function applyBaseVars() {
  const root = document.documentElement
  root.style.setProperty('--x', '50vw')
  root.style.setProperty('--y', '50vh')
  root.style.setProperty('--rx', '0deg')
  root.style.setProperty('--ry', '0deg')
  root.style.setProperty('--tx', '0px')
  root.style.setProperty('--ty', '0px')
  root.style.setProperty('--d', '128px')
  root.style.setProperty('--theta', '0deg')
  root.style.setProperty('--sx', '1')
  root.style.setProperty('--sy', '1')
  root.style.setProperty('--invert-contrast', '1.35')
  root.style.setProperty('--invert-bright', '1.00')
  root.style.setProperty('--soft', '1.5px')
}

function setVisibility(show: boolean) {
  document.documentElement.classList.toggle('cursor-hidden', !show)
  state.visible = show
}

onMounted(() => {
  aborter = new AbortController()
  const { signal } = aborter

  const measure = () => {
    state.w = window.innerWidth
    state.h = window.innerHeight
  }
  measure()

  const mqlMotion = window.matchMedia('(prefers-reduced-motion: reduce)')
  const mqlTouch = window.matchMedia('(hover: none), (pointer: coarse)')
  const updateDisabled = () => {
    state.disabled = mqlMotion.matches || mqlTouch.matches
    document.documentElement.classList.toggle('no-bubble', state.disabled)
    if (state.disabled) {
      document.body.classList.remove('is-clicking')
      setVisibility(false)
      cancelAnimationFrame(rafId)
      rafId = 0
    } else {
      applyBaseVars()
      setVisibility(true)
      measure()
      initCenter()
    }
  }
  mqlMotion.addEventListener('change', updateDisabled, { signal })
  mqlTouch.addEventListener('change', updateDisabled, { signal })
  updateDisabled()

  applyBaseVars()
  initCenter()

  if (!state.disabled) {
    window.addEventListener('pointermove', (e) => updateFromPointer(e.clientX, e.clientY), {
      passive: true,
      signal,
    })
    window.addEventListener('pointerdown', () => document.body.classList.add('is-clicking'), {
      passive: true,
      signal,
    })
    window.addEventListener('pointerup', () => document.body.classList.remove('is-clicking'), {
      passive: true,
      signal,
    })
    window.addEventListener('pointerenter', () => setVisibility(true), { signal })
    window.addEventListener('pointerleave', () => setVisibility(false), { signal })
    window.addEventListener(
      'resize',
      () => {
        measure()
        initCenter()
      },
      { passive: true, signal },
    )
  }

  document.addEventListener(
    'visibilitychange',
    () => {
      if (document.hidden) {
        setVisibility(false)
        cancelAnimationFrame(rafId)
        rafId = 0
        document.body.classList.remove('is-clicking')
      } else {
        measure()
        initCenter()
        setVisibility(!state.disabled)
      }
    },
    { signal },
  )
})

onUnmounted(() => {
  if (aborter) {
    aborter.abort()
    aborter = null
  }
  cancelAnimationFrame(rafId)
  rafId = 0
  document.body.classList.remove('is-clicking')
})
</script>

<style scoped>
:root {
  --fg: #0f0f0f;
  --muted: #6b6b6b;
  --ring: #0f0f0f;
  --accent: #1f1f1f;
  --rx: 0deg;
  --ry: 0deg;
  --tx: 0px;
  --ty: 0px;
}

/* black stage (real bg) */
.page {
  height: 100dvh;
  background: #000;
  color: var(--fg);
  font-family:
    'Helvetica Neue',
    Helvetica,
    Arial,
    system-ui,
    -apple-system,
    'Segoe UI',
    sans-serif;
  perspective: 800px;
  overflow: hidden; /* page itself doesn’t scroll */
}

/* WHITE SHEET: header / scrollable main / footer */
.tiltwrap {
  width: 100vw;
  height: 100dvh;
  margin: 0;
  background: #fff;
  box-sizing: border-box;

  display: grid;
  grid-template-rows: auto 1fr auto;
  align-items: stretch;

  transform: translate3d(var(--tx), var(--ty), 0) rotateX(var(--rx)) rotateY(var(--ry));
  transform-style: preserve-3d;
  transition: transform 140ms cubic-bezier(0.2, 0.8, 0.2, 1);
  will-change: transform;
}

/* middle area can scroll; header/footer stay pinned */
.main {
  min-height: 0; /* allow 1fr row to shrink */
  overflow: auto; /* internal scroll only */
  -webkit-overflow-scrolling: touch;
  padding: 32px 24px 24px; /* slightly less top space */
}

.header,
.footer {
  padding: 16px 24px;
}
.brand {
  letter-spacing: 0.04em;
  font-weight: 700;
  font-size: 14px;
  text-transform: lowercase;
}
.brand:hover {
  text-decoration: underline;
}

.hero {
  max-width: 1080px;
  margin: 0 auto;
  display: grid;
  grid-template-columns: 220px 1fr;
  gap: 48px;
  align-items: center;
}
.avatar {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  overflow: hidden;
  position: relative;
  z-index: 5;
}
.avatar::before {
  content: '';
  position: absolute;
  inset: -6px;
  border-radius: 50%;
  background:
    radial-gradient(closest-side, transparent 72%, var(--ring) 73% 74%, transparent 75%),
    conic-gradient(
      from 180deg at 50% 50%,
      #000 0 10%,
      #888 10% 20%,
      #000 20% 30%,
      #888 30% 40%,
      #000 40% 50%,
      #888 50% 60%,
      #000 60% 70%,
      #888 70% 80%,
      #000 80% 90%,
      #888 90% 100%
    );
  opacity: 0.1;
}
.avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  filter: grayscale(20%);
  position: relative;
  z-index: 1;
}

.name {
  font-size: clamp(36px, 6vw, 64px);
  line-height: 1.05;
  font-weight: 800;
  letter-spacing: -0.02em;
}
.me-text {
  display: inline-block;
}
.me-text:hover {
  transform: scale(1.2);
  text-decoration: underline;
}
.blurb {
  margin-top: 18px;
  max-width: 66ch;
  font-size: 18px;
  line-height: 1.55;
}

.section {
  max-width: 1080px;
  margin: 56px auto 0;
}
.section h2 {
  margin: 0 0 18px 0;
  font-size: 14px;
  text-transform: lowercase;
  letter-spacing: 0.08em;
  font-weight: 700;
  color: var(--muted);
}
.xp {
  display: grid;
  gap: 12px;
  padding: 0;
  margin: 0;
  list-style: none;
}
.xp-item {
  display: grid;
  grid-template-columns: 180px 1fr;
  gap: 24px;
  padding: 16px 0;
  border-top: 1px solid rgba(0, 0, 0, 0.06);
}
.xp-item:last-child {
  border-bottom: 1px solid rgba(0, 0, 0, 0.06);
}
.xp-company {
  font-weight: 700;
  cursor: pointer;
}
.xp-company:hover {
  transform: scale(1.05);
  text-decoration: underline;
}
.xp-meta {
  color: var(--muted);
  text-transform: lowercase;
  font-size: 14px;
}
.cta-row {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  margin-top: 28px;
}

/* primevue buttons */
:deep(.p-button) {
  border-radius: 999px;
  padding: 10px 16px;
  text-transform: lowercase;
  font-weight: 600;
  letter-spacing: 0.02em;
  transition:
    transform 120ms cubic-bezier(0.2, 0.8, 0.2, 1),
    box-shadow 120ms ease;
  transform: translateZ(0);
  backface-visibility: hidden;
}
:deep(.p-button:hover) {
  transform: scale(1.2);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.footer {
  max-width: 1080px;
  margin: 0 auto;
  width: 100%;
  border-top: 1px solid rgba(0, 0, 0, 0.06);
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
.foot-links {
  display: flex;
  gap: 18px;
  flex-wrap: wrap;
  align-items: center;
}
.foot-links a {
  color: var(--fg);
  text-decoration: none;
  text-transform: lowercase;
}
.foot-links a:hover {
  transform: scale(1.15);
  text-decoration: underline;
}

@media (max-width: 800px) {
  .hero {
    grid-template-columns: 1fr;
    gap: 24px;
  }
  .avatar {
    margin: 0 auto;
  }
}

/* compress vertical space on short viewports */
@media (max-height: 760px) {
  .main {
    padding-top: 16px;
  }
  .hero {
    gap: 32px;
  }
  .name {
    font-size: clamp(32px, 5vw, 56px);
  }
  .avatar {
    width: 180px;
    height: 180px;
  }
}
@media (max-height: 640px) {
  .main {
    padding-top: 8px;
  }
  .header {
    padding-top: 8px;
    padding-bottom: 8px;
  }
  .avatar {
    width: 160px;
    height: 160px;
  }
}
</style>

<!-- GLOBAL (unscoped) -->
<style>
html,
body {
  height: 100%;
  background: #000;
  margin: 0;
}

/* cursor defaults */
:root {
  --x: 50vw;
  --y: 50vh;
  --d: 128px;
  --theta: 0deg;
  --sx: 1;
  --sy: 1;
  --soft: 1.5px;
  --invert-contrast: 1.35;
  --invert-bright: 1;
  --laser-size: 7px;
  --laser-color: #faf7f3;
}

/* stop invert on avatar */
html.over-avatar .cursor-bubble {
  -webkit-backdrop-filter: none !important;
  backdrop-filter: none !important;
  box-shadow: none;
  background: transparent;
}

/* disable on touch / reduced motion */
.no-bubble .cursor-layer,
.no-bubble .laser-dot {
  display: none !important;
}
.no-bubble .page,
.no-bubble .page * {
  cursor: auto !important;
}
.no-bubble .tiltwrap {
  transform: none !important;
  transition: none !important;
}

/* fade blob when we hide it (leave window / hidden tab) */
.cursor-layer,
.laser-dot {
  opacity: 1;
  transition: opacity 120ms ease;
}
.cursor-hidden .cursor-layer,
.cursor-hidden .laser-dot {
  opacity: 0;
}

.cursor-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 4;
}
.cursor-wrap {
  position: fixed;
  left: var(--x);
  top: var(--y);
  transform: translate(-50%, -50%) rotate(var(--theta)) scale(var(--sx), var(--sy));
  transform-origin: 50% 50%;
  will-change: transform;
}
.cursor-bubble {
  width: var(--d);
  height: var(--d);
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.001);
  -webkit-backdrop-filter: invert(1) contrast(var(--invert-contrast))
    brightness(var(--invert-bright));
  backdrop-filter: invert(1) contrast(var(--invert-contrast)) brightness(var(--invert-bright));
  box-shadow: 0 0 0 var(--soft) rgba(0, 0, 0, 0);
}
.page,
.page * {
  cursor: none !important;
}

.laser-dot {
  position: fixed;
  left: var(--x);
  top: var(--y);
  width: var(--laser-size);
  height: var(--laser-size);
  transform: translate(-50%, -50%);
  border-radius: 50%;
  background: var(--laser-color);
  pointer-events: none;
  z-index: 9999;
}
@keyframes laser-pulse {
  0% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
  60% {
    transform: translate(-50%, -50%) scale(1.6);
    opacity: 0.2;
  }
  100% {
    transform: translate(-50%, -50%) scale(1);
    opacity: 1;
  }
}
body.is-clicking .laser-dot {
  animation: laser-pulse 220ms ease-out;
}
</style>
