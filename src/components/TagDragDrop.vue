<template>
  <div class="tag-drag-drop">
    <el-row :gutter="20">
      <el-col :span="6">
        <el-card shadow="never" title="Source Tags" class="tag-card">
          <el-input v-model="inputText" type="textarea" :rows="10" placeholder="Input waferIds and press Enter..."
            @keydown.enter.prevent="handleConvert" resize="none" class="input-area" ref="inputRef" />
          <draggable v-model="leftTags" :group="tagGroup" item-key="id" class="left-tags-container"
            v-bind="dragOptions">
            <template #item="{ element }">
              <el-tag :label="element.text" effect="light" class="draggable-tag" closable
                disable-transitions @close="handleDeleteTag(element, leftTags)">
                {{ element.text }}
              </el-tag>
            </template>
          </draggable>
        </el-card>
      </el-col>

      <el-col :span="18">
        <div class="right-section">
          <div class="right-header">
            <span class="section-title">Target Tags</span>
            <el-button type="primary" size="small" @click="handleAddContainer">
              <ElIcon><Plus /></ElIcon> Add Container
            </el-button>
          </div>
          <div class="right-containers-scroll">
            <div class="right-containers-wrapper">
              <el-card v-for="container in rightContainers" :key="container.id" shadow="never" class="tag-card-right">
                <template #header>
                  <div class="card-header">
                    <span>{{ container.name }}</span>
                    <el-button v-if="rightContainers.length > 1" type="danger" size="small"
                      @click="handleRemoveContainer(container.id)">
                      <ElIcon><Delete /></ElIcon>
                    </el-button>
                  </div>
                </template>
                <draggable v-model="container.tags" :group="tagGroup" item-key="id" class="right-tags-container"
                  v-bind="dragOptions">
                  <template #item="{ element }">
                    <el-tag :label="element.text" effect="light" class="draggable-tag" closable
                      disable-transitions @close="handleDeleteTag(element, container.tags)">
                      {{ element.text }}
                    </el-tag>
                  </template>
                </draggable>
              </el-card>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'
import draggable from 'vuedraggable'
import { Plus, Delete } from '@element-plus/icons-vue'
import { ElIcon } from 'element-plus'

interface Tag {
  id: string
  text: string
}

interface Container {
  id: string
  name: string
  tags: Tag[]
}

const generateInitialTags = (): Tag[] => {
  const tags: Tag[] = []
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
  ]
  const longPatterns = [
    'This tag contains extremely long text content that definitely exceeds the container width by far',
    'Here is another very long tag text that will be truncated with ellipsis showing',
    'A super lengthy tag demonstrating the text-overflow ellipsis behavior in action',
    'This example tag has so much text that it cannot fit within the 80% width limit',
    'Extraordinary long tag content that will surely trigger the ellipsis display',
  ]
  for (let i = 0; i < 50; i++) {
    let text: string
    if (i % 4 === 0) {
      const longPattern = longPatterns[i % longPatterns.length]
      text = longPattern.repeat(Math.floor(Math.random() * 2) + 1)
    } else {
      const pattern = patterns[i % patterns.length]
      const multiplier = Math.floor(Math.random() * 2) + 1
      text = pattern.repeat(multiplier).slice(0, Math.floor(Math.random() * 30) + 5)
    }
    tags.push({
      id: `tag-init-${i}`,
      text,
    })
  }
  return tags
}

const inputText = ref('')
const leftTags = ref<Tag[]>(generateInitialTags())
const rightContainers = ref<Container[]>([
  {
    id: 'container-1',
    name: 'Container 1',
    tags: [],
  },
])
const inputRef = ref<any>(null)

let tagIdCounter = 50
let containerIdCounter = 1

const generateId = () => `tag-${++tagIdCounter}-${Date.now()}`

const tagGroup = {
  name: 'tags',
  pull: true,
  put: true,
}

const dragOptions = {
  filter: '.el-tag__close',
  preventOnFilter: false,
}

const handleDeleteTag = (element: Tag, list: Tag[]) => {
  const index = list.findIndex(t => t.id === element.id)
  if (index !== -1) {
    list.splice(index, 1)
  }
}

const handleConvert = () => {
  const text = inputText.value.trim()
  if (!text) return

  const lines = text.split('\n').map(line => line.trim()).filter(line => line)

  lines.forEach(line => {
    if (!leftTags.value.some(t => t.text === line)) {
      leftTags.value.push({
        id: generateId(),
        text: line,
      })
    }
  })

  inputText.value = ''
}

const handleAddContainer = () => {
  rightContainers.value.push({
    id: `container-${++containerIdCounter}`,
    name: `Container ${containerIdCounter}`,
    tags: [],
  })
}

const handleRemoveContainer = (id: string) => {
  const container = rightContainers.value.find(c => c.id === id)
  if (container) {
    container.tags.forEach(tag => {
      if (!leftTags.value.some(t => t.id === tag.id)) {
        leftTags.value.push(tag)
      }
    })
    rightContainers.value = rightContainers.value.filter(c => c.id !== id)
  }
}
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

.tag-card-right {
  width: 320px;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
}

.tag-card :deep(.el-card__body) {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 12px;
}

.tag-card-right :deep(.el-card__body) {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 12px;
  min-height: 0;
}

.left-tags-container {
  flex: 1;
  min-height: 0;
  margin-top: 16px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  align-items: flex-start;
  overflow-x: hidden;
  overflow-y: auto;
}

.right-section {
  height: calc(100vh - 104px);
  display: flex;
  flex-direction: column;
}

.right-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.section-title {
  font-size: 16px;
  font-weight: 600;
  color: #303133;
}

.right-containers-scroll {
  flex: 1;
  overflow-x: auto;
  overflow-y: hidden;
}

.right-containers-wrapper {
  display: flex;
  gap: 20px;
  padding-bottom: 8px;
  height: 100%;
}

.right-tags-container {
  flex: 1;
  min-height: 0;
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

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>