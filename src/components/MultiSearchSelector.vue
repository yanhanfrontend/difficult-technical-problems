<template>
  <div class="p-6 max-w-md mx-auto">
    <!-- 触发器容器（相对定位，用于悬浮面板） -->
    <div class="relative">
      <!-- 触发器 -->
      <div
          @click="toggleDropdown"
          class="border border-gray-300 rounded px-3 py-2 cursor-pointer flex items-center justify-between"
          :class="{ 'border-blue-500': dropdownVisible }"
      >
        <!-- 左侧：显示标签或占位文本 -->
        <div class="flex-1 flex flex-wrap items-center gap-1 min-w-0">
          <span v-if="selectedValues.length === 0" class="text-gray-400 text-sm">Please select</span>
          <template v-else>
            <el-tag
                v-for="opt in displayTags"
                :key="opt.value"
                closable
                size="small"
                @close="removeTag(opt.value, $event)"
            >
              {{ opt.label }}
            </el-tag>
            <span v-if="extraCount > 0" class="text-sm text-gray-500 ml-1 flex-shrink-0">
                            +{{ extraCount }}
                        </span>
          </template>
        </div>

        <!-- 右侧：有值时显示清空按钮，无值时显示下拉箭头 -->
        <div class="flex items-center flex-shrink-0 ml-1">
          <button
              v-if="selectedValues.length > 0"
              @click.stop="clearSelected"
              class="text-gray-400 hover:text-gray-600 text-lg leading-none px-1"
              title="清空所有已选项"
          >
            ×
          </button>
          <svg
              v-else
              class="w-4 h-4 transition-transform"
              :class="{ 'rotate-180': dropdownVisible }"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
          >
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
          </svg>
        </div>
      </div>

      <!-- 下拉面板（悬浮） -->
      <div
          v-show="dropdownVisible"
          class="absolute left-0 right-0 top-full z-10 mt-1 border border-gray-200 rounded bg-white dropdown-panel"
      >
        <!-- 搜索框 -->
        <div class="p-2 border-b border-gray-300 relative">
          <input
              ref="searchInput"
              v-model="searchText"
              type="text"
              placeholder="Search..."
              class="w-full border border-gray-300 rounded px-2 py-1 text-sm pr-7 focus:outline-none focus:border-blue-400"
          />
          <button
              v-if="searchText"
              @click.stop="clearSearch"
              class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600 text-lg leading-none"
          >
            ×
          </button>
        </div>

        <div class="px-3 py-2 border-b border-b-gray-300 flex items-center justify-between">
          <el-checkbox :model-value="isAllSelected" @change="toggleAll" :indeterminate="isIndeterminate">
            All
          </el-checkbox>
          <span class="text-xs text-gray-500">{{ selectedValues.length }}/{{ options.length }}</span>
        </div>

        <div class="max-h-48 overflow-y-auto scroll-area p-1">
          <div v-if="filteredOptions.length === 0" class="text-center py-4 text-gray-400 text-sm">无匹配</div>
          <div
              v-for="opt in filteredOptions"
              :key="opt.value"
              class="option-row flex items-center px-2 py-1 rounded cursor-pointer"
              @click="toggleOption(opt.value)"
          >
            <el-checkbox :model-value="selectedValues.includes(opt.value)" @change="toggleOption(opt.value)" @click.stop>
              {{ opt.label }}
            </el-checkbox>
          </div>
        </div>
      </div>
    </div>

    <!-- 调试信息（位于面板下方，面板悬浮后不会被挤走） -->
    <div class="mt-3 text-xs text-gray-500">
      已选：{{ selectedValues.join('、') || '无' }}
    </div>
  </div>
</template>

<script lang="ts" setup>
import {computed, nextTick, onBeforeUnmount, onMounted, onUnmounted, PropType, ref, watch} from "vue";
import type {CheckboxValueType} from "element-plus";
import {Search} from "@element-plus/icons-vue";

const options = [
  { value: 'apple', label: '苹果' },
  { value: 'banana', label: '香蕉' },
  { value: 'orange', label: '橙子' },
  { value: 'grape', label: '葡萄' },
  { value: 'watermelon', label: '西瓜' },
  { value: 'strawberry', label: '草莓' },
];

const selectedValues = ref([]);
const searchText = ref('');
const dropdownVisible = ref(false);
const searchInput = ref(null);

// 可配置：最多显示标签数量
const maxDisplayTags = ref(2);

const filteredOptions = computed(() => {
  const kw = searchText.value.trim().toLowerCase();
  return kw ? options.filter(o => o.label.includes(kw) || o.value.includes(kw)) : options;
});

const selectedLabels = computed(() => {
  return options.filter(o => selectedValues.value.includes(o.value)).map(o => o.label);
});

const displayTags = computed(() => {
  const selected = options.filter(o => selectedValues.value.includes(o.value));
  return selected.slice(0, maxDisplayTags.value);
});

const extraCount = computed(() => {
  return Math.max(0, selectedValues.value.length - maxDisplayTags.value);
});

const isAllSelected = computed(() => {
  const filtered = filteredOptions.value;
  return filtered.length > 0 && filtered.every(o => selectedValues.value.includes(o.value));
});

const isIndeterminate = computed(() => {
  const filtered = filteredOptions.value;
  if (filtered.length === 0) return false;
  const count = filtered.filter(o => selectedValues.value.includes(o.value)).length;
  return count > 0 && count < filtered.length;
});

const toggleDropdown = () => {
  dropdownVisible.value = !dropdownVisible.value;
  if (dropdownVisible.value) {
    nextTick(() => searchInput.value?.focus());
  }
};

const toggleOption = (value) => {
  const idx = selectedValues.value.indexOf(value);
  if (idx > -1) selectedValues.value.splice(idx, 1);
  else selectedValues.value.push(value);
};

const toggleAll = (checked) => {
  const vals = filteredOptions.value.map(o => o.value);
  if (checked) {
    const set = new Set(selectedValues.value);
    vals.forEach(v => set.add(v));
    selectedValues.value = [...set];
  } else {
    const set = new Set(vals);
    selectedValues.value = selectedValues.value.filter(v => !set.has(v));
  }
};

const clearSearch = () => {
  searchText.value = '';
  nextTick(() => searchInput.value?.focus());
};

const clearSelected = () => {
  selectedValues.value = [];
};

const removeTag = (value, event) => {
  if (event) event.stopPropagation();
  const idx = selectedValues.value.indexOf(value);
  if (idx > -1) selectedValues.value.splice(idx, 1);
};

const onClickOutside = (e) => {
  if (dropdownVisible.value) {
    // 注意：容器现在是 .relative 父元素，但 .max-w-md 仍然包含它
    const container = document.querySelector('.max-w-md');
    if (container && !container.contains(e.target)) {
      dropdownVisible.value = false;
    }
  }
};

const onKeydown = (e) => {
  if (e.key === 'Escape' && dropdownVisible.value) dropdownVisible.value = false;
};

onMounted(() => {
  document.addEventListener('click', onClickOutside);
  document.addEventListener('keydown', onKeydown);
});
onUnmounted(() => {
  document.removeEventListener('click', onClickOutside);
  document.removeEventListener('keydown', onKeydown);
});
</script>

<style scoped>

</style>
