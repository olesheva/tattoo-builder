<template>
  <header
    class="flex gap-3 justify-between items-center p-3 fixed bottom-0 sm:top-0 sm:bottom-auto right-0 z-10"
  >
    <h1 class="font-bold text-xl header hidden sm:block">Olesheva Tattoo Builder</h1>
    <div class="flex flex-row gap-3 sm:gap-2 flex-wrap justify-end">
      <BaseButton look="secondary" href="https://www.instagram.com/oleshevatattoo">
        <div class="flex gap-2 items-center">
          <div>Book a tattoo session</div>
          <img class="h-4" src="@/assets/ig-logo.png" />
        </div>
      </BaseButton>
      <div class="flex items-center gap-3 sm:gap-2">
        <BaseButton @click="onSave">Download</BaseButton>
        <BaseButton @click="onReset">Reset</BaseButton>
      </div>
    </div>
  </header>
  <div v-if="!isMenuOpen" class="fixed left-3 z-10 translate-y-1/2 bottom-1/2">
    <BaseButton
      look="secondary"
      class="h-32 w-8 flex items-center justify-center"
      @click="isMenuOpen = true"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="9"
        height="16"
        viewBox="0 0 9 16"
        fill="none"
        class="text-gray-400"
      >
        <path
          fill-rule="evenodd"
          clip-rule="evenodd"
          d="M0.707077 15.9961L8.69926 8.00395L7.99216 7.29684L-2.97672e-05 15.289L0.707077 15.9961Z"
          fill="currentColor"
        />
        <path
          fill-rule="evenodd"
          clip-rule="evenodd"
          d="M8.69929 7.99219L0.707107 7.7486e-07L0 0.707108L7.99219 8.69929L8.69929 7.99219Z"
          fill="currentColor"
        />
      </svg>
    </BaseButton>
  </div>
  <div class="flex relative">
    <div
      v-show="isMenuOpen"
      class="overflow-auto h-screen absolute offset-0 z-10 border border-r w-[200px] bg-white/90"
    >
      <div class="flex flex-wrap gap-2 p-2">
        <template v-for="category in categories" :key="category">
          <BaseCheckbox
            v-model="filter"
            :value="category"
            :name="category"
            :label="category"
            class="border px-2 rounded-sm py-0.5"
          />
        </template>
      </div>
      <template v-for="image in filteredImages" :key="image.key">
        <img
          draggable
          :src="`${base || ''}/tattoo/${image.file}`"
          @dragstart.passive="handleDragStart"
          @dragend.passive="handleDragEnd"
          @click="onImageClick"
        />
      </template>
    </div>
    <div
      ref="canvasContainer"
      class="canvas-container"
      @dragover.prevent="handleDragOver"
      @drop="handleDrop"
      @touchstart="onCanvasClick"
      @click="onCanvasClick"
    >
      <canvas ref="canvasElement" class="w-full"></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, computed } from 'vue'
import * as fabric from 'fabric'
import BaseButton from '@/components/BaseButton.vue'
import BaseCheckbox from '@/components/BaseCheckbox.vue'
import { throttle } from 'lodash'

const canvasElement = ref<null | HTMLCanvasElement>(null)
const canvasContainer = ref<null | HTMLDivElement>(null)
const isMenuOpen = ref(true)
const filter = ref<string[]>([])
let canvas: undefined | fabric.Canvas

const images = ref<ConfigImage[]>([])
const base = import.meta.env.VITE_BASE
const isDev = import.meta.env.MODE === 'development'

console.log('constructor: is dev mode', isDev)

const filteredImages = computed(() => {
  if (!filter.value.length) {
    return images.value
  }
  return images.value.filter((image) => filter.value.includes(image.category))
})

interface ConfigImage {
  file: string
  category: string
  key?: symbol
}

async function loadConfig() {
  const configRequest = await fetch(`config${isDev ? '.dev' : ''}.json`)
  const config = await configRequest.json()

  config.forEach((image: ConfigImage) => images.value.push({ ...image, key: Symbol('image-key') }))
  images.value.concat(config)
}

const categories = computed(() => {
  return [...new Set(images.value.map((image) => image.category))]
})

loadConfig()
let imgOffsetX: undefined | number
let imgOffsetY: undefined | number
let draggingElement = null as null | HTMLImageElement
function onCanvasClick() {
  isMenuOpen.value = false
}
function onSave() {
  if (canvas) {
    const data = canvas?.toDataURL({
      format: 'jpeg',
      quality: 90,
      multiplier: 2
    })

    const link = document.createElement('a')
    link.download = 'tattoo.jpeg'
    link.href = data
    link.click()
  }
}

function onImageClick(e: MouseEvent) {
  if ('ontouchstart' in window) {
    const imageElement = e.target as HTMLImageElement
    var newImage = new fabric.Image(imageElement, {
      scaleX: imageElement.width / imageElement.naturalWidth,
      scaleY: imageElement.height / imageElement.naturalHeight,
      width: imageElement.naturalWidth,
      height: imageElement.naturalHeight,
      left: window.innerWidth / 2 - imageElement.width / 2,
      top: window.innerHeight / 2 - imageElement.height / 2
    })
    canvas?.add(newImage)
    isMenuOpen.value = false
  }
}

function handleDragStart(e: DragEvent) {
  draggingElement = e.target as HTMLImageElement

  const rect = (e.target as HTMLDivElement)?.getBoundingClientRect()

  imgOffsetX = e.clientX - (rect.left + window.scrollX)
  imgOffsetY = e.clientY - (rect.top + window.scrollY)
}

function handleDragOver(e: DragEvent) {
  if (e.dataTransfer) {
    e.dataTransfer.dropEffect = 'copy'
  }
  return false
}

function handleDragEnd() {
  draggingElement = null
}

function handleDrop(e: DragEvent) {
  e = e || window.event
  if (e.preventDefault) {
    e.preventDefault()
  }
  if (e.stopPropagation) {
    e.stopPropagation()
  }

  if (!canvasElement.value || !draggingElement) {
    return
  }
  const rect = canvasElement.value.getBoundingClientRect()

  if (imgOffsetY === undefined || imgOffsetX === undefined) {
    return
  }

  const x = e.clientX - (rect.left + window.scrollX + imgOffsetX)
  const y = e.clientY - (rect.top + window.scrollY + imgOffsetY)

  var newImage = new fabric.Image(draggingElement, {
    scaleX: draggingElement.width / draggingElement.naturalWidth,
    scaleY: draggingElement.height / draggingElement.naturalHeight,
    width: draggingElement.naturalWidth,
    height: draggingElement.naturalHeight,
    left: x,
    top: y
  })
  canvas?.add(newImage)

  isMenuOpen.value = false
  return false
}

const storeData = throttle(() => {
  console.log('constructor: store data')
  localStorage.setItem('canvas', JSON.stringify(canvas?.toJSON()))
}, 300, { leading: false, trailing: true })

function onReset() {
  localStorage.removeItem('canvas')
  if (canvas) {
    canvas.clear()
    canvas.backgroundColor = '#fff'
  }
}

function initCanvas() {
  if (!canvasElement.value || !canvasContainer.value) {
    return
  }
  canvas = new fabric.Canvas(canvasElement.value)
  canvas.backgroundColor = '#fff'

  function updateCanvasSize() {
    canvas?.setDimensions({
      width: window.innerWidth,
      height: window.innerHeight
    })
  }

  canvas.on('after:render', storeData)
  window.addEventListener('resize', () => updateCanvasSize())
  const storedCanvas = localStorage.getItem('canvas')

  if (storedCanvas) {
    const parsedCanvas = JSON.parse(storedCanvas)

    // // restore background on broken saves
    // if (!parsedCanvas.background) {
    //   parsedCanvas.background = '#fff'
    // }
    canvas?.loadFromJSON(parsedCanvas, () => {
      if (canvas) {
        canvas.requestRenderAll()
      }
    })
  }

  updateCanvasSize()

  const deleteIcon =
    "data:image/svg+xml,%3Csvg width='16' height='16' viewBox='0 0 16 16' fill='none' xmlns='http://www.w3.org/2000/svg'%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M15.289 15.9961L-1.44839e-05 0.707107L0.707092 0L15.9961 15.289L15.289 15.9961Z' fill='%23B2CCFF'/%3E%3Cpath fill-rule='evenodd' clip-rule='evenodd' d='M0.707107 15.9961L15.9961 0.707107L15.289 0L5.23321e-07 15.289L0.707107 15.9961Z' fill='%23B2CCFF'/%3E%3C/svg%3E%0A"

  const img = document.createElement('img')
  img.src = deleteIcon

  fabric.Object.ownDefaults.controls = {
    ...fabric.controlsUtils.createObjectDefaultControls(),
    // ...fabric.controlsUtils.createResizeControls(),
    deleteControl: new fabric.Control({
      x: 0.5,
      y: -0.5,
      offsetX: 24,
      cursorStyle: 'pointer',
      mouseUpHandler: (eventData, transform) => {
        const target = transform.target
        const canvas = target.canvas
        canvas?.remove(target)
        canvas?.requestRenderAll()
      },
      render: (ctx, left, top, styleOverride, fabricObject) => {
        const size = 16
        ctx.save()
        ctx.translate(left, top)
        ctx.rotate(fabric.util.degreesToRadians(fabricObject.angle))
        ctx.drawImage(img, -size / 2, -size / 2, size, size)
        ctx.restore()
      }
    })
  }
}

onMounted(() => {
  initCanvas()
})
</script>

<style>
.header {
  color: #b2ccff;
}

[draggable] {
  -moz-user-select: none;
  -khtml-user-select: none;
  -webkit-user-select: none;
  user-select: none;
  -khtml-user-drag: element;
  -webkit-user-drag: element;
  cursor: move;
}
</style>
