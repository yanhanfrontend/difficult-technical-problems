<template>
  <div class="h-[calc(100vh-100px)] w-full">
    <el-container class="h-full w-full">
      <el-aside width="400px" class="bg-blue-50 p-4">
        <div class="font-bold mb-2">API List</div>

        <el-card shadow="never" class="h-[calc(100vh-170px)]">
          <div class="flex flex-col justify-center items-start gap-3">
            <el-tag type="primary">Get</el-tag>

            <el-tag type="success">Post</el-tag>

            <el-tag type="info">Put</el-tag>

            <el-tag type="warning">Patch</el-tag>

            <el-tag type="danger">Delete</el-tag>
          </div>
        </el-card>
      </el-aside>

      <el-main class="bg-rose-50">
        <el-card shadow="never" class="h-full w-full">
          <template #header>
            <div class="flex flex-row justify-between items-center">
              <div class="font-bold">API Invokation</div>
            </div>
          </template>

          <el-table :data="tableData1" border style="width: 100%">
            <el-table-column prop="key" label="Key" width="150px" />

            <el-table-column prop="value" label="Value" />
          </el-table>

          <el-tabs type="border-card" class="mt-4 h-[300px]">
            <el-tab-pane label="Request">
            </el-tab-pane>

            <el-tab-pane label="Response">
            </el-tab-pane>
          </el-tabs>
        </el-card>
      </el-main>

      <el-aside width="500px" class="bg-blue-50 p-4">
        <div class="font-bold mb-2">Environment Variables</div>

        <el-card shadow="never" class="h-[calc(100vh-170px)]">
          <el-table :data="tableData2" border style="width: 100%">
            <el-table-column prop="key" label="Key" width="100px"/>

            <el-table-column prop="value" label="Value" />

            <el-table-column prop="desc" label="Desc" width="150px" />
          </el-table>
        </el-card>
      </el-aside>
    </el-container>
  </div>
</template>

<script setup>
const tableData1 = [
  {
    key: 'Content-Type',
    value: 'application/json',
  },
  {
    key: 'Token',
    value: 'xxxxxxxxxxxxxxxxxxxxx',
  },
]

const tableData2 = [
  {
    key: 'baseUrl',
    value: 'http://localhost:8080',
    desc: '本地服务地址',
  },
]
</script>
