<template>
  <div class="p-4 w-screen h-screen box-border">
    <el-tabs type="border-card" class="h-full w-full">
      <el-tab-pane label="MultiSearchDialog">
        <el-row align="middle">
          <el-col :span="24">
            <MultiSearchDialogPage/>
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

      <el-tab-pane label="TableScrollTo">
        <el-row align="middle">
          <el-col :span="24">
            <TableScrollTo/>
          </el-col>
        </el-row>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script lang="ts" setup>
import LargeDataGrid from "./components/LargeDataGrid.vue";
import MultiSearchDialogPage from "./components/MultiSearchSelector/MultiSearchDialogPage.vue";
import TableScrollTo from "./components/TableScrollTo.vue";
</script>

<style scoped>

</style>
