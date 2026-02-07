<template>
  <div class="container" :class="{ 'party-mode': partyMode }" :style="gridStyle">
    <div class="cell" v-for="i in totalCells" :key="i" :class="{ rotated: rotatedCells.has(i), special: i === specialCell, party: i === partyCell }" @click="onCellClick(i)">
      <img :src="shuffled[i - 1]" alt="flower" />
    </div>
  </div>

  <dialog ref="dialogRef" class="flower-dialog" @click="closeDialog">
    <div>
      Will you be my valentine?
    </div>
  </dialog>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'

import flower1 from '@/assets/images/flower_1.png'
import flower2 from '@/assets/images/flower_2.png'
import flower3 from '@/assets/images/flower_3.png'
import flower4 from '@/assets/images/flower_4.png'
import flower5 from '@/assets/images/flower_5.png'
import flower6 from '@/assets/images/flower_6.png'
import partyMusic from '@/assets/music/partyonwebbi.wav'

const images = [flower1, flower2, flower3, flower4, flower5, flower6]
const audio = new Audio(partyMusic)
audio.addEventListener('timeupdate', () => {
  if (audio.duration - audio.currentTime <= 0.3) {
    audio.currentTime = 0.95
  }
})

const width = ref(window.innerWidth)
const height = ref(window.innerHeight)

function onResize() {
  width.value = window.innerWidth
  height.value = window.innerHeight
}

onMounted(() => window.addEventListener('resize', onResize))
onUnmounted(() => window.removeEventListener('resize', onResize))

const isMobile = computed(() => width.value < 768)
const targetSize = computed(() => isMobile.value ? 75 : 150)
const minSize = computed(() => isMobile.value ? 60 : 120)
const maxSize = computed(() => isMobile.value ? 90 : 180)
const gap = 20

const rotatedCells = ref(new Set<number>())
const partyMode = ref(false)

const dialogRef = ref<HTMLDialogElement>()

function toggleRotation(i: number) {
  if (rotatedCells.value.has(i)) {
    rotatedCells.value.delete(i)
  } else {
    rotatedCells.value.add(i)
  }
  rotatedCells.value = new Set(rotatedCells.value)
}

function onCellClick(i: number) {
  if (i === specialCell.value) {
    dialogRef.value?.showModal()
  } else if (i === partyCell.value) {
    partyMode.value = !partyMode.value
    if (partyMode.value) {
      audio.currentTime = 0.95
      audio.play()
    } else {
      audio.pause()
    }
  } else {
    toggleRotation(i)
  }
}

function closeDialog() {
  dialogRef.value?.close()
}

const cols = computed(() => {
  let c = Math.round(width.value / targetSize.value)
  const size = (width.value - (c - 1) * gap) / c
  if (size < minSize.value) c--
  if (size > maxSize.value) c++
  return c
})

const cellSize = computed(() => (width.value - (cols.value - 1) * gap) / cols.value)
const rows = computed(() => Math.ceil((height.value + gap) / (cellSize.value + gap)))
const totalCells = computed(() => cols.value * rows.value)
const specialCell = computed(() => Math.floor(Math.random() * totalCells.value) + 1)
const partyCell = computed(() => {
  let cell
  do {
    cell = Math.floor(Math.random() * totalCells.value) + 1
  } while (cell === specialCell.value)
  return cell
})

const shuffled = computed(() => {
  return Array.from({ length: totalCells.value }, () =>
    images[Math.floor(Math.random() * images.length)]
  )
})

const gridStyle = computed(() => ({
  gridTemplateColumns: `repeat(${cols.value}, ${cellSize.value}px)`,
  gridTemplateRows: `repeat(${rows.value}, ${cellSize.value}px)`,
  gap: `${gap}px`,
}))
</script>

<style scoped lang="scss">
.container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: grid;

  &.party-mode {
    animation: strobe 0.3s infinite;
  }
}

@keyframes strobe {
  0% { background-color: black; }
  50% { background-color: white; }
  100% { background-color: black; }
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.cell {
  cursor: url('@/assets/cursors/hover_32.png'), pointer;

  &.special {
    cursor: url('@/assets/cursors/heart_32.png'), pointer;
  }

  &.party {
    cursor: url('@/assets/cursors/disco_32.png'), pointer;
  }

  img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    transition: transform 1.5s ease;
  }

  &:hover img {
    transform: rotate(360deg);
  }

  &.rotated img {
    transform: rotate(360deg);
  }

  .party-mode & img {
    animation: spin 1s linear infinite;
  }
}

.flower-dialog {
  border: 1px solid rgba(255, 255, 255, 0.3);
  height: 300px;
  width: 500px;
  border-radius: 24px;
  padding: 3rem 4rem;
  font-family: 'Poppins', sans-serif;
  font-weight: 300;
  font-size: 1.5rem;
  letter-spacing: 0.02em;
  text-align: center;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  box-shadow:
    0 8px 32px rgba(0, 0, 0, 0.2),
    inset 0 1px 0 rgba(255, 255, 255, 0.4);
  outline: none;
}
</style>
