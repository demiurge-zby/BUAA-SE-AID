<template>
  <v-container class="mt-8">
    <v-card class="mb-8 pa-6" elevation="2" rounded="lg">
      <v-card-title class="text-h5 font-weight-bold mb-4">全篇论文检测结果</v-card-title>
      <v-card-subtitle class="text-subtitle-1 mb-4">
        检测编号：{{ props.task_id }} | 状态：{{ status === 'completed' ? '已完成' : (status === 'in_progress' ? '检测中...' : '待处理') }}
      </v-card-subtitle>
      
      <v-card-text>
        <v-alert v-if="status !== 'completed'" type="info" variant="tonal">
          正在检测中，请稍候刷新...
        </v-alert>
        
        <v-alert v-if="status === 'completed' && (!results || results.length === 0)" type="warning" variant="tonal">
          未能获取到检测结果，可能是文本内容为空或处理失败。
        </v-alert>

        <v-list v-if="status === 'completed' && results && results.length > 0" lines="three">
          <template v-for="(item, index) in results" :key="index">
            <v-list-item class="mb-4">
              <template v-slot:prepend>
                <v-progress-circular
                  :model-value="item.prob * 100"
                  :color="getColor(item.prob)"
                  size="64"
                  width="8"
                  class="mr-4"
                >
                  <span class="text-caption font-weight-bold">{{ (item.prob * 100).toFixed(1) }}%</span>
                </v-progress-circular>
              </template>
              
              <v-list-item-title class="font-weight-bold mb-2">段落 {{ index + 1 }}</v-list-item-title>
              <v-list-item-subtitle class="text-body-2" style="white-space: pre-wrap; line-height: 1.6;">
                {{ item.text }}
              </v-list-item-subtitle>
              
              <template v-slot:append v-if="item.details && Object.keys(item.details).length">
                <v-tooltip bottom>
                  <template v-slot:activator="{ props }">
                    <v-btn icon="mdi-information-outline" variant="text" v-bind="props"></v-btn>
                  </template>
                  <pre class="text-caption">{{ JSON.stringify(item.details, null, 2) }}</pre>
                </v-tooltip>
              </template>
            </v-list-item>
            <v-divider v-if="index < results.length - 1" class="my-2"></v-divider>
          </template>
        </v-list>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import publisher from '@/api/publisher'
import { useSnackbarStore } from '@/stores/snackbar'

const props = defineProps<{
  task_id: string | number
}>()

const snackbar = useSnackbarStore()
const status = ref('pending')
const results = ref<any[]>([])

const fetchResults = async () => {
  try {
    const res = await publisher.getPaperResults(props.task_id)
    if (res.data) {
      status.value = res.data.status
      results.value = res.data.results || []
    }
  } catch (error) {
    console.error(error)
    snackbar.showMessage('获取论文检测结果失败', 'error')
  }
}

const getColor = (prob: number) => {
  if (prob >= 0.7) return 'error'
  if (prob >= 0.4) return 'warning'
  return 'success'
}

onMounted(() => {
  fetchResults()
})
</script>

<style scoped>
.v-list-item-subtitle {
  -webkit-line-clamp: unset !important;
}
</style>
