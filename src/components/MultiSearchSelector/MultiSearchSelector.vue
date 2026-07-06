<template>
  <div class="selector-wrapper">
    <div ref="containerRef" class="selector-container" @focusout="handleFocusOut">
      <el-input
          ref="triggerInputRef"
          readonly
          class="trigger-input"
          :class="{ 'dropdown-open': dropdownVisible }"
          @click="handleToggleDropdown"
      >
        <template #prefix>
          <div class="tags-area">
            <span v-if="modelValue.length === 0" class="placeholder-text">Please select</span>
            <template v-else>
              <el-tag
                  v-for="opt in displayTags"
                  :key="opt.value"
                  closable
                  size="small"
                  @close="handleRemoveTag(opt.value, $event)"
              >
                {{ opt.label }}
              </el-tag>
              <span v-if="extraCount > 0" class="extra-count">+{{ extraCount }}</span>
            </template>
          </div>
        </template>
        <template #suffix>
          <div class="suffix-area">
            <button
                v-if="modelValue.length > 0"
                @click.stop="handleClearSelected"
                class="clear-btn"
                title="Clear all selections"
            >
              ×
            </button>
            <svg
                v-else
                class="dropdown-arrow"
                :class="{ 'rotate-180': dropdownVisible }"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
            </svg>
          </div>
        </template>
      </el-input>
    </div>

    <Teleport to="body">
      <div
          ref="panelRef"
          v-show="dropdownVisible"
          class="dropdown-panel"
          :style="panelPosition"
          @mousedown="markPanelInteracting"
      >
        <div class="search-box">
          <el-input
              ref="searchInputRef"
              v-model="searchText"
              type="text"
              placeholder="Search..."
              @input="handleSearchInput"
              clearable
              @clear="handleClearSearch">
            <template #prefix>
              <el-icon class="el-input__icon">
                <search/>
              </el-icon>
            </template>
          </el-input>
        </div>

        <div class="select-all-bar">
          <el-checkbox :model-value="isAllSelected" @change="handleToggleAll" :indeterminate="isIndeterminate">
            All
          </el-checkbox>
          <span class="selection-count">{{ modelValue.length }}/{{ options.length }}</span>
        </div>

        <div class="scroll-area">
          <div v-if="filteredOptions.length === 0" class="no-match">No match</div>
          <div
              v-for="opt in filteredOptions"
              :key="opt.value"
              class="option-row"
              @click="handleToggleOption(opt.value)"
          >
            <el-checkbox :model-value="modelValue.includes(opt.value)" @change="handleToggleOption(opt.value)"
                         @click.stop>
              {{ opt.label }}
            </el-checkbox>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<script lang="ts" setup>
import {computed, nextTick, onMounted, onUnmounted, ref, useTemplateRef, watch} from "vue";
import { Search } from '@element-plus/icons-vue'

interface Option {
  value: string;
  label: string;
}

const props = defineProps<{
  options: Option[];
  modelValue: string[];
  maxSelection?: number;
}>();

const emit = defineEmits<{
  'update:modelValue': [value: string[]];
  change: [value: string[]];
}>();

const dropdownVisible = ref(false);

const searchInputRef = useTemplateRef("searchInputRef");
const triggerInputRef = useTemplateRef("triggerInputRef");
const containerRef = useTemplateRef("containerRef");
const panelRef = useTemplateRef("panelRef");

const searchText = ref('');
const panelInteracting = ref(false);
const panelPosition = ref({});

const maxDisplayTags = ref(2);

const effectiveMaxSelection = computed(() => props.maxSelection ?? 600);

const filteredOptions = computed(() => {
  const kw = searchText.value.trim().toLowerCase();
  return kw ? props.options.filter(o => o.label.toLowerCase().includes(kw) || o.value.toLowerCase().includes(kw)) : props.options;
});

const displayTags = computed(() => {
  const selected = props.options.filter(o => props.modelValue.includes(o.value));
  return selected.slice(0, maxDisplayTags.value);
});

const extraCount = computed(() => {
  return Math.max(0, props.modelValue.length - maxDisplayTags.value);
});

const isAllSelected = computed(() => {
  const filtered = filteredOptions.value;
  return filtered.length > 0 && filtered.every(o => props.modelValue.includes(o.value));
});

const isIndeterminate = computed(() => {
  const filtered = filteredOptions.value;
  if (filtered.length === 0) return false;
  const count = filtered.filter(o => props.modelValue.includes(o.value)).length;
  return count > 0 && count < filtered.length;
});

const calculatePanelPosition = () => {
  const container = containerRef.value;
  if (!container) return;
  const rect = container.getBoundingClientRect();
  panelPosition.value = {
    left: rect.left + 'px',
    top: rect.bottom + 4 + 'px',
    width: rect.width + 'px'
  };
};

const handleToggleDropdown = () => {
  dropdownVisible.value = !dropdownVisible.value;
  if (dropdownVisible.value) {
    nextTick(() => {
      calculatePanelPosition();
      searchInputRef.value?.focus();
    });
  }
};

const updateValue = (newValue: string[]) => {
  emit('update:modelValue', newValue);
  emit('change', newValue);
};

const handleToggleOption = (value: string) => {
  const index = props.modelValue.indexOf(value);
  let newValue: string[];
  if (index > -1) {
    newValue = props.modelValue.filter(v => v !== value);
  } else {
    if (props.modelValue.length >= effectiveMaxSelection.value) return;
    newValue = [...props.modelValue, value];
  }
  updateValue(newValue);
};

const handleSearchInput = (value: string) => {
  if (value) {
    updateValue([]);
  }
};

const handleToggleAll = (checked: boolean) => {
  const vals = filteredOptions.value.map(o => o.value);
  let newValue: string[];
  if (checked) {
    const set = new Set(props.modelValue);
    vals.forEach(v => {
      if (set.size < effectiveMaxSelection.value) {
        set.add(v);
      }
    });
    newValue = [...set];
  } else {
    const set = new Set(vals);
    newValue = props.modelValue.filter(v => !set.has(v));
  }
  updateValue(newValue);
};

const handleClearSearch = () => {
  searchText.value = '';
  nextTick(() => searchInputRef.value?.focus());
};

const handleClearSelected = () => {
  updateValue([]);
};

const handleRemoveTag = (value: string, event: Event | undefined) => {
  if (event) event.stopPropagation();
  const newValue = props.modelValue.filter(v => v !== value);
  updateValue(newValue);
};

const handleClickOutside = (e: MouseEvent) => {
  if (dropdownVisible.value) {
    const container = containerRef.value;
    const panel = panelRef.value;
    if (container && !container.contains(e.target) && (!panel || !panel.contains(e.target))) {
      dropdownVisible.value = false;
    }
  }
};

const handleFocusOut = (e: FocusEvent) => {
  if (!dropdownVisible.value) return;
  const container = containerRef.value;
  const panel = panelRef.value;
  if (!container) return;
  if (e.relatedTarget && (container.contains(e.relatedTarget as Node) || (panel && panel.contains(e.relatedTarget as Node)))) return;
  if (panelInteracting.value) {
    panelInteracting.value = false;
    return;
  }
  setTimeout(() => {
    const active = document.activeElement;
    if (container && !container.contains(active) && (!panel || !panel.contains(active))) {
      dropdownVisible.value = false;
    }
  }, 0);
};

const markPanelInteracting = () => {
  panelInteracting.value = true;
};

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape' && dropdownVisible.value) dropdownVisible.value = false;
};

const handleResize = () => {
  if (dropdownVisible.value) {
    calculatePanelPosition();
  }
};

watch(() => props.modelValue, () => {
  if (dropdownVisible.value) {
    nextTick(() => calculatePanelPosition());
  }
});

onMounted(() => {
  document.addEventListener('click', handleClickOutside);
  document.addEventListener('keydown', handleKeydown);
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside);
  document.removeEventListener('keydown', handleKeydown);
  window.removeEventListener('resize', handleResize);
});
</script>

<style scoped>
.selector-wrapper {
  width: 100%;
}

.selector-container {
  position: relative;
}

.trigger-input {
  width: 100%;
}

:deep(.trigger-input .el-input__wrapper) {
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  cursor: pointer;
  padding: 6px 12px;
}

:deep(.trigger-input.dropdown-open .el-input__wrapper) {
  border-color: #409eff;
}

.tags-area {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 4px;
  min-width: 0;
  flex: 1;
}

.placeholder-text {
  color: #c0c4cc;
  font-size: 14px;
}

.extra-count {
  font-size: 14px;
  color: #909399;
  margin-left: 4px;
  flex-shrink: 0;
}

.suffix-area {
  display: flex;
  align-items: center;
  flex-shrink: 0;
  margin-left: 4px;
}

.clear-btn {
  color: #c0c4cc;
  font-size: 18px;
  line-height: 1;
  padding: 0 4px;
  cursor: pointer;
  border: none;
  background: transparent;
}

.clear-btn:hover {
  color: #606266;
}

.dropdown-arrow {
  width: 16px;
  height: 16px;
  transition: transform 0.3s ease;
}

.dropdown-arrow.rotate-180 {
  transform: rotate(180deg);
}

.dropdown-panel {
  position: fixed;
  z-index: 3000;
  margin-top: 4px;
  border: 1px solid #ebeef5;
  border-radius: 4px;
  background-color: #ffffff;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
}

.search-box {
  padding: 8px;
  border-bottom: 1px solid #dcdfe6;
  position: relative;
}

.select-all-bar {
  padding: 8px 12px;
  border-bottom: 1px solid #dcdfe6;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.selection-count {
  font-size: 12px;
  color: #909399;
}

.scroll-area {
  max-height: 192px;
  overflow-y: auto;
  padding: 4px;
}

.no-match {
  text-align: center;
  padding: 16px;
  color: #c0c4cc;
  font-size: 14px;
}

.option-row {
  display: flex;
  align-items: center;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
}

.option-row:hover {
  background-color: #f5f7fa;
}
</style>