<template>
  <div class="w-full h-screen">
    <div
        ref="gridWrapRef"
        class="overflow-auto border border-gray-300 bg-white w-full h-[calc(100vh-100px)]"
    >
      <div
          class="grid"
          :style="{
          gridTemplateColumns: 'repeat(100, 50px)',
          width: '5000px',
          height: '5000px',
        }"
      >
        <div
            v-for="idx in 10000"
            :key="idx"
            class="w-[50px] h-[50px] border border-gray-200 hover:bg-sky-100 cursor-pointer transition-colors"
            @mouseenter="handleCellHover($event, idx)"
            @mousemove="handleMouseMove($event, idx)"
            @mouseleave="hideTooltip"
        />
      </div>
    </div>

    <div
        ref="tooltipRef"
        v-show="tooltipVisible"
        class="fixed z-[9999] pointer-events-none bg-white text-[#303133] px-3 py-2 rounded text-[13px] leading-[1.5] max-w-[280px] whitespace-pre shadow-[0_2px_8px_rgba(0,0,0,0.25)]"
        :style="{ left: tooltipPos.x + 'px', top: tooltipPos.y + 'px' }"
    >
      <p>Variable X: {{ tooltipPos.x }}</p>
      <p>Variable Y: {{ tooltipPos.y }}</p>
      <p>Score: {{ tooltipPos.x * tooltipPos.y }}</p>
    </div>
  </div>
</template>

<script setup>
import {ref, onBeforeUnmount, useTemplateRef} from 'vue'

const gridWrapRef = useTemplateRef("gridWrapRef")
const tooltipRef = useTemplateRef("tooltipRef")

// 全局Tooltip状态
const tooltipVisible = ref(false)
const tooltipPos = ref({ x: 0, y: 0 })

// tooltip 尺寸缓存，避免每次移动都强制布局
let cachedWidth = 0
let cachedHeight = 0
let sizeMeasured = false

const TOOLTIP_OFFSET = 12 // 与鼠标之间的间距
const TOOLTIP_MARGIN = 8 // 与视口边缘的安全距离

/**
 * 计算tooltip实际宽高（只测量一次）
 */
const measureTooltip = () => {
  if (sizeMeasured) return
  const el = tooltipRef.value
  if (el && el.offsetWidth > 0 && el.offsetHeight > 0) {
    cachedWidth = el.offsetWidth
    cachedHeight = el.offsetHeight
    sizeMeasured = true
  } else {
    // 兜底默认值（与 CSS 设计一致）
    cachedWidth = 220
    cachedHeight = 56
    sizeMeasured = true
  }
}

/**
 * 计算tooltip坐标（跟随鼠标右侧，且不超出视口边界）
 * @param {MouseEvent} e
 */
const computeTooltipPosition = (e) => {
  const vw = window.innerWidth
  const vh = window.innerHeight

  // 先假设放在鼠标右侧
  let left = e.clientX + TOOLTIP_OFFSET
  let top = e.clientY

  // 在计算前读取一次实际尺寸
  measureTooltip()

  // 水平方向：若右侧放不下，则放到鼠标左侧
  if (left + cachedWidth + TOOLTIP_MARGIN > vw) {
    left = e.clientX - cachedWidth - TOOLTIP_OFFSET
    // 若左侧也放不下，则贴右边
    if (left < TOOLTIP_MARGIN) {
      left = Math.max(TOOLTIP_MARGIN, vw - cachedWidth - TOOLTIP_MARGIN)
    }
  }

  // 垂直方向：不超出视口上下边界
  if (top + cachedHeight + TOOLTIP_MARGIN > vh) {
    top = Math.max(TOOLTIP_MARGIN, vh - cachedHeight - TOOLTIP_MARGIN)
  }
  if (top < TOOLTIP_MARGIN) {
    top = TOOLTIP_MARGIN
  }

  tooltipPos.value = { x: left, y: top }
}

/**
 * 计算格子坐标
 * idx: 1~10000
 * x: 列 1~100
 * y: 行 1~100
 */
const handleCellHover = (e, idx) => {
  tooltipVisible.value = true
  // 下一帧再测量，保证 tooltip DOM 已插入
  requestAnimationFrame(() => {
    sizeMeasured = false
    computeTooltipPosition(e)
  })
}

const handleMouseMove = (e) => {
  if (!tooltipVisible.value) return
  computeTooltipPosition(e)
}

const hideTooltip = () => {
  tooltipVisible.value = false
}

// 窗口尺寸变化时，若 tooltip 仍显示则重新定位
const handleResize = () => {
  if (!tooltipVisible.value) return
  // 使用当前鼠标位置难以精确获取，退而求其次：以当前存储的位置做边界校正
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
  tooltipPos.value = { x: left, y: top }
}

if (typeof window !== 'undefined') {
  window.addEventListener('resize', handleResize)
}

onBeforeUnmount(() => {
  if (typeof window !== 'undefined') {
    window.removeEventListener('resize', handleResize)
  }
})
</script>

<style scoped></style>