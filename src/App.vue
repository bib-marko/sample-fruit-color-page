<template>
  <div class="wheel-stage-vertical">
    <div
      v-if="showIntroText"
      class="spin-intro-overlay"
    >
      <span>SPIN THE WHEELS</span>
      <br />
      <span>TO WIN YOUR BONUS!</span>
    </div>
    <!-- TOP -->
    <div class="wheel wheel-top" :class="{ 'wheel-disabled': topDisabled }" :style="{ top: IS_LANDSCAPE ? '0': '8%' }">
      <Wheel
        ref="topWheel"
        position="top"
        :prizes="prizesTop"
        :disabled="topDisabled"
        @finished="onTopFinished"
      />
    </div>

    <!-- BOTTOM -->
    <div class="wheel wheel-bottom"    :class="{
        'wheel-disabled': [0].includes(step),
        'wheel-active': step === 2
      }">
      <Wheel
        ref="bottomWheel"
        position="bottom"
        :prizes="prizesBttom"
        :disabled="bottomDisabled"
        @finished="onBottomFinished"
      />
    </div>

    <div class="spin-row">
      <div class="spin-result">
        PHP {{ data }}
      </div>

      <button
        class="golden-button"
        @click="startSpin"
        :disabled="spinning || step >= 2"
      >
        SPIN
      </button>
    </div>

  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Wheel from './components/Wheel.vue'
import Swal from 'sweetalert2'
import { useOrientation } from '../src/service/orientation'
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
  showIntroText.value = false;
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

function onBottomFinished() {
  bottomDisabled.value = true
  spinning.value = false
  step.value = 2

  data.value = data.value + 300;

Swal.fire({
  title: "CONGRATULATIONS!",
  width: 500,
  padding: "3em",
  color: "#716add",
  background: "#fff url(/images/trees.png)",
  confirmButtonText: "CLAIM NOW",
  text: "Congratulations! Your Welcome Bonus Spin rewarded you with a 200 + 300 bonus , enjoy!",
  backdrop: `
    rgba(0,0,123,0.4)
    url("/images/nyan-cat.gif")
    left top
    no-repeat
  `
});


}
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
  overflow: hidden;
  background: url('/public/img/bg-desktop.png') center center / cover no-repeat;
}

/* BLACK OVERLAY */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.3); /* 👈 adjust opacity */
  z-index: 0;
  pointer-events: none;
}
.wheel-stage-vertical {
  margin-top: 4rem;
  position: relative;
  width: 100%;
  height: 90vh;
}

/* COMMON */
.wheel {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  filter: drop-shadow(0 0 1vmin #000000);
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
  transform: translateX(-50%);
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0.5rem 0.75rem;
  background: rgba(0, 0, 0, 0.45);
  border-radius: 0.6rem;
  z-index: 1;
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
}


/* From Uiverse.io by elijahgummer */ 
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
  overflow: hidden; /* 👈 important */
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


</style>
