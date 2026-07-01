<template>
  <div class="novel" ref="novelRef">
    <div class="headBtn jb ac" ref="headBtnRef">
      <div class="flex items-center gap-2">
        <Button @click="importNovelFn">
          <Plus :size="16" class="mr-2" />
          {{ $t("workbench.novel.importText") }}
        </Button>
        <Button variant="destructive" :disabled="selectedRowKeys.length === 0" @click="handleBatchDelete">
          <Trash2 :size="16" class="mr-2" />
          {{ $t("workbench.novel.batchDelete") }} {{ selectedRowKeys.length > 0 ? `(${selectedRowKeys.length})` : "" }}
        </Button>
        <Button variant="outline" @click="startEventAnalysis" :disabled="selectedRowKeys.length === 0">
          <BarChart2 :size="16" class="mr-2" />
          {{ $t("workbench.novel.eventAnalysis") }} {{ selectedRowKeys.length > 0 ? `(${selectedRowKeys.length})` : "" }}
        </Button>
      </div>
      <div class="flex items-center gap-2">
        <div class="relative">
          <Search :size="16" class="absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground" />
          <Input v-model="searchText" :placeholder="$t('workbench.novel.searchPlaceholder')" class="pl-9 w-64" />
        </div>
        <Button variant="outline" @click="onChange">
          {{ $t("workbench.novel.search") }}
        </Button>
      </div>
    </div>

    <div class="mt-4 flex-1 overflow-hidden flex flex-col border border-border rounded-md">
      <div v-if="loading" class="flex items-center justify-center h-64">
        <div class="flex flex-col items-center gap-2">
          <svg class="animate-spin h-6 w-6 text-muted-foreground" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
          <span class="text-sm text-muted-foreground">{{ $t("workbench.novel.generating") }}</span>
        </div>
      </div>
      <Table v-else>
        <TableHeader>
          <TableRow>
            <TableHead class="w-12">
              <Checkbox
                :checked="selectedRowKeys.length === tableData.length && tableData.length > 0"
                @update:checked="(v) => (selectedRowKeys = v ? tableData.map(r => r.id) : [])"
              />
            </TableHead>
            <TableHead class="w-12">{{ $t("workbench.novel.col.id") }}</TableHead>
            <TableHead class="w-24">{{ $t("workbench.novel.col.reel") }}</TableHead>
            <TableHead class="w-32">{{ $t("workbench.novel.col.chapter") }}</TableHead>
            <TableHead>{{ $t("workbench.novel.col.chapterData") }}</TableHead>
            <TableHead>{{ $t("workbench.novel.col.event") }}</TableHead>
            <TableHead class="w-40">{{ $t("workbench.novel.col.operation") }}</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          <TableRow v-for="(row, index) in tableData" :key="row.id" class="hover:bg-muted/50">
            <TableCell>
              <Checkbox
                :checked="selectedRowKeys.includes(row.id)"
                @update:checked="(v) => {
                  if (v) selectedRowKeys.push(row.id); else selectedRowKeys = selectedRowKeys.filter(k => k !== row.id);
                }"
              />
            </TableCell>
            <TableCell class="text-xs text-muted-foreground">{{ row.index }}</TableCell>
            <TableCell class="text-sm">{{ row.reel }}</TableCell>
            <TableCell class="text-sm truncate max-w-[128px]">{{ row.chapter }}</TableCell>
            <TableCell>
              <div class="text-sm text-muted-foreground truncate max-w-[200px]">
                {{ formatPreview(row.chapterData) }}
              </div>
              <Button
                v-if="row.chapterData && row.chapterData.length > PREVIEW_MAX_LENGTH"
                variant="link"
                size="sm"
                class="text-primary h-auto p-0 mt-1"
                @click="openPreview($t('workbench.novel.col.chapterData'), row.chapterData)">
                {{ $t("workbench.novel.viewDetail") }}
              </Button>
            </TableCell>
            <TableCell>
              <div v-if="row.eventState === 0" class="flex items-center gap-1">
                <svg class="animate-spin h-4 w-4 text-muted-foreground" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                </svg>
                <span class="text-xs text-muted-foreground">{{ $t("workbench.novel.generating") }}</span>
              </div>
              <Button
                v-else-if="row.eventState === -1 && !row.event"
                variant="link"
                size="sm"
                class="text-destructive h-auto p-0"
                @click="openPreview($t('workbench.novel.genFailed'), row?.errorReason)">
                {{ $t("workbench.novel.genFailed") }}
              </Button>
              <div v-else class="text-sm text-muted-foreground truncate max-w-[200px]">
                {{ formatPreview(row.event) }}
                <Button
                  v-if="row.event && row.event.length > PREVIEW_MAX_LENGTH"
                  variant="link"
                  size="sm"
                  class="text-primary h-auto p-0 mt-1 ml-1"
                  @click="openPreview($t('workbench.novel.col.event'), row.event)">
                  {{ $t("workbench.novel.viewDetail") }}
                </Button>
              </div>
            </TableCell>
            <TableCell>
              <div class="flex items-center gap-1">
                <Button
                  size="sm"
                  variant="ghost"
                  :disabled="row.eventState === 0"
                  @click="handleEdit(row)">
                  <Edit2 :size="14" />
                </Button>
                <Button
                  size="sm"
                  variant="ghost"
                  :disabled="row.eventState === 0"
                  @click="handleDelete(row)">
                  <Trash2 :size="14" class="text-destructive" />
                </Button>
              </div>
            </TableCell>
          </TableRow>
        </TableBody>
      </Table>
    </div>

    <div v-if="!loading && tableData.length > 0" class="flex items-center justify-between mt-4">
      <p class="text-sm text-muted-foreground">
        {{ $t("workbench.novel.total") }} {{ pagination.total }}
      </p>
      <div class="flex items-center gap-2">
        <Button
          size="sm"
          variant="outline"
          :disabled="pagination.page <= 1"
          @click="pagination.page--; getNovel()">
          <ChevronLeft :size="16" />
        </Button>
        <span class="text-sm text-muted-foreground">
          {{ pagination.page }} / {{ Math.ceil(pagination.total / pagination.pageSize) }}
        </span>
        <Button
          size="sm"
          variant="outline"
          :disabled="pagination.page >= Math.ceil(pagination.total / pagination.pageSize)"
          @click="pagination.page++; getNovel()">
          <ChevronRight :size="16" />
        </Button>
      </div>
    </div>

    <Dialog v-model:open="previewVisible">
      <DialogContent class="max-w-2xl max-h-[80vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle>{{ previewTitle }}</DialogTitle>
        </DialogHeader>
        <div class="text-sm text-foreground whitespace-pre-wrap word-break-break-word py-4">
          {{ previewContent || $t("workbench.novel.none") }}
        </div>
      </DialogContent>
    </Dialog>

    <importNovel v-model="importNovelShow" @select="getNovel" />
    <editNodel v-model="editNodelShow" :formData="formData" @select="getNovel" />
  </div>
</template>

<script setup lang="ts">
import dayjs from "dayjs";
import axios from "@/utils/axios";
import importNovel from "./components/importNovel.vue";
import editNodel from "./components/editNodel.vue";
import projectStore from "@/stores/project";
import settingStore from "@/stores/setting";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Card from "@/components/ui/Card.vue";
import CardHeader from "@/components/ui/CardHeader.vue";
import CardTitle from "@/components/ui/CardTitle.vue";
import CardContent from "@/components/ui/CardContent.vue";
import Badge from "@/components/ui/Badge.vue";
import Checkbox from "@/components/ui/Checkbox.vue";
import { Plus, Trash2, Search, Download, Layers, FileText } from "lucide-vue-next";

const { otherSetting } = storeToRefs(settingStore());
const { project } = storeToRefs(projectStore());

// 搜索文本
const searchText = ref("");

const editNodelShow = ref(false);
interface OriginalText {
  id: number;
  index: number;
  reel: string;
  chapter: string;
  chapterData: string;
  event: string;
  eventState?: number;
  errorReason?: string;
}
const formData = ref<OriginalText>({ id: -1, index: 0, reel: "", chapter: "", chapterData: "", event: "" });
const PREVIEW_MAX_LENGTH = 80;
const previewVisible = ref(false);
const previewTitle = ref("");
const previewContent = ref("");

function formatPreview(text?: string) {
  if (!text) return $t("workbench.novel.none");
  if (text.length <= PREVIEW_MAX_LENGTH) return text;
  return `${text.slice(0, PREVIEW_MAX_LENGTH)}...`;
}

function openPreview(title: string, content?: string) {
  previewTitle.value = title;
  previewContent.value = content || "";
  previewVisible.value = true;
}

// 表格数据
const tableData = ref<OriginalText[]>([]);
// 加载状态
const loading = ref(false);
// 选中行
const selectedRowKeys = ref<Array<string | number>>([]);
// 分页
const pagination = ref({
  page: 1,
  pageSize: 10,
  total: 0,
});

onMounted(() => {
  getNovel();
});

onUnmounted(() => {
  stopPolling();
});

function onChange() {
  pagination.value.page = 1;
  getNovel();
}

function getNovel() {
  loading.value = true;
  axios
    .post("/novel/getNovel", {
      projectId: project.value?.id,
      page: pagination.value.page,
      limit: pagination.value.pageSize,
      search: searchText.value,
    })
    .then((res) => {
      tableData.value = res.data.data;
      pagination.value.total = res.data.total;
    })
    .finally(() => {
      loading.value = false;
    });
}

const importNovelShow = ref(false);

function importNovelFn() {
  importNovelShow.value = true;
}

function handleBatchDelete() {
  if (selectedRowKeys.value.length === 0) return;
  const dialog = DialogPlugin.confirm({
    header: $t("workbench.novel.msg.batchDeleteHeader"),
    body: $t("workbench.novel.msg.batchDeleteBody", { count: selectedRowKeys.value.length }),
    onConfirm: async () => {
      await axios.post("/novel/batchDeleteNovel", {
        ids: selectedRowKeys.value,
      });
      getNovel();
      window.$message.success($t("workbench.novel.msg.batchDeleteSuccess"));
      dialog.destroy();
    },
  });
}

function handleEdit(row: OriginalText) {
  editNodelShow.value = true;
  formData.value = { ...row };
}

function handleDelete(row: OriginalText) {
  const dialog = DialogPlugin.confirm({
    header: $t("workbench.novel.msg.deleteHeader"),
    body: $t("workbench.novel.msg.deleteBody", { name: row.chapter }),
    onConfirm: async () => {
      try {
        await axios.post("/novel/delNovel", { id: row.id });
        window.$message.success($t("workbench.novel.msg.deleteSuccess"));
        if (tableData.value.length === 1 && pagination.value.page > 1) {
          pagination.value.page -= 1;
        }
        getNovel();
      } catch (e) {
        window.$message.error((e as Error).message);
      }
      dialog.destroy();
    },
  });
}

function startEventAnalysis() {
  const dialog = DialogPlugin.confirm({
    header: $t("workbench.novel.msg.eventAnalysisHeader"),
    body: $t("workbench.novel.msg.eventAnalysisBody", { count: selectedRowKeys.value.length }),
    onConfirm: () => {
      dialog.destroy();
      axios
        .post("/novel/event/generateEvents", {
          projectId: project.value?.id!,
          novelIds: selectedRowKeys.value,
          concurrentCount: otherSetting.value.assetsBatchGenereateSize,
        })
        .then((res) => {
          selectedRowKeys.value.length = 0;
          getNovel();
        });
    },
  });
}

const notCompultedData = computed(() => {
  return tableData.value.filter((item) => !item.eventState);
});

// 轮询相关
let pollingTimer: ReturnType<typeof setInterval> | null = null;

async function pollEventState() {
  if (notCompultedData.value.length === 0) return;
  const ids = notCompultedData.value.map((item) => item.id);
  try {
    const { data } = await axios.post("/novel/getNovelEventState", { ids });
    if (Array.isArray(data)) {
      data.forEach((item: { id: number; eventState: number; event?: string; errorReason?: string }) => {
        const target = tableData.value.find((row) => row.id === item.id);
        if (target) {
          target.eventState = item.eventState;
          if (target.eventState == -1) target.errorReason = item.errorReason;
          if (item.event !== undefined) target.event = item.event;
        }
      });
    }
  } catch (e) {
    console.error("轮询事件状态失败:", e);
  }
}

function startPolling() {
  if (pollingTimer) return;
  pollingTimer = setInterval(async () => {
    if (notCompultedData.value.length === 0) {
      stopPolling();
      return;
    }
    await pollEventState();
  }, 3000);
}

function stopPolling() {
  if (pollingTimer) {
    clearInterval(pollingTimer);
    pollingTimer = null;
  }
}

watch(notCompultedData, (val) => {
  if (val.length > 0) {
    startPolling();
  } else {
    stopPolling();
  }
});

onUnmounted(() => {
  stopPolling();
});
</script>

<style lang="scss" scoped>
.novel {
  height: 100%;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  padding-bottom: 16px;
  padding-right: 16px;
  padding-left: 16px;
  
  .headBtn {
    margin-top: 20px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
}
</style>
