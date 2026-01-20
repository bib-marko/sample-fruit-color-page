<script setup lang="ts">
import { ref } from 'vue'
import Wheel from './Wheel.vue'

const topWheel = ref<InstanceType<typeof Wheel> | null>(null)
const middleWheel = ref<InstanceType<typeof Wheel> | null>(null)
const bottomWheel = ref<InstanceType<typeof Wheel> | null>(null)

const spinning = ref(false)
const spinStep = ref(0)

/* Example prize data */
const topPrizes = [
  { id: 1, label: '200', color: '#1abc9c', weight: 1 },
  { id: 2, label: '0', color: '#3498db', weight: 1 },
  { id: 3, label: '1000', color: '#9b59b6', weight: 1 },
]

const middlePrizes = [
  { id: 1, label: 'LOSE', color: '#8e44ad', weight: 1 },
  { id: 2, label: 'BONUS', color: '#f1c40f', weight: 1 },
]

const bottomPrizes = [
  { id: 1, label: 'x1', color: '#27ae60', weight: 1 },
  { id: 2, label: 'x2', color: '#2ecc71', weight: 1 },
  { id: 3, label: 'x3', color: '#16a085', weight: 1 },
]

async function spinAll() {
  if (spinning.value) return
  spinning.value = true
  spinStep.value = 0

  // 1️⃣ Bottom wheel
  spinStep.value = 1
  bottomWheel.value?.spin()
  await delay(4500)

  // 2️⃣ Top wheel
  spinStep.value = 2
  topWheel.value?.spin()
  await delay(4500)

  // 3️⃣ Middle wheel
  spinStep.value = 3
  middleWheel.value?.spin()
  await delay(4500)

  spinning.value = false
}

function delay(ms: number) {
  return new Promise(resolve => setTimeout(resolve, ms))
}
</script>

<template>
  <div class="triple-wheel-wrapper">
    <!-- TOP -->
    <div class="wheel top">
      <Wheel ref="topWheel" :prizes="topPrizes" />
    </div>

    <!-- MIDDLE -->
    <div class="wheel middle">
      <Wheel ref="middleWheel" :prizes="middlePrizes" />
    </div>

    <!-- BOTTOM -->
    <div class="wheel bottom">
      <Wheel ref="bottomWheel" :prizes="bottomPrizes" />
    </div>

    <!-- SPIN BUTTON -->
    <button class="spin-btn" @click="spinAll">
      SPIN
    </button>
  </div>
</template>


<style scoped>
    .triple-wheel-wrapper {
  position: relative;
  width: 360px;
  margin: 0 auto;
  padding-bottom: 120px;
}

.wheel {
  position: relative;
  display: flex;
  justify-content: center;
}

.wheel.top {
  margin-bottom: -60px;
  z-index: 1;
  transform: scale(0.85);
}

.wheel.middle {
  margin-bottom: -60px;
  z-index: 2;
}

.wheel.bottom {
  z-index: 3;
  transform: scale(1.05);
}

/* Spin button */
.spin-btn {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  width: 200px;
  height: 56px;
  font-size: 22px;
  font-weight: bold;
  border-radius: 14px;
  background: linear-gradient(#ffe066, #f1c40f);
  border: none;
  cursor: pointer;
}

</style>