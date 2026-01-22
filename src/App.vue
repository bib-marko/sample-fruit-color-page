<template>
  <!-- FULLSCREEN LOADING -->
  <div id="fireworks-layer"></div>
  <div
    v-if="loading"
    class="loading-screen"
    :class="{ exiting: loadingExiting }"
    @click="onLoadingTap"
    @touchstart="onLoadingTap"
  >
    <div class="loading-content">
      <img src="/public/img/megabet_logo.webp" class="animate__animated animate__bounce animate__infinite" />
      <div class="loading-text">TAP TO START...</div>
    </div>
  </div>

  <div
    v-else
    class="wheel-stage-vertical loaded"
  >
    <div
      v-if="showIntroText"
      class="spin-intro-overlay"
      ref="introContainer"
    >
      <span ref="line1">SPIN THE WHEELS</span>
      <br />
      <span ref="line2">TO WIN YOUR BONUS!</span>
    </div>

    <div
      class="wheel wheel-top"
      :class="{ 'wheel-disabled': topDisabled }"
      :style="{ top: IS_LANDSCAPE ? '0' : '8%' }"
    >
      <Wheel
        ref="topWheel"
        position="top"
        :prizes="prizesTop"
        :disabled="topDisabled"
        @spin-request="startSpin"
        @finished="onTopFinished"
      />
    </div>

    <!-- BOTTOM -->
    <div
      class="wheel-wrapper"
      :class="{
        'wheel-disabled': step === 0,
        'wheel-active': step === 2
      }"
    >
      <div class="wheel wheel-bottom">
        <Wheel
          ref="bottomWheel"
          position="bottom"
          :prizes="prizesBttom"
          :disabled="bottomDisabled"
          @spin-request="startSpin"
          @finished="onBottomFinished"
        />
      </div>
    </div>

    <!-- SPIN ROW -->
    <div class="spin-row">
      <!-- <div class="spin-result">
        PHP {{ data }}
      </div> -->

      <!-- <button
        class="golden-button"
        @click="startSpin"
        :disabled="spinning || step >= 2"
      >
        SPIN
      </button> -->

      
      <div class="comic-brutal-button-container">
  <button class="comic-brutal-button">
    <div class="button-inner">
      <span class="button-text"> ₱ {{ data }}</span>
      <div class="halftone-overlay"></div>
      <div class="ink-splatter"></div>
    </div>
    <div class="button-shadow"></div>
    <div class="button-frame"></div>
  </button>
</div>

      
    </div>
    <div class="icons">
      <img src="/public/img/TAO.webp" v-if="!IS_LANDSCAPE" class="girl" />
      <img src="/public/img/tree.webp" v-if="!IS_LANDSCAPE" class="tree" />
    </div>
    
  </div>
</template>


<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import Wheel from './components/Wheel.vue'
import Swal from 'sweetalert2'
import { useOrientation } from '../src/service/orientation'
import { Fireworks } from 'fireworks-js'
import gsap from 'gsap'
const { IS_LANDSCAPE } = useOrientation()

const prizesTop = [
  { id: 1, label: '200', color: '#dd3832', weight: 1 },
  { id: 2, label: '500', color: '#dd3832', weight: 1 },
  { id: 3, label: '50', color: '#dd3832', weight: 1 },
  { id: 4, label: '200', color: '#dd3832', weight: 1 },
  { id: 5, label: '500', color: '#dd3832', weight: 1 },
  { id: 6, label: '50', color: '#dd3832', weight: 1 },
]

const prizesBttom = [
  { id: 1, label: '300', color: '#45ace9', weight: 1 },
  { id: 2, label: '100', color: '#45ace9', weight: 1 },
  { id: 3, label: '50', color: '#45ace9', weight: 1 },
  { id: 4, label: '300', color: '#45ace9', weight: 1 },
  { id: 5, label: '100', color: '#45ace9', weight: 1 },
  { id: 6, label: '50', color: '#45ace9', weight: 1 },
]

const showIntroText = ref(true)
const topWheel = ref<any>(null)
const bottomWheel = ref<any>(null)

const data = ref(0);
const step = ref(0)
const spinning = ref(false)

const topDisabled = ref(false)
const bottomDisabled = ref(true)

function startSpin() {
  showIntroText.value = false
  gsap.killTweensOf([line1.value, line2.value])

  if (spinning.value) return
  spinning.value = true

  if (step.value === 0) {
    topWheel.value.spin(1)
  } else if (step.value === 1) {
    bottomWheel.value.spin(1)
  }
}


  function onTopFinished() {
    topDisabled.value = true
    bottomDisabled.value = false
    spinning.value = false
    step.value = 1
    data.value = 200;
  }


  let fireworks: Fireworks | null = null

function startFireworks() {
  if (fireworks) return

  const container = document.getElementById('fireworks-layer')
  if (!container) return

  fireworks = new Fireworks(container, {
    autoresize: true,
    opacity: 1,                 // brighter
    acceleration: 1.03,
    friction: 0.97,
    gravity: 1.1,               // floaty
    particles: 140,
    traceLength: 4,
    traceSpeed: 10,
    explosion: 7,
    intensity: 32,
    flickering: 75,             // sparkle
    lineStyle: 'round',

    // 🌟 LIGHT COLORS
    hue: {
      min: 40,                  // gold
      max: 65                   // light yellow
    },

    delay: { min: 15, max: 30 },
    rocketsPoint: { min: 20, max: 80 }
  })

  fireworks.start()
}


function stopFireworks() {
  fireworks?.stop()
  fireworks = null
}

function onBottomFinished() {
  bottomDisabled.value = true
  spinning.value = false
  step.value = 2

  data.value = data.value + 300

  // 🎆 START FIREWORKS
  startFireworks()

  Swal.fire({
    title: "CONGRATULATIONS!",
    width: 500,
    background: "#fff url(/images/trees.png)",
    confirmButtonText: "CLAIM NOW",
    text: "Your Welcome Bonus Spin rewarded you with a 200 + 300 bonus. Have fun and enjoy!",
    allowOutsideClick: false,
  }).then((result) => {
    // 🛑 STOP FIREWORKS
    stopFireworks()

    if (result.isConfirmed) {
      window.location.href = "https://megabet-paradise.com/welcome-bonus"
    }
  })
}

const introContainer = ref<HTMLElement | null>(null)
const line1 = ref<HTMLElement | null>(null)
const line2 = ref<HTMLElement | null>(null)

function playIntroReveal() {
  if (!line1.value || !line2.value) return

  gsap.killTweensOf([line1.value, line2.value])

  gsap.set([line1.value, line2.value], {
    opacity: 0,
    y: 30,
    scale: 0.95,
    filter: 'blur(6px)'
  })

  gsap.timeline()
    .to(line1.value, {
      opacity: 1,
      y: 0,
      scale: 1,
      filter: 'blur(0px)',
      duration: 0.9,
      ease: 'power4.out'
    })
    .to(line2.value, {
      opacity: 1,
      y: 0,
      scale: 1,
      filter: 'blur(0px)',
      duration: 0.9,
      ease: 'power4.out'
    }, '+=0.2')
}



function updateBodyBackground() {
  document.body.style.backgroundImage = IS_LANDSCAPE.value
    ? "url('/img/bg.webp')"
    : "url('/img/bg-mobile.png')"

  document.body.style.backgroundSize = 'cover'
  document.body.style.backgroundPosition = 'center'
  document.body.style.backgroundRepeat = 'no-repeat'
}

let bgMusic: HTMLAudioElement | null = null
const musicUnlocked = ref(false)

function setupMusic() {
  if (bgMusic) return

  bgMusic = new Audio('/public/music/music-1.mp3')
  bgMusic.loop = true
  bgMusic.volume = 0.35
  bgMusic.preload = 'auto'
  bgMusic.muted = true
}

function onLoadingTap() {
  // 🔓 unlock music (iOS safe)
  if (bgMusic && !musicUnlocked.value) {
    bgMusic.muted = false
    bgMusic.play().then(() => {
      musicUnlocked.value = true
    })
  }

  // 🎬 exit loading screen
  loadingExiting.value = true

  setTimeout(() => {
    loading.value = false
  }, 250)
}

const loading = ref(true)
const loadingExiting = ref(false)

function preloadFromGlob(globResult: Record<string, () => Promise<any>>) {
  return Promise.all(
    Object.values(globResult).map((importer) =>
      importer().then((mod) => {
        return new Promise<void>((resolve) => {
          const img = new Image()
          img.src = mod.default
          img.onload = () => resolve()
          img.onerror = () => resolve()
        })
      })
    )
  )
}

function wait(ms: number) {
  return new Promise(resolve => setTimeout(resolve, ms))
}

onMounted(async () => {
  setupMusic()
  updateBodyBackground()
  const assetImages = import.meta.glob(
    '/src/assets/**/*.{png,jpg,jpeg,webp,gif,svg}'
  )

  const publicImages = import.meta.glob(
    '/public/img/**/*.{png,jpg,jpeg,webp,gif,svg}'
  )

  await Promise.all([
    preloadFromGlob(assetImages),
    preloadFromGlob(publicImages),
    wait(1000),
  ])

  playIntroReveal()

  // loading.value = false
})


watch(IS_LANDSCAPE, () => {
  updateBodyBackground()
})

watch(showIntroText, async (visible) => {
  if (!visible) return

  // ⏳ wait for v-if DOM to exist
  await nextTick()

  playIntroReveal()
}, { immediate: true })


</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Bagel+Fat+One&family=Geist:wght@100..900&family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&family=Work+Sans:ital,wght@0,100..900;1,100..900&display=swap');

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  font-family: 'Poppins';
}

body {
  position: relative;
  overflow-x: hidden;
  overflow-y: hidden;
  background: url('/public/img/bg.webp') center center / cover no-repeat;
}

/* BLACK OVERLAY */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  z-index: 0;
  pointer-events: none;
}
.wheel-stage-vertical {
  margin-top: 3%;
  position: relative;
  width: 100%;
  height: 90vh;
}

/* COMMON */
/* wrapper controls visual state */
.wheel-wrapper {
  position: absolute;
  left: 50%;
  bottom: 20%;
  transform: translateX(-50%);
  z-index: 1;
}

/* actual wheel keeps transform only */
.wheel {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  filter: drop-shadow(0 0 1vmin #000000);
}
/* DISABLED STATE */
.wheel-wrapper.wheel-disabled {
  filter: grayscale(.8) brightness(0.6);
  transition: filter 0.3s ease;
}

/* ACTIVE GLOW */
.wheel-wrapper.wheel-active {
  filter:
    drop-shadow(0 0 0.4rem rgba(255, 215, 0, 0.5))
    drop-shadow(0 0 1rem rgba(255, 215, 0, 0.65))
    drop-shadow(0 0 2rem rgba(255, 180, 0, 0.85))
    drop-shadow(0 0 3rem rgba(255, 150, 0, 0.6));
  transition: filter 0.45s ease;
}
/* TOP */
.wheel-top {
  z-index: 1;
}

/* BOTTOM */
.wheel-bottom {
  bottom: 20%;
  z-index: 1;
}


.spin-row {
  position: fixed;
  bottom: 5%;
  left: 50%;
  transform: translateX(-50%) translateZ(0); /* iOS safe */
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0.75rem;
  /* background: rgba(0, 0, 0, 0.45); */
  border-radius: 0.6rem;
  z-index: 2;
}

.spin-result {
  min-width: 5rem;
  padding: 0.4rem 0.6rem;
  font-size: 1.2rem;
  font-weight: 700;
  color: black;
  background: white;
  border-radius: 0.4rem;
  text-align: center;

    /* glow */
  filter: drop-shadow(0 0 0.4rem rgba(255, 215, 0, 0.6));
  animation: spinRowPulse 1.8s ease-in-out infinite;
  will-change: filter;
}


@keyframes spinRowPulse {
  0% {
    filter:
      drop-shadow(0 0 0.3rem rgba(255, 215, 0, 0.35))
      drop-shadow(0 0 0.6rem rgba(255, 180, 0, 0.25));
  }

  50% {
    filter:
      drop-shadow(0 0 0.8rem rgba(255, 215, 0, 0.8))
      drop-shadow(0 0 1.6rem rgba(255, 180, 0, 0.6))
      drop-shadow(0 0 2.4rem rgba(255, 140, 0, 0.45));
  }

  100% {
    filter:
      drop-shadow(0 0 0.3rem rgba(255, 215, 0, 0.35))
      drop-shadow(0 0 0.6rem rgba(255, 180, 0, 0.25));
  }
}


.golden-button {
  position: relative; /* 👈 important */
  touch-action: manipulation;
  display: inline-block;
  outline: none;
  font-family: inherit;
  font-size: 1em;
  box-sizing: border-box;
  border: none;
  border-radius: 0.3em;
  height: 2.75em;
  line-height: 2.5em;
  text-transform: uppercase;
  padding: 0 1.2em;
  font-weight: bolder;

  box-shadow:
    0 3px 6px rgba(0, 0, 0, 0.16),
    0 3px 6px rgba(110, 80, 20, 0.4),
    inset 0 -2px 5px 1px rgba(139, 66, 8, 1),
    inset 0 -1px 1px 3px rgba(250, 227, 133, 1);

  background-image: linear-gradient(
    160deg,
    #a54e07,
    #b47e11,
    #fef1a2,
    #bc881b,
    #a54e07
  );

  border: 1px solid #a55d07;
  color: rgb(120, 50, 5);
  text-shadow: 0 2px 2px rgba(250, 227, 133, 1);
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}

.golden-button:focus,
.golden-button:hover {
  background-size: 150% 150%;
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.19), 0 6px 6px rgba(0, 0, 0, 0.23),
    inset 0 -2px 5px 1px #b17d10, inset 0 -1px 1px 3px rgba(250, 227, 133, 1);
  border: 1px solid rgba(165, 93, 7, 0.6);
  color: rgba(120, 50, 5, 0.8);
}

.golden-button:active {
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.16), 0 3px 6px rgba(110, 80, 20, 0.4),
    inset 0 -2px 5px 1px #b17d10, inset 0 -1px 1px 3px rgba(250, 227, 133, 1);
}

.wheel-disabled {
  filter: grayscale(5%) brightness(0.45);
  transition: filter 0.3s ease;
}

.wheel-active {
  filter:
    drop-shadow(0 0 0.4rem rgba(255, 215, 0, 0.5))
    drop-shadow(0 0 1rem rgba(255, 215, 0, 0.65))
    drop-shadow(0 0 2rem rgba(255, 180, 0, 0.85))
    drop-shadow(0 0 3rem rgba(255, 150, 0, 0.6));
  transition:
    filter 0.45s ease,
    transform 0.45s ease;
}

.swal2-popup {
  position: relative;
  padding: .9em !important;
  background: #faf2dd !important;
  border-radius: 16px;
  overflow: hidden;
}

.swal2-popup::before {
  content: '';
  position: absolute;
  inset: 0;
  padding: 5px;
  background: linear-gradient(to bottom, #ffb620, #ff8c26);
  border-radius: 16px;
  -webkit-mask:
    linear-gradient(#fff 0 0) content-box,
    linear-gradient(#fff 0 0);
  -webkit-mask-composite: xor;
  mask-composite: exclude;
}

.swal2-title {
  font-size: 28px;
  color:#dd692b;
}


.swal2-html-container {
  color:#ba8966;
}

.swal2-confirm {
  width: 150px;
  height: 40px;
  margin-bottom: .2em !important;
  font-size: .875rem !important;
  padding: .3rem !important;
  border-radius: 2rem !important;
  color: #fdf9d7 !important;
  font-weight: 700 !important;
  border: .124rem solid #175813 !important;
  border-bottom: .125rem solid #175813 !important;
  background: linear-gradient(to bottom, #4ed107, #1c8f00) !important;
  text-shadow: .1em 0 0 #316826, -.1em 0 0 #316826, 0 .1em 0 #316826, 0 -.08em 0 #316826, -.08em -.08em 0 #316826, .08em -.08em 0 #316826, -.08em .08em 0 #316826, .08em .08em 0 #316826, 0 .15em .1em #0000004b !important; 
  box-shadow: 0 .15em .2em #00000047 !important;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.spin-intro-overlay {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  line-height: 12px;
  width: 80%;
  text-align: center;

  z-index: 2;
  pointer-events: none;
}

.spin-intro-overlay span {
  display: inline-block;

  padding: 0.6rem 1.2rem;
  font-size: clamp(1.4rem, 3.5vw, 2.2rem);
  font-weight: 900;
  letter-spacing: 0.08em;

  color: #fff;
  text-transform: uppercase;

  text-shadow:
    0 0 6px rgba(255, 215, 0, 0.9),
    0 0 16px rgba(255, 180, 0, 0.8),
    0 0 28px rgba(255, 140, 0, 0.6);
}

.loading-screen {
  position: fixed;
  inset: 0;
  z-index: 9999;
  background-image: radial-gradient(circle, orange, transparent 20%, orangered);
  background-size: cover;
  background-repeat: no-repeat;
  background-color: orange;
  display: flex;
  align-items: center;
  justify-content: center;
  opacity: 1;
  transition: opacity 0.25s ease;
}

.loading-screen.exiting {
  opacity: 0;
  transform: scale(1.05);
}

.loading-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.loading-content img {
    width: 50%;
}

.loading-spinner {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  border: 4px solid rgba(0, 0, 0, 0.25);
  border-top-color: white;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.loading-text {
  font-weight: 800;
  letter-spacing: 0.2em;
  color: white;
  font-size: 0.9rem;
}

.wheel-stage-vertical {
  opacity: 0;
  transition: opacity 0.45s ease;
}

.wheel-stage-vertical.loaded {
  opacity: 1;
}

.icons {
  margin-top: -1rem;
}
.girl {
  position: fixed;        /* stick to viewport */
  left: -10%;              /* adjust spacing */
  bottom: 0;              /* bottom aligned */
  z-index: 1;             /* above background, below loader/UI */
  pointer-events: none;  /* optional: clicks pass through */
  width: 220px; /* responsive size */
  height: auto;
}

.tree {
  position: fixed;        /* stick to viewport */
  right: -30%;              /* adjust spacing */
  bottom: 0%;              /* bottom aligned */
  z-index: 1;             /* above background, below loader/UI */
  pointer-events: none;  /* optional: clicks pass through */
  width: 300px; /* responsive size */
  height: auto;
}

#fireworks-layer {
  position: fixed;
  inset: 0;
  z-index: 2; /* below loader (9999), above game */
  pointer-events: none;
}

/* From Uiverse.io by dopekid_9309 – MODIFIED (NO HOVER, AUTO ANIMATE) */

.comic-brutal-button-container {
  --primary-color: #ff3d3d;
  --secondary-color: #3d3dff;
  --text-color: #000000;
  --accent-color: #ffef00;
  --panel-color: #ffffff;
  --shadow-color: rgba(0, 0, 0, 0.75);

  display: flex;
  justify-content: center;
  align-items: center;
}

/* ===========================
   BASE BUTTON
=========================== */

.comic-brutal-button {
  position: relative;
  font-size: 1.5em;
  border: none;
  background: none;
  cursor: pointer;
  padding: 0;
  transform-origin: center;
  outline: none;

  /* 🔥 AUTO IDLE ANIMATION */
  animation: comic-idle 2.2s ease-in-out infinite;
}

/* ===========================
   INNER PANEL
=========================== */

.button-inner {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: var(--primary-color);
  color: var(--text-color);
  padding: 0.8em 1.5em;
  z-index: 3;
  overflow: hidden;

  transform: skew(-3deg, 1deg);

  clip-path: polygon(
    0% 10%,
    3% 0%,
    97% 0%,
    100% 10%,
    100% 90%,
    97% 100%,
    3% 100%,
    0% 90%
  );

  animation: inner-idle 2.2s ease-in-out infinite;
}

/* ===========================
   TEXT
=========================== */

.button-text {
  position: relative;
  font-weight: 900;
  font-size: 1.2em;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  z-index: 5;
  color: var(--text-color);

  text-shadow:
    0.05em 0.05em 0 var(--panel-color),
    -0.05em -0.05em 0 var(--panel-color),
    0.05em -0.05em 0 var(--panel-color),
    -0.05em 0.05em 0 var(--panel-color);

  transform: rotate(2deg);
}

/* ===========================
   FRAME & SHADOW
=========================== */

.button-frame {
  position: absolute;
  top: -0.3em;
  left: -0.3em;
  right: -0.3em;
  bottom: -0.3em;
  background-color: var(--accent-color);
  border: 0.15em solid var(--text-color);
  z-index: 1;
}

.button-shadow {
  position: absolute;
  top: 0.4em;
  left: 0.4em;
  right: -0.4em;
  bottom: -0.4em;
  background-color: var(--shadow-color);
  z-index: 0;
}

/* ===========================
   HALFTONE + INK EFFECTS
=========================== */

.halftone-overlay {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(
    circle at 30% 30%,
    rgba(0, 0, 0, 0.2) 0.1em,
    transparent 0.1em
  );
  background-size: 0.5em 0.5em;
  opacity: 0.3;
  z-index: 2;
  mix-blend-mode: multiply;
}

.ink-splatter {
  position: absolute;
  inset: 0;
  z-index: 1;
  opacity: 0;
  background:
    radial-gradient(circle at 20% 30%, rgba(0,0,0,0.15), transparent 50%),
    radial-gradient(circle at 70% 65%, rgba(0,0,0,0.15), transparent 50%),
    radial-gradient(circle at 40% 50%, rgba(0,0,0,0.15), transparent 60%),
    radial-gradient(circle at 85% 15%, rgba(0,0,0,0.15), transparent 40%);

  animation: ink-flicker 2.2s infinite;
}

/* ===========================
   ACTIVE (TAP) STATE
=========================== */

.comic-brutal-button:active {
  animation: none;
  transform: scale(0.97);
}

.comic-brutal-button:active .button-inner {
  transform: skew(0, 0);
}

.comic-brutal-button:active .button-shadow {
  transform: translate(0.15em, 0.15em);
}

.comic-brutal-button:active .button-text {
  animation: burst 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

/* ===========================
   ANIMATIONS
=========================== */

@keyframes comic-idle {
  0%   { transform: rotate(-2deg) scale(1); }
  25%  { transform: rotate(-1deg) scale(1.03); }
  50%  { transform: rotate(-2deg) scale(1.05); }
  75%  { transform: rotate(-3deg) scale(1.03); }
  100% { transform: rotate(-2deg) scale(1); }
}

@keyframes inner-idle {
  0%, 100% {
    transform: skew(-3deg, 1deg);
    background-color: var(--primary-color);
  }
  50% {
    transform: skew(-5deg, 2deg);
    background-color: var(--secondary-color);
  }
}

@keyframes ink-flicker {
  0%, 70%, 100% { opacity: 0; }
  80% { opacity: 0.6; }
}

@keyframes burst {
  0%   { transform: scale(0.95); }
  50%  { transform: scale(1.05); }
  100% { transform: scale(1); }
}



</style>
