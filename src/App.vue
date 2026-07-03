<template>
  <div class="p-4 w-screen h-screen box-border">
    <el-tabs type="border-card" class="h-full w-full">
      <el-tab-pane label="TableScrollTo">
        <el-row align="middle">
          <el-col :span="24">
            <TableScrollTo/>
          </el-col>
        </el-row>
      </el-tab-pane>

      <el-tab-pane label="LargeDataGrid">
        <el-row align="middle">
          <el-col :span="24">
            <LargeDataGrid/>
          </el-col>
        </el-row>
      </el-tab-pane>

      <el-tab-pane label="MultiSearchSelector">
        <el-row align="middle">
          <el-col :offset="8" :span="8">
            <MultiSearchSelector/>
          </el-col>
        </el-row>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script lang="ts" setup>
import LargeDataGrid from "./components/LargeDataGrid.vue";
import MultiSearchSelector from "./components/MultiSearchSelector.vue";
import TableScrollTo from "./components/TableScrollTo.vue";
</script>

<style scoped>

</style>
