<template>
  <div class="wheel-stage-vertical">
    <!-- TOP -->
    <div class="wheel wheel-top" :class="{ 'wheel-disabled': topDisabled }">
      <Wheel
        ref="topWheel"
        :position="'top'"
        :prizes="prizesTop"
        :disabled="topDisabled"
        @finished="onTopFinished"
      />
    </div>

    <!-- BOTTOM -->
    <div class="wheel wheel-bottom" :class="{ 'wheel-disabled': bottomDisabled  }">
      <Wheel
        ref="bottomWheel"
        :position="'bottom'"
        :prizes="prizesBttom"
        :disabled="bottomDisabled"
        @finished="onBottomFinished"
      />
    </div>
    <!-- CENTER -->
    <div class="wheel wheel-center" 
      :class="{
        'wheel-disabled': [0,1].includes(step),
        'wheel-active': step === 3
      }"
      :style="{  top: IS_LANDSCAPE? '42%' : '37%'}"
    >
      <Wheel
        ref="centerWheel"
        :position="'center'"
        :prizes="prizesCenter"
        :disabled="centerDisabled"
        @finished="onCenterFinished"
      />
    </div>

    <!-- SPIN BUTTON -->
   <!-- <button
  class="Btn"
  @click="startSpin"
  :disabled="spinning || step >= 3"
>
  SPIN
</button> -->

<button role="button" class="golden-button" :style="{ width: IS_LANDSCAPE ? '15%' : '35%', bottom: IS_LANDSCAPE ? '8%' : '5%',  }"  @click="startSpin"  :disabled="spinning || step >= 3">
  <span class="golden-text">SPIN HERE!</span>
</button>


  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Wheel from './components/Wheel.vue'
import { useOrientation } from "../src/service/orientation";
const { IS_LANDSCAPE } = useOrientation();

const prizesBttom = [
  { id: 1, label: '200', color: '#45ace9', weight: 1 },
  { id: 2, label: '50', color: '#45ace9', weight: 1 },
  { id: 3, label: '100', color: '#45ace9', weight: 1 },
  { id: 4, label: '200', color: '#45ace9', weight: 1 },
  { id: 5, label: '50', color: '#45ace9', weight: 1 },
  { id: 6, label: '100', color: '#45ace9', weight: 1 },
]

const prizesTop = [
  { id: 1, label: '500', color: '#dd3832', weight: 1 },
  { id: 2, label: '50', color: '#dd3832', weight: 1 },
  { id: 3, label: '200', color: '#dd3832', weight: 1 },
  { id: 4, label: '500', color: '#dd3832', weight: 1 },
  { id: 5, label: '50', color: '#dd3832', weight: 1 },
  { id: 6, label: '200', color: '#dd3832', weight: 1 },
]

const prizesCenter = [
  { id: 1, label: '500', color: '#002F05', weight: 1 },
  { id: 2, label: '50', color: '#002F05', weight: 1 },
  { id: 3, label: '200', color: '#002F05', weight: 1 },
  { id: 4, label: '500', color: '#002F05', weight: 1 },
  { id: 5, label: '50', color: '#002F05', weight: 1 },
  { id: 6, label: '200', color: '#002F05', weight: 1 },
]

const bottomWheel = ref<any>(null)
const topWheel = ref<any>(null)
const centerWheel = ref<any>(null)

const step = ref(0)

const spinning = ref(false)

const bottomDisabled = ref(false)
const topDisabled = ref(true)
const centerDisabled = ref(true)

const results = {
  bottom: '',
  top: '',
  center: '',
}

/* MAIN SPIN BUTTON */
function startSpin() {
  if (spinning.value || step.value >= 3) return

  spinning.value = true

  if (step.value === 0) {
    bottomWheel.value.spin()
  } else if (step.value === 1) {
    topWheel.value.spin()
  } else if (step.value === 2) {
    centerWheel.value.spin()
  }
}

/* BOTTOM FINISHED */
function onBottomFinished(prize: any) {
  results.bottom = prize.label
  bottomDisabled.value = true
  topDisabled.value = false

  spinning.value = false
  step.value = 1
}

/* TOP FINISHED */
function onTopFinished(prize: any) {
  results.top = prize.label
  topDisabled.value = true
  centerDisabled.value = false

  spinning.value = false
  step.value = 2
}

/* CENTER FINISHED */
function onCenterFinished(prize: any) {
  results.center = prize.label
  centerDisabled.value = true

  spinning.value = false
  step.value = 3

//   alert(`
// RESULTS:
// Bottom: ${results.bottom}
// Top: ${results.top}
// Center: ${results.center}
//   `)
}
</script>


<style>

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

body {
  position: relative;
  overflow: hidden;
  background: url('/public/img/bg.webp') center center / cover no-repeat;
}

/* BLACK OVERLAY */
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6); /* 👈 adjust opacity */
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
  top: 0;
  z-index: 1;
}

/* BOTTOM */
.wheel-bottom {
  top: 40%;
  bottom: 0;
  z-index: 1;
  
}

/* CENTER */
.wheel-center {
  transform: translate(-50%, -50%);
  z-index: 3;
}


/* From Uiverse.io by elijahgummer */ 
.golden-button {
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  z-index: 50;       

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
  filter: grayscale(95%) brightness(0.45);
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


</style>
