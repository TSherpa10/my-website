<template>
  <div class="page">
    <!-- tilt/translate the whole content -->
    <div class="tiltwrap">
      <header class="header">
        <div class="brand" @click="open('https://github.com/TSherpa10/my-website')">my portfolio</div>
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
          <a
            :href="links.linkedin"
            target="https://www.linkedin.com/in/tsherpa10/"
            rel="noreferrer noopener"
            >linkedin</a
          >
          <a :href="links.github" target="https://github.com/TSherpa10" rel="noreferrer noopener"
            >github</a
          >
          <a :href="links.gamedev" target="https://notoshi.itch.io/" rel="noreferrer noopener"
            >game dev</a
          >
        </nav>
        <div class="brand">© {{ year }} tashi sherpa</div>
      </footer>
    </div>

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

const imageSrc = ref<string>(
  'https://media.licdn.com/dms/image/v2/D4E03AQGfWDjhgOlNpg/profile-displayphoto-crop_800_800/B4EZoCjxODIMAI-/0/1760979522831?e=1762992000&v=beta&t=Ga7gq9y2CY1vt51js5CkFvNwbpijXUcwUWHlZkLHK64',
)
const hobbies = ref<string>('sports + cooking + photography + nature + game dev')
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
  if (!href || href === '#') return
  window.open(href, '_blank', 'noopener')
}

const year = computed(() => new Date().getFullYear())

function onMove(e: PointerEvent) {
  const w = window.innerWidth
  const h = window.innerHeight
  const nx = (e.clientX / w) * 2 - 1 // -1..1
  const ny = (e.clientY / h) * 2 - 1 // -1..1

  const root = document.documentElement
  root.style.setProperty('--x', `${e.clientX}px`)
  root.style.setProperty('--y', `${e.clientY}px`)

  // page tilt + subtle translate
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

  // bubble orientation/warp tied to same movement
  const theta = (Math.atan2(ny, nx) * 180) / Math.PI
  root.style.setProperty('--theta', `${theta.toFixed(2)}deg`)
  root.style.setProperty('--sx', `${(1 + Math.abs(nx) * 0.25).toFixed(3)}`)
  root.style.setProperty('--sy', `${(1 + Math.abs(ny) * 0.18).toFixed(3)}`)

  const hit = document.elementFromPoint(e.clientX, e.clientY) as HTMLElement | null
  document.documentElement.classList.toggle('over-avatar', !!hit && !!hit.closest('.avatar'))
}

const onDown = () => document.body.classList.add('is-clicking')
const onUp = () => document.body.classList.remove('is-clicking')

onMounted(() => {
  const root = document.documentElement
  root.style.setProperty('--x', '50vw')
  root.style.setProperty('--y', '50vh')
  root.style.setProperty('--rx', '0deg')
  root.style.setProperty('--ry', '0deg')
  root.style.setProperty('--tx', '0px')
  root.style.setProperty('--ty', '0px')

  // bubble base (no mask needed)
  root.style.setProperty('--d', '128px') // diameter; tweak
  root.style.setProperty('--theta', '0deg')
  root.style.setProperty('--sx', '1')
  root.style.setProperty('--sy', '1')

  // ensure text becomes *white* (no dulling)
  root.style.setProperty('--invert-contrast', '1.35')
  root.style.setProperty('--invert-bright', '1.00') // keep whites pure

  // soft edge thickness (sub-pixel AA; no blur())
  root.style.setProperty('--soft', '1.5px')

  window.addEventListener('pointermove', onMove, { passive: true })
  window.addEventListener('pointerdown', onDown, { passive: true })
  window.addEventListener('pointerup', onUp, { passive: true })
})
onUnmounted(() => {
  window.removeEventListener('pointermove', onMove)
  window.removeEventListener('pointerdown', onDown)
  window.removeEventListener('pointerup', onUp)
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

.page {
  min-height: 100vh;
  background: black;
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
  overflow: hidden;
}

.tiltwrap {
  /* size & look */
  width: 100vw;
  min-height: 100vh;
  margin: 0; /* remove margins so it “reads” as the page */
  background: #fff; /* white front */
  box-sizing: border-box;

  /* keep your existing wobble */
  transform: translate3d(calc(var(--tx)), var(--ty), 0) rotateX(var(--rx)) rotateY(var(--ry));
  transform-style: preserve-3d;
  transition: transform 140ms cubic-bezier(0.2, 0.8, 0.2, 1);
  will-change: transform;
}

.header,
.footer {
  padding: 24px;
}
.main {
  padding: 48px 24px 32px;
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
  position: relative;
  width: 200px;
  height: 200px;
  position: relative;
  border-radius: 50%;
  overflow: hidden;
  box-shadow: none;
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

/* primevue buttons (minimal) */
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
</style>

<!-- GLOBAL (unscoped) — visible invert bubble without magnify -->
<style>
html,
body {
  height: 100%;
  background: #000; /* make the real stage black */
}

body {
  margin: 0; /* <— fixes the thin outer ring */
}

:root {
  --x: 50vw; /* cursor x */
  --y: 50vh; /* cursor y */
  --d: 128px; /* bubble diameter */
  --theta: 0deg; /* bubble rotation */
  --sx: 1; /* bubble warp X */
  --sy: 1; /* bubble warp Y */

  /* sub-pixel anti-aliased edge width (no blur()) */
  --soft: 1.5px;

  /* inverted text should be pure white */
  --invert-contrast: 1.35;
  --invert-bright: 1;

  --laser-size: 7px; /* diameter of the red dot */
  --laser-color: #faf7f3; /* laser */
}

html.over-avatar .cursor-bubble {
  -webkit-backdrop-filter: none !important;
  backdrop-filter: none !important;
  box-shadow: none;
  background: transparent;
}

.cursor-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 4;
}

/* position the bubble directly at the cursor */
.cursor-wrap {
  position: fixed;
  left: var(--x);
  top: var(--y);
  transform: translate(-50%, -50%) rotate(var(--theta)) scale(var(--sx), var(--sy));
  transform-origin: 50% 50%;
  will-change: transform;
}

/* the element itself is only the circle area; backdrop-filter inverts what's beneath it */
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
