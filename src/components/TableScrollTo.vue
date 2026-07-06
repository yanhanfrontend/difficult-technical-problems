<template>
  <div class="w-full h-screen">
    <div class="flex flex-wrap gap-4 justify-center items-center mb-6 p-4 bg-gray-50 rounded-lg">
      <span>Target Page:</span>
      <el-input-number
          v-model="targetPage"
          label="Target Page"
          :min="1"
          :max="pageTotal"
          class="w-40"
      />

      <span>Row Number:</span>
      <el-input-number
          v-model="targetRow"
          label="Row Number"
          :min="1"
          :max="pageSize"
          class="w-40"
      />

      <el-button type="primary" @click="scrollToTargetRow">Jump to Row</el-button>
    </div>

    <el-table
        ref="tableRef"
        :data="tableData"
        border
        stripe
        height="600"
        :row-key="(row) => row.id"
        :highlight-current-row="true"
        :row-class-name="rowClassName"
        class="w-full"
    >
      <el-table-column prop="id" label="ID" width="100" align="center" />
      <el-table-column prop="name" label="Name" min-width="120" />
      <el-table-column prop="phone" label="Phone" min-width="140" />
      <el-table-column prop="address" label="Address" min-width="220" />
      <el-table-column prop="createTime" label="Create Time" min-width="180" />
    </el-table>

    <!-- Pagination -->
    <div class="mt-4 flex justify-end">
      <el-pagination
          v-model:current-page="pageNum"
          v-model:page-size="pageSize"
          :page-sizes="[50, 100, 200, 500]"
          :total="total"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handlePageChange"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { ElMessage } from 'element-plus'
import LargeDataGrid from "./LargeDataGrid.vue";

// Table DOM instance
const tableRef = ref(null)
// Total data count: 3000 records
const total = ref(3000)
// Pagination params
const pageNum = ref(1)
const pageSize = ref(100)
// Jump target
const targetPage = ref(1)
const targetRow = ref(1)
// Current highlighted row ID
const currentHighlightId = ref(null)

// row-class-name callback: add custom class based on highlightedRowId
const rowClassName = ({ row }) => {
  return row.id === currentHighlightId.value ? 'highlight-target-row' : ''
}

// Full 3000 raw records
const allData = ref([])
// Current page data
const tableData = computed(() => {
  const start = (pageNum.value - 1) * pageSize.value
  const end = start + pageSize.value
  return allData.value.slice(start, end)
})
// Total pages
const pageTotal = computed(() => Math.ceil(total.value / pageSize.value))

// Generate mock 3000 records
function generateMockData() {
  const nameList = ['ZhangSan', 'LiSi', 'WangWu', 'ZhaoLiu', 'QianQi', 'SunBa', 'ZhouJiu', 'WuShi']
  const addrList = ['Chaoyang District, Beijing', 'Pudong District, Shanghai', 'Tianhe District, Guangzhou', 'Nanshan District, Shenzhen', 'West Lake District, Hangzhou']
  const list = []
  for (let i = 1; i <= 3000; i++) {
    list.push({
      id: i,
      name: nameList[Math.floor(Math.random() * nameList.length)] + i,
      phone: `138${Math.floor(Math.random() * 100000000)}`,
      address: addrList[Math.floor(Math.random() * addrList.length)] + ' Street ' + i,
      createTime: new Date(Date.now() - Math.floor(Math.random() * 365 * 24 * 3600 * 1000)).toLocaleString()
    })
  }
  allData.value = list
}

// Pagination change
const handleSizeChange = () => {
  pageNum.value = 1
}
const handlePageChange = () => {}

// Core: Jump to specified page and row
async function scrollToTargetRow() {
  // Validate page number
  if (targetPage.value < 1 || targetPage.value > pageTotal.value) {
    ElMessage.error(`Page range: 1 ~ ${pageTotal.value}`)
    return
  }
  // Validate row number
  const maxRow = Math.min(pageSize.value, total.value - (targetPage.value - 1) * pageSize.value)
  if (targetRow.value < 1 || targetRow.value > maxRow) {
    ElMessage.error(`Row range: 1 ~ ${maxRow}`)
    return
  }

  // Clear old highlight to avoid previous page highlight residue
  currentHighlightId.value = null

  // Switch to target page
  pageNum.value = targetPage.value

  // Wait for table DOM to fully render: el-table needs multiple rounds of updates
  // Use double nextTick + requestAnimationFrame to ensure row elements are mounted
  await nextTick()
  await nextTick()
  await new Promise(resolve => requestAnimationFrame(resolve))

  // Get target row data
  const targetIndex = targetRow.value - 1
  const rowItem = tableData.value[targetIndex]
  if (!rowItem) return

  // Set highlighted row (driven by row-class-name callback, completely independent of Element Plus internal state)
  currentHighlightId.value = rowItem.id

  // Calculate scroll position precisely
  const bodyWrapper = tableRef.value?.$el?.querySelector('.el-scrollbar__wrap')
    || tableRef.value?.$el?.querySelector('.el-table__body-wrapper')

  let scrollTop = 0
  if (bodyWrapper) {
    // Accumulate real offsetHeight of previous targetIndex rows to avoid fixed height estimation errors
    const rowEls = bodyWrapper.querySelectorAll('.el-table__row')
    for (let i = 0; i < targetIndex && i < rowEls.length; i++) {
      scrollTop += rowEls[i].offsetHeight
    }
  } else {
    scrollTop = targetIndex * 48
  }

  // Execute scroll
  if (tableRef.value && typeof tableRef.value.scrollTo === 'function') {
    tableRef.value.scrollTo({ top: scrollTop, behavior: 'smooth' })
  } else if (bodyWrapper) {
    bodyWrapper.scrollTo({ top: scrollTop, behavior: 'smooth' })
  }
}

onMounted(() => {
  generateMockData()
})
</script>

<style scoped>
/* Highlight target row: use custom class bound by row-class-name, completely independent of Element Plus internal state */
:deep(.el-table__body tr.highlight-target-row > td.el-table__cell) {
  background-color: #ffe8a3 !important;
  color: #b8860b;
  font-weight: 600;
}
:deep(.el-table__body tr.highlight-target-row:hover > td.el-table__cell) {
  background-color: #ffd873 !important;
}
</style>