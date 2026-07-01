<template>
  <Dialog v-model:open="batchGenerationShow">
    <DialogContent class="max-w-[90vw] max-h-[94vh] flex flex-col overflow-hidden">
      <DialogHeader>
        <DialogTitle>{{ $t('workbench.assets.batch.header') }}</DialogTitle>
      </DialogHeader>
      <div class="flex-1 overflow-hidden flex flex-col gap-3 px-6 py-2">
        <!-- Toolbar -->
        <div class="flex items-center justify-between gap-3 flex-wrap">
          <div class="flex items-center gap-2">
            <span class="text-sm text-muted-foreground">{{ $t('workbench.assets.batch.selected', { count: selectedRowKeys.length }) }}</span>
            <Button size="sm" @click="handleSelectAll">{{ $t('workbench.assets.batch.selectAll') }}</Button>
            <Button variant="outline" size="sm" @click="handleClearSelection">{{ $t('workbench.assets.batch.clearSelection') }}</Button>
          </div>
          <div class="relative w-48">
            <Search :size="14" class="absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground" />
            <Input v-model="searchText" :placeholder="$t('workbench.assets.searchPlaceholder')" class="pl-9" />
          </div>
          <div class="flex items-center gap-2">
            <Button @click="handleBatchGeneratePrompt" :disabled="textLoading">
              <Loader2 v-if="textLoading" :size="16" class="mr-1 animate-spin" />
              <Wand2 v-else :size="16" class="mr-1" />
              {{ $t('workbench.assets.generatePrompt') }}
            </Button>
            <Button @click="handleBatchGenerateImage" :disabled="imageLoading">
              <Loader2 v-if="imageLoading" :size="16" class="mr-1 animate-spin" />
              <ImageIcon v-else :size="16" class="mr-1" />
              {{ $t('workbench.assets.generateImage') }}
            </Button>
          </div>
        </div>

        <!-- Table -->
        <div class="flex-1 overflow-auto border border-border rounded-md">
          <Table>
            <TableHeader class="sticky top-0 bg-background">
              <TableRow>
                <TableHead class="w-12">
                  <Checkbox
                    :checked="selectedRowKeys.length === tableData.length && tableData.length > 0"
                    @update:checked="(v) => (selectedRowKeys = v ? tableData.map(r => r.id) : [])"
                  />
                </TableHead>
                <TableHead class="w-20">{{ $t('workbench.assets.batch.colPreviewImg') }}</TableHead>
                <TableHead class="w-32">{{ $t('workbench.assets.colName') }}</TableHead>
                <TableHead>{{ $t('workbench.assets.colPrompt') }}</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              <TableRow v-for="row in tableData" :key="row.id" class="hover:bg-muted/50">
                <TableCell>
                  <Checkbox
                    :checked="selectedRowKeys.includes(row.id)"
                    @update:checked="(v) => { if (v) selectedRowKeys.push(row.id); else selectedRowKeys = selectedRowKeys.filter(k => k !== row.id); }"
                  />
                </TableCell>
                <TableCell>
                  <div class="previewCell flex items-center justify-center h-16">
                    <div v-if="row.filePath" class="imageTrigger relative w-14 h-14 rounded overflow-hidden border border-border cursor-pointer group">
                      <img :src="row.filePath" :alt="row.name" class="w-full h-full object-cover" />
                      <div class="absolute inset-0 bg-black/60 opacity-0 group-hover:opacity-100 flex items-center justify-center transition-opacity">
                        <Eye :size="16" class="text-white" />
                      </div>
                    </div>
                    <div v-else class="noImage w-14 h-14 rounded bg-muted flex items-center justify-center">
                      <ImageIcon :size="20" class="text-muted-foreground" />
                    </div>
                  </div>
                </TableCell>
                <TableCell class="text-sm truncate max-w-[128px]">{{ row.name }}</TableCell>
                <TableCell>
                  <textarea
                    v-model="row.prompt"
                    :placeholder="$t('workbench.assets.batch.inputPh')"
                    class="w-full min-h-[60px] rounded-md border border-input bg-background px-2 py-1 text-sm resize-none"
                  />
                </TableCell>
              </TableRow>
            </TableBody>
          </Table>
        </div>
      </div>

      <DialogFooter>
        <Button variant="outline" @click="handleCancel">{{ $t('workbench.assets.cancelBtn') }}</Button>
        <Button @click="onConfirm" :disabled="selectedRowKeys.length === 0">{{ $t('workbench.assets.batch.saveSelected', { count: selectedRowKeys.length }) }}</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import settingStore from "@/stores/setting";
const { otherSetting } = storeToRefs(settingStore());
import axios from "@/utils/axios";
import projectStore from "@/stores/project";
const { project } = storeToRefs(projectStore());
import Dialog from "@/components/ui/Dialog.vue";
import DialogContent from "@/components/ui/DialogContent.vue";
import DialogHeader from "@/components/ui/DialogHeader.vue";
import DialogTitle from "@/components/ui/DialogTitle.vue";
import DialogFooter from "@/components/ui/DialogFooter.vue";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Table from "@/components/ui/Table.vue"
import TableHeader from "@/components/ui/TableHeader.vue"
import TableBody from "@/components/ui/TableBody.vue"
import TableRow from "@/components/ui/TableRow.vue"
import TableHead from "@/components/ui/TableHead.vue"
import TableCell from "@/components/ui/TableCell.vue";
import Checkbox from "@/components/ui/Checkbox.vue";
import { Search, ImageIcon, Eye, Loader2, Wand2 } from "lucide-vue-next";

const batchGenerationShow = defineModel<boolean>({ default: false });

interface AssetItem {
  id: number;
  name: string;
  prompt: string;
  type?: string;
  describe?: string;
  filePath?: string;
  remark?: string;
}

const tableData = ref<AssetItem[]>([]);
const localData = ref<AssetItem[]>([]);
const selectedRowKeys = ref<number[]>([]);
const searchText = ref("");
const loading = ref(false);
const textLoading = ref(false);
const imageLoading = ref(false);
const promptGenerateCancel = ref(false);
const imageGenerateCancel = ref(false);
const rowPromptLoading = ref<Record<number, boolean>>({});
const rowImageLoading = ref<Record<number, boolean>>({});

const pagination = ref({ current: 1, pageSize: 10, total: 0 });

const props = defineProps<{ type: "role" | "tool" | "scene" | "clip" }>();

watch(
  () => batchGenerationShow.value,
  (newVal) => {
    if (newVal) {
      localData.value = [];
      handlePageChange({ current: 1, pageSize: pagination.value.pageSize });
    }
  }
);

watch(
  () => searchText.value,
  () => {
    if (batchGenerationShow.value) {
      handlePageChange({ current: 1, pageSize: pagination.value.pageSize });
    }
  }
);

function handleSelectChange(value: any[]) { selectedRowKeys.value = value; }
function handleSelectAll() { selectedRowKeys.value = tableData.value.map((item) => item.id); }
function handleClearSelection() { selectedRowKeys.value = []; }

async function handlePageChange(pageInfo: { current: number; pageSize: number }) {
  pagination.value.current = pageInfo.current;
  pagination.value.pageSize = pageInfo.pageSize;
  try {
    loading.value = true;
    const { data } = await axios.post("/assets/batchGenerationData", {
      projectId: project.value?.id,
      type: props.type,
      name: searchText.value || undefined,
      page: pageInfo.current,
      limit: pageInfo.pageSize,
    });
    tableData.value = data.data || [];
    localData.value = JSON.parse(JSON.stringify(tableData.value));
    pagination.value.total = data.total || 0;
  } catch (error) {
    console.error("加载资产数据失败:", error);
    tableData.value = [];
    pagination.value.total = 0;
  } finally {
    loading.value = false;
  }
}

function closeModal() {
  promptGenerateCancel.value = true;
  imageGenerateCancel.value = true;
  batchGenerationShow.value = false;
  selectedRowKeys.value = [];
  searchText.value = "";
}

function handleCancel() { closeModal(); }

const emit = defineEmits(["update"]);

async function onConfirm() {
  if (selectedRowKeys.value.length === 0) {
    window.$message.warning($t('workbench.assets.selectAtLeastOne'));
    return;
  }
  const selectedAssets = tableData.value.filter((item) => selectedRowKeys.value.includes(item.id));
  if (selectedAssets.length === 0) {
    window.$message.error($t('workbench.assets.batch.selectToSave'));
    return;
  }
  try {
    const batchSize = otherSetting.value.assetsBatchGenereateSize || 5;
    for (let i = 0; i < selectedAssets.length; i += batchSize) {
      await Promise.all(
        selectedAssets.slice(i, i + batchSize).map((item) =>
          axios.post("/assets/updateAssets", {
            id: item.id,
            name: item.name,
            describe: item.describe ?? "",
            type: item.type,
            remark: item.remark ?? "",
            prompt: item.prompt,
          })
        )
      );
    }
    window.$message.success($t('workbench.assets.batch.saveSuccess'));
    emit("update");
    closeModal();
  } catch (error) {
    console.error("保存失败:", error);
    window.$message.error($t('workbench.assets.batch.saveFail'));
  }
}

async function handleBatchGeneratePrompt() {
  const selectedAssets = tableData.value.filter((item) => selectedRowKeys.value.includes(item.id));
  if (selectedAssets.length === 0) {
    window.$message.error($t('workbench.assets.selectAtLeastOne'));
    return;
  }
  promptGenerateCancel.value = false;
  textLoading.value = true;
  const batchSize = otherSetting.value.assetsBatchGenereateSize || 5;
  try {
    for (let i = 0; i < selectedAssets.length; i += batchSize) {
      if (promptGenerateCancel.value) throw new Error($t('workbench.assets.batch.promptGenCancelled'));
      const batch = selectedAssets.slice(i, i + batchSize);
      await Promise.allSettled(batch.map((item) => generatePrompt(item)));
    }
    window.$message.success($t('workbench.assets.batch.promptDone'));
  } catch (e) {
    if (e instanceof Error && e.message !== $t('workbench.assets.batch.promptGenCancelled')) {
      window.$message.error(e.message);
    }
  } finally {
    textLoading.value = false;
  }
}

async function generatePrompt(data: AssetItem) {
  rowPromptLoading.value[data.id] = true;
  try {
    const res = await axios.post("/assets/polishAssetsPrompt", {
      projectId: project.value?.id,
      assetsId: data.id,
      type: props.type ?? "props",
      name: data.name,
      describe: data.describe ?? "",
    });
    const index = tableData.value.findIndex((item) => item.id === res.data.assetsId);
    if (index !== -1 && !promptGenerateCancel.value) {
      tableData.value[index].prompt = res.data.prompt;
    }
  } catch (e: any) {
    window.$message.error(`"${data.name}" ${e?.message ?? $t('workbench.assets.batch.promptFail')}`);
  } finally {
    rowPromptLoading.value[data.id] = false;
  }
}

async function handleBatchGenerateImage() {
  const selectedAssets = tableData.value.filter((item) => selectedRowKeys.value.includes(item.id));
  if (selectedAssets.length === 0) {
    window.$message.warning($t('workbench.assets.selectAtLeastOne'));
    return;
  }
  const assetsWithoutPrompt = selectedAssets.filter((item) => !item.prompt || item.prompt.trim() === "");
  if (assetsWithoutPrompt.length > 0) {
    window.$message.warning($t('workbench.assets.batch.missingPrompts', { count: assetsWithoutPrompt.length }));
    return;
  }
  imageGenerateCancel.value = false;
  imageLoading.value = true;
  const batchSize = otherSetting.value.assetsBatchGenereateSize || 5;
  try {
    for (let i = 0; i < selectedAssets.length; i += batchSize) {
      if (imageGenerateCancel.value) throw new Error($t('workbench.assets.batch.promptGenCancelled'));
      const batch = selectedAssets.slice(i, i + batchSize);
      await Promise.allSettled(batch.map((item) => startGenerate({ id: item.id, name: item.name, prompt: item.prompt, type: props.type ?? "props" })));
    }
    window.$message.success($t('workbench.assets.batch.imageDone'));
  } catch (e) {
    if (e instanceof Error && e.message !== $t('workbench.assets.batch.promptGenCancelled')) {
      window.$message.error(e.message);
    }
  } finally {
    imageLoading.value = false;
  }
}

async function startGenerate(data: { id: number; prompt: string; name: string; type: string }) {
  if (imageGenerateCancel.value) return;
  rowImageLoading.value[data.id] = true;
  try {
    const res = await axios.post("/assets/generateAssets", {
      type: data.type,
      projectId: project.value?.id,
      name: data.name,
      base64: undefined,
      prompt: data.prompt ?? "",
      id: data.id,
    });
    if (!imageGenerateCancel.value) {
      const index = tableData.value.findIndex((item) => item.id === res.data.assetsId);
      if (index !== -1) {
        tableData.value[index].filePath = res.data.path;
      }
    }
  } catch (e: any) {
    if (!imageGenerateCancel.value) {
      window.$message.error(`"${data.name}" ${$t('workbench.assets.batch.imageGenFail')}: ${e?.message ?? $t('workbench.assets.batch.unknownError')}`);
    }
  } finally {
    rowImageLoading.value[data.id] = false;
  }
}
</script>
