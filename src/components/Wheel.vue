<template>
  <div
    class="wheel-container"
    :style="{ width: size + 'px', height: size + 'px' }"
  >
    <!-- BORDER IMAGE -->
    <img
      src="/img/border.webp"
      class="wheel-border"
      :style="{
        transform: `translate(-50%, -50%) rotate(${borderRotation}rad)`
      }"
      alt="wheel border"
    />

    <div v-if="props.position === 'top'" class="wheel-pointer top">
      <img src="/img/arrow-1.webp" />
    </div>

    <canvas
      ref="canvasRef"
      :width="size"
      :height="size"
      class="wheel-canvas"
    />

    <!-- VISUAL BUTTON ONLY -->
    <img
      v-if="props.position === 'center'"
      src="/img/btn-center.webp"
      class="wheel-button"
      loading="lazy"
    />

    <div v-if="props.position === 'bottom'" class="wheel-pointer bottom">
      <img src="/img/arrow-2.webp" />
    </div>
  </div>
</template>


<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import gsap from 'gsap'
import { useOrientation } from '../service/orientation'
const { IS_LANDSCAPE } = useOrientation()

interface Prize {
  id: number
  label: string
  color: string
  weight: number
}

const borderRotation = ref(0)

const props = defineProps<{
  prizes: Prize[]
  disabled?: boolean
  position: string
}>()

const emit = defineEmits<{
  (e: 'finished', prize: Prize): void
}>()


const canvasRef = ref<HTMLCanvasElement | null>(null)
const isSpinning = ref(false)

const size = ref(350)
const radius = ref(size.value / 2)

let ctx: CanvasRenderingContext2D
let rotation = 0

const totalWeight = props.prizes.reduce((a, b) => a + b.weight, 0)


watch(
  IS_LANDSCAPE,
  (isLandscape) => {
    size.value = isLandscape ? 350 : 280
    radius.value = size.value / 2

    // redraw only if canvas is ready
    if (ctx) {
      ctx.clearRect(0, 0, size.value, size.value)
      drawWheel()
    }
  },
  { immediate: true }
)

function lightenColor(hex: string, percent: number) {
  const num = parseInt(hex.replace('#', ''), 16)
  const r = Math.min(255, (num >> 16) + 255 * percent)
  const g = Math.min(255, ((num >> 8) & 0xff) + 255 * percent)
  const b = Math.min(255, (num & 0xff) + 255 * percent)
  return `rgb(${r}, ${g}, ${b})`
}

function darkenColor(hex: string, percent: number) {
  const num = parseInt(hex.replace('#', ''), 16)
  const r = Math.max(0, (num >> 16) - 255 * percent)
  const g = Math.max(0, ((num >> 8) & 0xff) - 255 * percent)
  const b = Math.max(0, (num & 0xff) - 255 * percent)
  return `rgb(${r}, ${g}, ${b})`
}


function drawWheel() {
  ctx.clearRect(0, 0, size.value, size.value)
  let startAngle = 0

  props.prizes.forEach((p) => {
    const sliceAngle = (p.weight / totalWeight) * Math.PI * 2

    ctx.beginPath()
    ctx.moveTo(radius.value, radius.value)
    ctx.arc(radius.value, radius.value, radius.value, startAngle, startAngle + sliceAngle)
    ctx.closePath()

    // 🔥 DARK CENTER → LIGHT EDGE GRADIENT
    const gradient = ctx.createRadialGradient(
      radius.value, radius.value, 0,
      radius.value, radius.value, radius.value
    )

    gradient.addColorStop(0, darkenColor(p.color, 0.35))  // darker center
    gradient.addColorStop(0.7, p.color)
    gradient.addColorStop(1, lightenColor(p.color, 0.2)) // lighter edge

    ctx.fillStyle = gradient
    ctx.fill()

    // 🖤 DARK SLICE BORDER
    ctx.strokeStyle = props.position === 'center' ? '#011802' : props.position === 'bottom' ? '#166FAC' : '#970C0A'
    ctx.lineWidth = 6
    ctx.stroke()

    const TEXT_OFFSET_X = -12 // ← adjust this value
    // TEXT (UNCHANGED)
    ctx.save()
    ctx.translate(radius.value, radius.value)
    ctx.rotate(startAngle + sliceAngle / 2)
    ctx.fillStyle = '#fff'
    ctx.font = 'bold 30px sans-serif'
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'
    ctx.fillText(
  p.label,
  radius.value * 0.65 + TEXT_OFFSET_X,
  0
)
    ctx.restore()

    startAngle += sliceAngle
  })
}


function getPrizeFromRotation(): Prize {
  const normalized =
    (Math.PI * 2 - (rotation % (Math.PI * 2)) + Math.PI / 2) %
    (Math.PI * 2)

  let acc = 0
  for (const p of props.prizes) {
    acc += (p.weight / totalWeight) * Math.PI * 2
    if (normalized <= acc) return p
  }
  return props.prizes[0]
}

function spin() {
  if (isSpinning.value || props.disabled) return

  isSpinning.value = true

  const spins = 6
  const anglePerSlice = (Math.PI * 2) / props.prizes.length
  const randomIndex = Math.floor(Math.random() * props.prizes.length)

  const targetRotation =
    rotation +
    spins * Math.PI * 2 +
    randomIndex * anglePerSlice +
    anglePerSlice / 2 -
    Math.PI / 2

  const tween = { r: rotation }

  gsap.to(tween, {
    r: targetRotation,
    duration: 4,
    ease: 'power4.out',
    onUpdate: () => {
      rotation = tween.r
        borderRotation.value = rotation
      ctx.save()
      ctx.clearRect(0, 0, size.value, size.value)
      ctx.translate(radius.value, radius.value)
      ctx.rotate(rotation)
      ctx.translate(-radius.value, -radius.value)
      drawWheel()
      ctx.restore()
    },
    onComplete: () => {
      isSpinning.value = false
      emit('finished', getPrizeFromRotation())
    },
  })
}

defineExpose({ spin })

onMounted(() => {
  ctx = canvasRef.value!.getContext('2d')!
  drawWheel()
})
</script>

<style scoped>

  @import url('https://fonts.googleapis.com/css2?family=Lilita+One&display=swap');

  .wheel-border {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 110%;
  height: 110%;
  transform: translate(-50%, -50%);
  pointer-events: none; /* IMPORTANT */
  z-index: 5;
}

.wheel-container {
  position: relative;
  /* width: 350px;
  height: 350px; */
}

.wheel-canvas {
  border-radius: 50%;
}

.wheel-button {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 150px;
  height: 150px;
  pointer-events: none;
}

.wheel-pointer {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  z-index: 5;
  pointer-events: none;
}

.wheel-pointer img {
  width: 48px;          /* adjust size */
  height: auto;
  display: block;
}

/* TOP */
.wheel-pointer.top {
  top: -8%;
  transform: translateX(-50%);
}

/* BOTTOM (INVERSE TRIANGLE) */
.wheel-pointer.bottom {
  bottom: -8%;
  transform: translateX(-50%) rotate(180deg);
}

</style>
