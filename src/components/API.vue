<template>
  <div class="h-[calc(100vh-100px)] w-full">
    <el-container class="h-full w-full">
      <el-aside width="400px" class="bg-blue-50 p-4">
        <div class="font-bold mb-2">API List</div>

        <el-card shadow="never" class="h-[calc(100vh-170px)]">
          <div class="flex justify-center items-center gap-3">
            <el-button type="primary" plain>&nbsp;&nbsp;&nbsp;&nbsp;</el-button>

            <el-button type="success" plain round>&nbsp;&nbsp;&nbsp;&nbsp;</el-button>

            <el-button type="info" plain dashed>&nbsp;&nbsp;&nbsp;&nbsp;</el-button>

            <el-button type="warning" plain circle></el-button>
          </div>
        </el-card>
      </el-aside>

      <el-main class="bg-rose-50">
        <el-card shadow="never" class="h-full w-full">
          <template #header>
            <div class="flex flex-row justify-between items-center">
              <div class="font-bold">API Invokation</div>

              <el-button type="success" plain circle>
                <el-icon>
                  <Plus />
                </el-icon>
              </el-button>
            </div>
          </template>

          <div class="flex flex-col justify-center items-start gap-3">
            <el-input v-model="input4" class="w-full" placeholder="Search">
              <template #prefix>
                <el-icon class="el-input__icon">
                  <search />
                </el-icon>
              </template>
            </el-input>

            <el-badge :value="1">
              <el-tag type="primary">Container 1</el-tag>
            </el-badge>
            <el-tag type="success">Container 2</el-tag>
            <el-tag type="info">Container 3</el-tag>
            <el-tag type="warning">Container 4</el-tag>

            <div class="flex flex-row justify-between items-center w-full gap-3">
              <el-tag type="danger">Container 5</el-tag>

              <div class="flex flex-row items-center">
                <el-button type="primary" plain circle class="">
                  <el-icon>
                    <Edit />
                  </el-icon>
                </el-button>

                <el-button type="danger" plain circle class="">
                  <el-icon>
                    <Delete />
                  </el-icon>
                </el-button>
              </div>
            </div>

            <div class="flex flex-row justify-between items-center w-full gap-3">
              <el-input v-model="input" placeholder="Please input" class="flex flex-1" />

              <div class="flex flex-row items-center">
                <el-button type="success" plain circle class="">
                  <el-icon>
                    <Check />
                  </el-icon>
                </el-button>

                <el-button type="info" plain circle class="">
                  <el-icon>
                    <Close />
                  </el-icon>
                </el-button>
              </div>
            </div>
          </div>
        </el-card>
      </el-main>

      <el-aside width="400px" class="bg-blue-50 p-4">
        <div class="font-bold mb-2">Environment Variables</div>

        <el-card shadow="never" class="h-[calc(100vh-170px)]">
          <div class="flex flex-row justify-start items-center gap-1">
            <el-button type="danger" circle>
              <el-icon>
                <ChatRound />
              </el-icon>
            </el-button>
            <p class="flex flex-row justify-start items-center gap-1">
              此处引用<el-link type="primary" underline>/api/v1/service/getList</el-link>
            </p>
          </div>
          <p class="pl-9 mb-1">
            在输入框中输入搜索内容，即可在列表中显示搜索结果。
          </p>
          <p class="pl-9 mb-1">
            点新增后可在下方的文本框中快速添加记录，点击确认后即可添加到列表中。
          </p>
          <p class="pl-9 mb-1">
            列表记录中非默认项鼠标悬停时会显示编辑和删除按钮，默认项则不显示。
          </p>
          <p class="pl-9 mb-1">
            新增时校验输入值不能与先前的值重复，编辑时校验输入值不能与除自身以外的其他值重复。
          </p>
        </el-card>
      </el-aside>
    </el-container>
  </div>
</template>

<script setup>
</script>
