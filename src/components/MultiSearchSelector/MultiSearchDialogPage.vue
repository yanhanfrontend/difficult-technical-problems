<template>
  <div class="dialog-page">
    <el-button type="primary" @click="dialogVisible = true">Open Dialog</el-button>
    <el-dialog
        v-model="dialogVisible"
        title="MultiSearchSelector Test"
        width="500px"
        :close-on-click-modal="false"
    >
      <div class="dialog-content">
        <MultiSearchSelector
            :options="options"
            v-model="selectedValues"
            :max-selection="600"
            @change="handleChange"
        />
      </div>
    </el-dialog>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import MultiSearchSelector from './MultiSearchSelector.vue'

const dialogVisible = ref(false)

const options = Array.from({ length: 1000 }, (_, i) => ({
  value: `item-${i + 1}`,
  label: `Item ${i + 1}`
}));

const selectedValues = ref<string[]>([]);

const handleChange = (value: string[]) => {
  console.log('Selected values changed:', value);
};
</script>

<style scoped>
.dialog-page {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.dialog-content {
  padding: 20px;
}
</style>