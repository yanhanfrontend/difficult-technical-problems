<template>
  <div class="flex flex-col h-full">
    <div ref="boxRef" class="h-[500px] border border-gray-300 overflow-auto relative">
      <canvas ref="canvasRef"></canvas>
      <div
          ref="tipDom"
          class="absolute z-50 px-2 py-1 text-xs text-white bg-gray-800 rounded pointer-events-none"
          style="display:none;"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const canvasRef = ref(null)
const boxRef = ref(null)
const tipDom = ref(null)

const cellSize = 50
const gap = 1
const total = 10000

function getRandomColor() {
  const r = Math.floor(Math.random() * 220)
  const g = Math.floor(Math.random() * 220)
  const b = Math.floor(Math.random() * 220)
  return `rgb(${r},${g},${b})`
}

// 数学直接计算行列，不再循环遍历（更快）
function getCellByMouse(mx, my) {
  const col = Math.floor(mx / (cellSize + gap))
  const row = Math.floor(my / (cellSize + gap))
  return { row, col }
}

let ctx = null
let currentText = ''

onMounted(() => {
  const canvas = canvasRef.value
  const boxDom = boxRef.value
  const tipEl = tipDom.value

  const colCount = Math.ceil(Math.sqrt(total))
  const rowCount = Math.ceil(total / colCount)
  const totalWidth = (cellSize + gap) * colCount - gap
  const totalHeight = (cellSize + gap) * rowCount - gap

  canvas.width = totalWidth
  canvas.height = totalHeight
  ctx = canvas.getContext('2d')

  // 绘制网格
  let count = 0
  for (let r = 0; r < rowCount; r++) {
    for (let c = 0; c < colCount; c++) {
      if (count >= total) break
      const x = c * (cellSize + gap)
      const y = r * (cellSize + gap)
      ctx.fillStyle = getRandomColor()
      ctx.fillRect(x, y, cellSize, cellSize)

      ctx.fillStyle = '#fff'
      ctx.font = '11px sans-serif'
      ctx.textAlign = 'center'
      ctx.textBaseline = 'middle'
      ctx.fillText(`${r + 1},${c + 1}`, x + cellSize / 2, y + cellSize / 2)
      count++
    }
  }

  // mousemove：实时更新位置，只在格子变化时修改文字
  canvas.onmousemove = (e) => {
    // 实时跟随鼠标位置（核心修复）
    const containerRect = boxDom.getBoundingClientRect()
    tipEl.style.left = `${e.clientX - containerRect.left + 15}px`
    tipEl.style.top = `${e.clientY - containerRect.top + 20}px`

    // 获取格子
    const canvasRect = canvas.getBoundingClientRect()
    const mx = e.clientX - canvasRect.left
    const my = e.clientY - canvasRect.top
    const cell = getCellByMouse(mx, my)

    const newText = `行：${cell.row} 列：${cell.col}`
    if (newText !== currentText) {
      currentText = newText
      tipEl.innerText = newText
    }
    tipEl.style.display = 'block'
  }

  canvas.onmouseleave = () => {
    tipEl.style.display = 'none'
    currentText = ''
  }

  canvas.onclick = (e) => {
    const canvasRect = canvas.getBoundingClientRect()
    const mx = e.clientX - canvasRect.left
    const my = e.clientY - canvasRect.top
    const cell = getCellByMouse(mx, my)
    console.log(`点击格子：行${cell.row} 列${cell.col}`)
  }
})

onUnmounted(() => {
  ctx = null
})
</script>