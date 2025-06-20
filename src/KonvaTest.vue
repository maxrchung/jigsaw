<script setup lang="ts">
interface Props {
  title?: string
}

interface Piece extends Konva.ImageConfig {
  pieceX: number
  pieceY: number
}

const { title } = defineProps<Props>()

import Konva from 'konva'
import type { IRect, KonvaNodeEvent } from 'konva/lib/types'
import { ref, onMounted, watchEffect } from 'vue'
import { useImage } from 'vue-konva'

const stageSize = {
  width: window.innerWidth,
  height: window.innerHeight,
}

const list = ref<Konva.ShapeConfig[]>([])
const dragItemId = ref<string | null>(null)

const [image] = useImage('/sonic-disturb.jpg')
const pieces = ref<Piece[]>([])

watchEffect(() => {
  console.log(image)
  console.log(image.value?.width)

  const width = image.value?.width
  const height = image.value?.height

  if (!image.value || !width || !height) {
    return
  }

  const isWidthLarger = width >= height
  const ratio = width / height

  const imageWidth = 300
  const pieceWidth = isWidthLarger ? imageWidth : imageWidth * ratio
  const pieceHeight = isWidthLarger ? imageWidth / ratio : imageWidth

  pieces.value.push({
    id: '0',
    image: image.value,
    crop: {
      height: height / 2,
      width: width / 2,
      x: 0,
      y: 0,
    },
    draggable: true,
    width: pieceWidth,
    height: pieceHeight,
    pieceX: 0,
    pieceY: 0,
  })

  pieces.value.push({
    id: '1',
    image: image.value,
    crop: {
      height: height / 2,
      width: width / 2,
      x: width / 2,
      y: 0,
    },
    draggable: true,
    width: pieceWidth,
    height: pieceHeight,
    pieceX: 1,
    pieceY: 0,
  })

  pieces.value.push({
    id: '2',
    image: image.value,
    crop: {
      height: height / 2,
      width: width / 2,
      x: 0,
      y: height / 2,
    },
    draggable: true,
    width: pieceWidth,
    height: pieceHeight,
    pieceX: 0,
    pieceY: 1,
  })

  pieces.value.push({
    id: '3',
    image: image.value,
    crop: {
      height: height / 2,
      width: width / 2,
      x: width / 2,
      y: height / 2,
    },
    draggable: true,
    width: pieceWidth,
    height: pieceHeight,
    pieceX: 1,
    pieceY: 1,
  })
})

const handleDragStart = (e: Konva.KonvaEventObject<KonvaNodeEvent.dragstart>) => {
  // save drag element:
  dragItemId.value = e.target.id()
  // move current element to the top:
  const item = list.value.find((i) => i.id === dragItemId.value)

  if (!item) {
    return
  }

  const index = list.value.indexOf(item)
  list.value.splice(index, 1)
  list.value.push(item)
}

const handleDragEnd = () => {
  dragItemId.value = null
}

const hasIntersection = (a: IRect, b: IRect) => {
  if (!a.x || !a.y || !a.width || !a.height || !b.x || !b.y || !b.width || !b.height) {
    return
  }

  return !(
    b.x > a.x + a.width ||
    b.x + b.width < a.x ||
    b.y > a.y + a.height ||
    b.y + b.height < a.y
  )
}

const handleDragMove = (e: Konva.KonvaEventObject<KonvaNodeEvent.dragmove>, current: Piece) => {
  const target = e.target
  const layer = target.getLayer()

  if (!layer) {
    return
  }

  const currentRect = target.getClientRect()

  const p1x = current.pieceX
  const p1y = current.pieceY

  for (const piece of pieces.value) {
    if (piece.id === current.id) {
      continue
    }

    const p2x = piece.pieceX
    const p2y = piece.pieceY
    const isAdjacentX = Math.abs(p1x - p2x) < 1
    const isAdjacentY = Math.abs(p1y - p2y) < 1

    // Only do collision test if pieces are directly adjacent
    if (!isAdjacentX && !isAdjacentY) continue

    const other = layer?.findOne(`#${piece.id}`)

    if (!other) {
      continue
    }

    const otherRect = other?.getClientRect()

    if (hasIntersection(currentRect, otherRect)) {
      console.log(`${other?.getAttr('pieceX')}, ${other?.getAttr('pieceY')}`)
      console.log('intersect')
    }
  }
}

onMounted(() => {
  for (let n = 0; n < 30; n++) {
    list.value.push({
      id: Math.round(Math.random() * 10000).toString(),
      x: Math.random() * stageSize.width,
      y: Math.random() * stageSize.height,
      rotation: Math.random() * 180,
      scaleX: Math.random(),
      scaleY: Math.random(),
    })
  }
})
</script>

<template>
  <v-stage
    ref="stage"
    :config="stageSize"
    @dragstart="handleDragStart"
    @dragend="handleDragEnd"
    draggable="true"
  >
    <v-layer ref="layer">
      <v-image
        v-for="piece in pieces"
        :key="piece.id"
        :config="piece"
        @dragmove="(e: Konva.KonvaEventObject<KonvaNodeEvent.dragmove>) => handleDragMove(e, piece)"
      />

      <v-star
        v-for="item in list"
        :key="item.id"
        :config="{
          x: item.x,
          y: item.y,
          rotation: item.rotation,
          id: item.id,
          numPoints: 5,
          innerRadius: 30,
          outerRadius: 50,
          fill: '#89b717',
          opacity: 0.8,
          draggable: true,
          scaleX: dragItemId === item.id ? (item.scale ?? 1 * 1.2) : item.scale,
          scaleY: dragItemId === item.id ? (item.scale ?? 1 * 1.2) : item.scale,
          shadowColor: 'black',
          shadowBlur: 10,
          shadowOffsetX: dragItemId === item.id ? 15 : 5,
          shadowOffsetY: dragItemId === item.id ? 15 : 5,
          shadowOpacity: 0.6,
        }"
      />
    </v-layer>
  </v-stage>
</template>
