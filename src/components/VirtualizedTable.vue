<template>
  <div class="flex justify-center items-center">
    <el-table-v2
        :columns="columns"
        :data="tableData"
        :width="1000"
        :height="500"
        fixed
    />
  </div>
</template>

<script lang="ts" setup>
import {ref, shallowRef} from "vue";

const generateColumns = (length = 10, prefix = 'column-', props?: any) =>
    Array.from({length})?.map((_, columnIndex) => ({
      ...props,
      key: `${prefix}${columnIndex}`,
      dataKey: `${prefix}${columnIndex}`,
      title: `Column ${columnIndex}`,
      width: 150,
    }))

const generateData = (
    columns: ReturnType<typeof generateColumns>,
    length = 200,
    prefix = 'row-'
) =>
    Array.from({length}).map((_, rowIndex) => {
      return columns.value.reduce(
          (rowData, column, columnIndex) => {
            rowData[column.dataKey] = `Row ${rowIndex} - Col ${columnIndex}`
            return rowData
          },
          {
            id: `${prefix}${rowIndex}`,
            parentId: null,
          }
      )
    })

const columns = shallowRef(generateColumns(10))
const tableData = shallowRef(generateData(columns, 1000))
</script>

<style scoped>

</style>