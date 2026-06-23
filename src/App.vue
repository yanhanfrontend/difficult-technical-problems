<template>
  <div class="p-4 w-screen h-screen box-border">
    <el-tabs type="border-card" class="h-full w-full">
      <el-tab-pane label="MultiSearchSelector">
        <el-row align="middle" class="h-full w-full">
          <el-col :offset="10" :span="4">
            <MultiSearchSelector/>
          </el-col>
        </el-row>
      </el-tab-pane>

      <el-tab-pane label="VirtualizedMultiSearchSelector">
        <el-row align="middle" class="h-[300px] w-full">
          <el-col :offset="10" :span="4">
            <VirtualizedMultiSearchSelector v-model:value="value" v-model:options="options" :loading="loading"
                                            @visible-change="handleVisibleChange"/>

            <div class="mt-2 max-h-[200px] overflow-auto">
              <span>Selected: {{ value }}</span>
            </div>
          </el-col>
        </el-row>
      </el-tab-pane>

      <el-tab-pane label="VirtualizedTable">
        <el-row align="middle" class="h-[300px] w-full">
          <el-col :offset="8" :span="8">
            <VirtualizedTable/>
          </el-col>
        </el-row>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script lang="ts" setup>
import {onBeforeUnmount, ref} from "vue";
import type {CheckboxValueType} from "element-plus";
import VirtualizedMultiSearchSelector from "./components/VirtualizedMultiSearchSelector.vue";
import VirtualizedTable from "./components/VirtualizedTable.vue";
import MultiSearchSelector from "./components/MultiSearchSelector.vue";

type OptionItem = {
  value: string;
  label: string;
};

const initials = ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j"];

const value = ref<CheckboxValueType[]>([]);

const loading = ref(false);

const allOptions: OptionItem[] = Array.from({length: 1000}).map((_, idx) => ({
  value: `Option ${idx + 1}`,
  label: `${initials[idx % 10]}${idx}`,
}));

const options = ref<OptionItem[]>([]);

let searchTimer: ReturnType<typeof setTimeout> | undefined;
let requestId = 0;

const fetchRemoteOptions = async (keyword = '') => {
  const currentRequestId = ++requestId;
  const normalizedKeyword = keyword.trim().toLowerCase();

  loading.value = true;

  await new Promise((resolve) => setTimeout(resolve, 300));

  if (currentRequestId !== requestId) {
    return;
  }

  options.value = allOptions.filter((option) => {
    if (!normalizedKeyword) {
      return true;
    }

    return (
        option.label.toLowerCase().includes(normalizedKeyword) ||
        option.value.toLowerCase().includes(normalizedKeyword)
    );
  });

  loading.value = false;
};

const handleVisibleChange = (visible: boolean) => {
  if (visible) {
    fetchRemoteOptions();
  }
};

onBeforeUnmount(() => {
  if (searchTimer) {
    clearTimeout(searchTimer);
  }
});
</script>

<style scoped>

</style>
