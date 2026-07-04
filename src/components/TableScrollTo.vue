<template>
  <div class="w-full h-screen">
    <div class="flex flex-wrap gap-4 justify-center items-center mb-6 p-4 bg-gray-50 rounded-lg">
      <span>目标页码：</span>
      <el-input-number
          v-model="targetPage"
          label="目标页码"
          :min="1"
          :max="pageTotal"
          class="w-40"
      />

      <span>页内行号：</span>
      <el-input-number
          v-model="targetRow"
          label="页内行号"
          :min="1"
          :max="pageSize"
          class="w-40"
      />

      <el-button type="primary" @click="scrollToTargetRow">跳转到指定行</el-button>
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
      <el-table-column prop="name" label="姓名" min-width="120" />
      <el-table-column prop="phone" label="手机号" min-width="140" />
      <el-table-column prop="address" label="地址" min-width="220" />
      <el-table-column prop="createTime" label="创建时间" min-width="180" />
    </el-table>

    <!-- 分页 -->
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

// 表格DOM实例
const tableRef = ref(null)
// 总数据量 3000条
const total = ref(3000)
// 分页参数
const pageNum = ref(1)
const pageSize = ref(100)
// 跳转目标
const targetPage = ref(1)
const targetRow = ref(1)
// 当前高亮行ID
const currentHighlightId = ref(null)

// row-class-name 回调：根据 highlightedRowId 给目标行添加自定义类
const rowClassName = ({ row }) => {
  return row.id === currentHighlightId.value ? 'highlight-target-row' : ''
}

// 全量3000条原始数据
const allData = ref([])
// 当前页展示数据
const tableData = computed(() => {
  const start = (pageNum.value - 1) * pageSize.value
  const end = start + pageSize.value
  return allData.value.slice(start, end)
})
// 总页数
const pageTotal = computed(() => Math.ceil(total.value / pageSize.value))

// 生成模拟3000条数据
function generateMockData() {
  const nameList = ['张三', '李四', '王五', '赵六', '钱七', '孙八', '周九', '吴十']
  const addrList = ['北京市朝阳区', '上海市浦东新区', '广州市天河区', '深圳市南山区', '杭州市西湖区']
  const list = []
  for (let i = 1; i <= 3000; i++) {
    list.push({
      id: i,
      name: nameList[Math.floor(Math.random() * nameList.length)] + i,
      phone: `138${Math.floor(Math.random() * 100000000)}`,
      address: addrList[Math.floor(Math.random() * addrList.length)] + '某某街道小区' + i + '号',
      createTime: new Date(Date.now() - Math.floor(Math.random() * 365 * 24 * 3600 * 1000)).toLocaleString()
    })
  }
  allData.value = list
}

// 分页切换
const handleSizeChange = () => {
  pageNum.value = 1
}
const handlePageChange = () => {}

// 核心：跳转到指定页+指定行
async function scrollToTargetRow() {
  // 校验页码
  if (targetPage.value < 1 || targetPage.value > pageTotal.value) {
    ElMessage.error(`页码范围 1 ~ ${pageTotal.value}`)
    return
  }
  // 校验页内行号
  const maxRow = Math.min(pageSize.value, total.value - (targetPage.value - 1) * pageSize.value)
  if (targetRow.value < 1 || targetRow.value > maxRow) {
    ElMessage.error(`页内行号范围 1 ~ ${maxRow}`)
    return
  }

  // 先清除旧高亮，避免上一页高亮残留
  currentHighlightId.value = null

  // 切换到目标页
  pageNum.value = targetPage.value

  // 等待表格 DOM 完全渲染：el-table 内部需要多轮更新
  // 使用双重 nextTick + requestAnimationFrame 确保行元素已挂载
  await nextTick()
  await nextTick()
  await new Promise(resolve => requestAnimationFrame(resolve))

  // 获取目标行数据
  const targetIndex = targetRow.value - 1
  const rowItem = tableData.value[targetIndex]
  if (!rowItem) return

  // 设置高亮行（通过 row-class-name 回调驱动，完全不依赖 Element Plus 内部状态）
  currentHighlightId.value = rowItem.id

  // 精确计算目标行的滚动位置
  const bodyWrapper = tableRef.value?.$el?.querySelector('.el-scrollbar__wrap')
    || tableRef.value?.$el?.querySelector('.el-table__body-wrapper')

  let scrollTop = 0
  if (bodyWrapper) {
    // 累加前 targetIndex 行的真实 offsetHeight，避免固定行高估算错误
    const rowEls = bodyWrapper.querySelectorAll('.el-table__row')
    for (let i = 0; i < targetIndex && i < rowEls.length; i++) {
      scrollTop += rowEls[i].offsetHeight
    }
  } else {
    scrollTop = targetIndex * 48
  }

  // 执行滚动
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
/* 高亮目标行：使用 row-class-name 绑定的自定义类，完全不依赖 Element Plus 内部状态 */
:deep(.el-table__body tr.highlight-target-row > td.el-table__cell) {
  background-color: #ffe8a3 !important;
  color: #b8860b;
  font-weight: 600;
}
:deep(.el-table__body tr.highlight-target-row:hover > td.el-table__cell) {
  background-color: #ffd873 !important;
}
</style>