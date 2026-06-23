<template>
  <el-select-v2
    v-model="value"
    :options="options"
    :loading="loading"
    multiple
    clearable
    popper-class="custom-header"
    collapse-tags
    :max-collapse-tags="1"
    :multiple-limit="MULTIPLE_LIMIT"
    @visible-change="handleVisibleChange"
    placeholder=""
  >
    <template #header>
      <el-input v-model="searchWord" clearable @clear="handleClear">
        <template #prefix>
          <el-icon class="el-input__icon"><search /></el-icon>
        </template>
      </el-input>

      <el-checkbox
        v-model="checkAll"
        :indeterminate="indeterminate"
        @change="handleCheckAll"
        class="mt-2"
      >
        All
      </el-checkbox>
    </template>
  </el-select-v2>
</template>

<script lang="ts" setup>
import { onBeforeUnmount, onMounted, PropType, ref, watch } from "vue";
import type { CheckboxValueType } from "element-plus";
import { Search } from "@element-plus/icons-vue";

defineEmits(["visible-change"]);

const value = defineModel("value");
const options = defineModel("options");

type OptionItem = {
  value: string;
  label: string;
};

const props = defineProps({
  loading: {
    type: Boolean,
    default: false,
  },
});

const MULTIPLE_LIMIT = 100;

const initials = ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j"];

const searchWord = ref("");

const checkAll = ref(false);

const indeterminate = ref(false);

const loading = ref(false);

const allOptions: OptionItem[] = Array.from({ length: 1000 }).map((_, idx) => ({
  value: `Option ${idx + 1}`,
  label: `${initials[idx % 10]}${idx}`,
}));

let searchTimer: ReturnType<typeof setTimeout> | undefined;
let requestId = 0;

const fetchRemoteOptions = async (keyword = "") => {
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

watch(searchWord, (keyword) => {
  if (searchTimer) {
    clearTimeout(searchTimer);
  }

  searchTimer = setTimeout(() => {
    fetchRemoteOptions(keyword);
  }, 500);
});

onMounted(() => {
  fetchRemoteOptions();
});

onBeforeUnmount(() => {
  if (searchTimer) {
    clearTimeout(searchTimer);
  }
});

watch([value, options], ([selectedValues, visibleOptions]) => {
  const visibleValues = visibleOptions.map((option) => option.value);
  const checkedVisibleCount = selectedValues.filter((item) =>
    visibleValues.includes(item),
  ).length;

  if (visibleValues.length === 0 || checkedVisibleCount === 0) {
    checkAll.value = false;
    indeterminate.value = false;
  } else if (checkedVisibleCount === visibleValues.length) {
    checkAll.value = true;
    indeterminate.value = false;
  } else {
    checkAll.value = false;
    indeterminate.value = true;
  }
});

const handleClear = () => {
  searchWord.value = "";
};

const handleCheckAll = (val: CheckboxValueType) => {
  indeterminate.value = false;
  const visibleValues = options.value.map((option) => option.value);

  if (val) {
    const mergedValues = Array.from(
      new Set([...value.value, ...visibleValues]),
    );

    if (mergedValues.length > MULTIPLE_LIMIT) {
      value.value = mergedValues.slice(0, MULTIPLE_LIMIT);
      // ElMessage.warning(`最多只能选择 ${MULTIPLE_LIMIT} 项`);
      return;
    }

    value.value = mergedValues;
  } else {
    value.value = value.value.filter((item) => !visibleValues.includes(item));
  }
};

const handleVisibleChange = (visible: boolean) => {
  if (visible) {
    fetchRemoteOptions();
  }
};
</script>

<style scoped>

</style>
