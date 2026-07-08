<template>
  <div class="tag-drag-drop">
    <el-row :gutter="20">
      <el-col :span="12">
        <el-card shadow="never" title="Source Tags" class="tag-card">
          <el-input v-model="inputText" type="textarea" :rows="3" placeholder="Input text and press Enter..."
            @keydown.enter.prevent="handleConvert" resize="none" class="input-area" ref="inputRef" />
          <div class="left-tags-container" @dragover.prevent @drop="handleDropLeft">
            <el-tag v-for="tag in leftTags" :key="tag.id" :label="tag.text" closable draggable="true"
              @dragstart="handleDragStart($event, tag, 'left')" @close="handleRemoveTag(tag.id, 'left')"
              class="draggable-tag">
              {{ tag.text }}
            </el-tag>
          </div>
        </el-card>
      </el-col>

      <el-col :span="12">
        <el-card shadow="never" title="Target Tags" class="tag-card">
          <div class="right-tags-container" @dragover.prevent @drop="handleDropRight">
            <el-tag v-for="tag in rightTags" :key="tag.id" :label="tag.text" closable draggable="true"
              @dragstart="handleDragStart($event, tag, 'right')" @close="handleRemoveTag(tag.id, 'right')"
              class="draggable-tag">
              {{ tag.text }}
            </el-tag>
          </div>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

interface Tag {
  id: string;
  text: string;
}

const generateInitialTags = (): Tag[] => {
  const tags: Tag[] = [];
  const patterns = [
    'Short',
    'Medium length text',
    'This is a longer tag',
    'A',
    'Normal',
    'Small',
    'Short tag',
    'Medium',
    'Brief',
    'Concise',
  ];
  const longPatterns = [
    'This tag contains extremely long text content that definitely exceeds the container width by far',
    'Here is another very long tag text that will be truncated with ellipsis showing',
    'A super lengthy tag demonstrating the text-overflow ellipsis behavior in action',
    'This example tag has so much text that it cannot fit within the 80% width limit',
    'Extraordinary long tag content that will surely trigger the ellipsis display',
  ];
  for (let i = 0; i < 50; i++) {
    let text: string;
    if (i % 4 === 0) {
      const longPattern = longPatterns[i % longPatterns.length];
      text = longPattern.repeat(Math.floor(Math.random() * 2) + 1);
    } else {
      const pattern = patterns[i % patterns.length];
      const multiplier = Math.floor(Math.random() * 2) + 1;
      text = pattern.repeat(multiplier).slice(0, Math.floor(Math.random() * 30) + 5);
    }
    tags.push({
      id: `tag-init-${i}`,
      text,
    });
  }
  return tags;
};

const inputText = ref('');
const leftTags = ref<Tag[]>(generateInitialTags());
const rightTags = ref<Tag[]>([]);
const draggedTag = ref<{ tag: Tag; source: 'left' | 'right' } | null>(null);
const inputRef = ref<any>(null);

let tagIdCounter = 50;

const generateId = () => `tag-${++tagIdCounter}-${Date.now()}`;

const handleConvert = () => {
  const text = inputText.value.trim();
  if (!text) return;

  const lines = text.split('\n').map(line => line.trim()).filter(line => line);

  lines.forEach(line => {
    if (!leftTags.value.some(t => t.text === line)) {
      leftTags.value.push({
        id: generateId(),
        text: line
      });
    }
  });

  inputText.value = '';
};

const handleDragStart = (e: DragEvent, tag: Tag, source: 'left' | 'right') => {
  draggedTag.value = { tag, source };
  if (e.dataTransfer) {
    e.dataTransfer.effectAllowed = 'move';
    e.dataTransfer.setData('text/plain', tag.id);
  }
};

const handleDropLeft = () => {
  if (!draggedTag.value) return;
  if (draggedTag.value.source === 'right') {
    const tag = draggedTag.value.tag;
    rightTags.value = rightTags.value.filter(t => t.id !== tag.id);
    if (!leftTags.value.some(t => t.id === tag.id)) {
      leftTags.value.push(tag);
    }
  }
  draggedTag.value = null;
};

const handleDropRight = () => {
  if (!draggedTag.value) return;
  if (draggedTag.value.source === 'left') {
    const tag = draggedTag.value.tag;
    leftTags.value = leftTags.value.filter(t => t.id !== tag.id);
    if (!rightTags.value.some(t => t.id === tag.id)) {
      rightTags.value.push(tag);
    }
  }
  draggedTag.value = null;
};

const handleRemoveTag = (id: string, source: 'left' | 'right') => {
  if (source === 'left') {
    leftTags.value = leftTags.value.filter(t => t.id !== id);
  } else {
    rightTags.value = rightTags.value.filter(t => t.id !== id);
  }
};
</script>

<style scoped>
.tag-drag-drop {
  height: 100%;
}

.tag-drag-drop :deep(.el-row) {
  height: 100%;
}

.tag-drag-drop :deep(.el-col) {
  height: 100%;
}

.tag-card {
  height: calc(100vh - 104px);
  display: flex;
  flex-direction: column;
}

.tag-card :deep(.el-card__body) {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 12px;
}

.left-tags-container {
  height: calc(100% - 94px);
  margin-top: 16px;
  padding: 8px;
  border: 2px dashed #dcdfe6;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: flex-start;
  overflow-x: hidden;
  overflow-y: auto;
}

.right-tags-container {
  height: 100%;
  padding: 8px;
  border: 2px dashed #dcdfe6;
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: flex-start;
  overflow-x: hidden;
  overflow-y: auto;
}

.input-area {
  width: 100%;
}

.draggable-tag {
  cursor: grab;
  user-select: none;
  display: inline-flex;
  align-items: center;
  width: auto;
  max-width: 80%;
  height: 28px;
  line-height: 28px;
}

.draggable-tag :deep(.el-tag__content) {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.draggable-tag:active {
  cursor: grabbing;
}

.empty-hint {
  text-align: center;
  color: #c0c4cc;
  font-size: 14px;
  padding: 40px 0;
}

.count-info {
  margin-top: 12px;
  padding-top: 12px;
  border-top: 1px solid #ebeef5;
  font-size: 14px;
  color: #606266;
}
</style>