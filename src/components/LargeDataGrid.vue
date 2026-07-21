<template>
  <div class="w-full h-screen bg-gray-100 p-4">
    <div class="grid grid-cols-[100px_1fr] grid-rows-[60px_1fr] h-full">
      <!-- 左上角合并单元格 -->
      <div class="bg-gray-500 text-white font-bold flex items-center justify-center border border-gray-300">
        行/列
      </div>

      <!-- 上方列标题 -->
      <div
          ref="colHeaderRef"
          class="overflow-hidden border border-gray-300"
      >
        <div
            class="flex"
            :style="{ width: totalWidth + 'px' }"
        >
          <div
              v-for="col in 100"
              :key="'col-' + col"
              class="w-[100px] h-[60px] bg-gray-400 text-white font-medium flex items-center justify-center border border-gray-300"
          >
            Col {{ col }}
          </div>
        </div>
      </div>

      <!-- 左侧行标题 -->
      <div
          ref="rowHeaderRef"
          class="overflow-hidden border border-gray-300"
      >
        <div
            :style="{ height: totalHeight + 'px' }"
        >
          <div
              v-for="row in 100"
              :key="'row-' + row"
              class="w-[100px] h-[100px] bg-gray-400 text-white font-medium flex items-center justify-center border border-gray-300"
          >
            Row {{ row }}
          </div>
        </div>
      </div>

      <!-- 主网格区域 -->
      <div
          ref="gridWrapRef"
          class="overflow-auto border border-gray-300 bg-white"
          @scroll="handleGridScroll"
      >
        <div
            class="grid"
            :style="{
          gridTemplateColumns: 'repeat(100, 100px)',
          width: totalWidth + 'px',
          height: totalHeight + 'px',
        }"
        >
          <div
              v-for="idx in 10000"
              :key="idx"
              class="w-full h-[100px] border border-gray-200 hover:bg-sky-100 cursor-pointer transition-colors flex items-center justify-center"
              :class="{ 'bg-sky-300': highlightedIdx === idx }"
              @mouseenter="handleCellHover($event, idx)"
              @mousemove="handleMouseMove($event, idx)"
              @mouseleave="hideTooltip"
              @click.stop="handleCellClick(idx)"
          >
            {{ Math.floor((idx - 1) / 100) + 1 }}-{{ ((idx - 1) % 100) + 1 }}
          </div>
        </div>
      </div>
    </div>

    <div
        ref="tooltipRef"
        v-show="tooltipVisible"
        class="fixed z-[9999] pointer-events-none bg-white text-[#303133] px-3 py-2 rounded text-[13px] leading-[1.5] max-w-[280px] whitespace-pre shadow-[0_2px_8px_rgba(0,0,0,0.25)]"
        :style="{ left: tooltipPos.x + 'px', top: tooltipPos.y + 'px' }"
    >
      <p>Variable Y: {{ tooltipPos.row }}</p>
      <p>Variable X: {{ tooltipPos.column }}</p>
    </div>
  </div>
</template>

<script setup>
import {ref, onBeforeUnmount, useTemplateRef, nextTick, onMounted} from 'vue'

const tooltipRef = useTemplateRef("tooltipRef")
const gridWrapRef = useTemplateRef("gridWrapRef")
const colHeaderRef = useTemplateRef("colHeaderRef")
const rowHeaderRef = useTemplateRef("rowHeaderRef")

const tooltipVisible = ref(false)
const tooltipPos = ref({x: 0, y: 0, row: 0, column: 0})
const highlightedIdx = ref(null)

const totalWidth = 100 * 100
const totalHeight = 100 * 100

let cachedWidth = 0
let cachedHeight = 0
let sizeMeasured = false

const TOOLTIP_OFFSET = 12
const TOOLTIP_MARGIN = 8

const measureTooltip = () => {
  if (sizeMeasured) return
  const el = tooltipRef.value
  if (el && el.offsetWidth > 0 && el.offsetHeight > 0) {
    cachedWidth = el.offsetWidth
    cachedHeight = el.offsetHeight
    sizeMeasured = true
  } else {
    cachedWidth = 220
    cachedHeight = 56
    sizeMeasured = true
  }
}

const computeTooltipPosition = async (e, idx) => {
  const vw = window.innerWidth
  const vh = window.innerHeight

  let left = e.clientX + TOOLTIP_OFFSET
  let top = e.clientY

  measureTooltip()

  if (left + cachedWidth + TOOLTIP_MARGIN > vw) {
    left = e.clientX - cachedWidth - TOOLTIP_OFFSET
    if (left < TOOLTIP_MARGIN) {
      left = Math.max(TOOLTIP_MARGIN, vw - cachedWidth - TOOLTIP_MARGIN)
    }
  }

  if (top + cachedHeight + TOOLTIP_MARGIN > vh) {
    top = Math.max(TOOLTIP_MARGIN, vh - cachedHeight - TOOLTIP_MARGIN)
  }
  if (top < TOOLTIP_MARGIN) {
    top = TOOLTIP_MARGIN
  }

  tooltipPos.value = {x: left, y: top, row: Math.floor((idx - 1) / 100) + 1, column: ((idx - 1) % 100) + 1}
}

const handleCellHover = async (e, idx) => {
  tooltipVisible.value = true
  requestAnimationFrame(() => {
    sizeMeasured = false
    computeTooltipPosition(e, idx)
  })
}

const handleMouseMove = (e, idx) => {
  if (!tooltipVisible.value) return
  computeTooltipPosition(e, idx)
}

const hideTooltip = () => {
  tooltipVisible.value = false
}

const handleCellClick = (idx) => {
  highlightedIdx.value = idx
}

const handleDocumentClick = (e) => {
  const gridWrap = gridWrapRef.value
  if (gridWrap && !gridWrap.contains(e.target)) {
    highlightedIdx.value = null
  }
}

const handleGridScroll = () => {
  const gridWrap = gridWrapRef.value
  const colHeader = colHeaderRef.value
  const rowHeader = rowHeaderRef.value

  if (colHeader) {
    colHeader.scrollLeft = gridWrap.scrollLeft
  }
  if (rowHeader) {
    rowHeader.scrollTop = gridWrap.scrollTop
  }
}

const handleResize = () => {
  if (!tooltipVisible.value) return
  const x = tooltipPos.value.x
  const y = tooltipPos.value.y
  measureTooltip()
  const vw = window.innerWidth
  const vh = window.innerHeight
  let left = x
  let top = y
  if (left + cachedWidth + TOOLTIP_MARGIN > vw) {
    left = Math.max(TOOLTIP_MARGIN, vw - cachedWidth - TOOLTIP_MARGIN)
  }
  if (left < TOOLTIP_MARGIN) left = TOOLTIP_MARGIN
  if (top + cachedHeight + TOOLTIP_MARGIN > vh) {
    top = Math.max(TOOLTIP_MARGIN, vh - cachedHeight - TOOLTIP_MARGIN)
  }
  if (top < TOOLTIP_MARGIN) top = TOOLTIP_MARGIN
  tooltipPos.value = {x: left, y: top}
}

onMounted(() => {
  document.addEventListener('click', handleDocumentClick)
})

onBeforeUnmount(() => {
  document.removeEventListener('click', handleDocumentClick)
  if (typeof window !== 'undefined') {
    window.removeEventListener('resize', handleResize)
  }
})

if (typeof window !== 'undefined') {
  window.addEventListener('resize', handleResize)
}
</script>

<style scoped></style>
