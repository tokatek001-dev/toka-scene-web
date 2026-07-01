<template>
  <div class="script flex flex-col h-full px-4 pb-4">
    <!-- Action Bar -->
    <div class="flex items-center justify-between gap-3 mt-5 mb-6 flex-wrap">
      <div class="flex items-center gap-2 flex-wrap">
        <div class="relative">
          <Search :size="16" class="absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground" />
          <Input :placeholder="$t('workbench.script.searchPlaceholder')" v-model="searchQuery" class="pl-9 w-64" />
        </div>
        <Button variant="outline" @click="onChange">{{ $t("workbench.script.search") }}</Button>
        <Button @click="handleAddScript">
          <Plus :size="16" class="mr-2" />{{ $t("workbench.script.addScript") }}
        </Button>
        <Button variant="outline" @click="handleBatchAddScript">
          <Plus :size="16" class="mr-2" />{{ $t("workbench.script.batchAddScript") }}
        </Button>
      </div>
      <div v-if="scripts.length" class="flex items-center gap-2 flex-wrap">
        <Button :variant="isAllSelected ? 'secondary' : 'outline'" size="sm" @click="toggleSelectAll(!isAllSelected)">
          {{ isAllSelected ? $t("workbench.script.cancelSelectAll") : $t("workbench.script.selectAll") }}
        </Button>
        <Button size="sm" variant="outline" :disabled="selectedIds.length === 0" @click="handleExportScript">
          <Download :size="14" class="mr-1" />
          {{ $t("workbench.script.exportScript") }}{{ selectedIds.length ? `(${selectedIds.length})` : "" }}
        </Button>
        <Button size="sm" variant="outline" :disabled="selectedIds.length === 0" @click="handleExtractAssets">
          <Layers :size="14" class="mr-1" />
          {{ $t("workbench.script.extractAssets") }}{{ selectedIds.length ? `(${selectedIds.length})` : "" }}
        </Button>
        <Button size="sm" variant="destructive" :disabled="selectedIds.length === 0" @click="handleBatchDelete">
          <Trash2 :size="14" class="mr-1" />
          {{ $t("workbench.script.deleteScript") }}{{ selectedIds.length ? `(${selectedIds.length})` : "" }}
        </Button>
      </div>
    </div>

    <!-- Content -->
    <div class="flex-1 overflow-y-auto">
      <div v-if="scripts.length === 0" class="flex flex-col items-center justify-center h-64 text-muted-foreground">
        <FileText :size="48" class="mb-4 opacity-30" />
        <p class="text-sm">{{ $t("workbench.script.empty") }}</p>
      </div>
      <div v-else class="flex flex-wrap gap-5">
        <Card
          v-for="(item, index) in scripts"
          :key="item.id"
          class="w-96 cursor-pointer hover:shadow-lg transition-shadow"
          @click="handleScriptClick(item)">
          <CardHeader class="pb-2">
            <div class="flex items-center justify-between">
              <CardTitle class="text-base font-semibold truncate flex-1">{{ item.name }}</CardTitle>
              <Checkbox
                :checked="selectedIds.includes(item.id)"
                @click.stop
                @update:checked="() => toggleSelect(item.id)"
                class="ml-3 shrink-0"
              />
            </div>
          </CardHeader>
          <CardContent>
            <p class="text-sm text-muted-foreground line-clamp-2">{{ item.content }}</p>

            <div v-if="item.extractState === 0" class="flex items-center gap-1 mt-2">
              <svg class="animate-spin h-3 w-3 text-muted-foreground" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              </svg>
              <span class="text-xs text-muted-foreground">{{ $t('workbench.script.msg.extracting') }}</span>
            </div>
            <div v-else-if="item.extractState === 2" class="flex items-center gap-1 mt-2">
              <svg class="animate-spin h-3 w-3 text-muted-foreground" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              </svg>
              <span class="text-xs text-muted-foreground">{{ $t('workbench.script.msg.waitExtract') }}</span>
            </div>
            <Badge v-else-if="item.extractState === -1" variant="destructive" class="mt-2 text-xs">
              {{ $t("workbench.script.msg.extractFailed") }}
            </Badge>
            <div v-else-if="item.relatedAssets?.length" class="flex flex-wrap gap-1 mt-2" @click.stop>
              <Badge v-for="asset in item.relatedAssets" :key="asset.id" variant="secondary" class="text-xs">
                {{ asset.name }}
              </Badge>
            </div>

            <div class="flex justify-end mt-2">
              <Button size="icon" variant="ghost" @click.stop="handleDeleteScript(item.id)">
                <Trash2 :size="16" class="text-muted-foreground hover:text-destructive" />
              </Button>
            </div>
          </CardContent>
        </Card>
      </div>
    </div>

    <editScript v-model="detailsShow" :item="selectedScript" @searchScripts="searchScripts" />
    <addScript v-model="addScriptShow" @searchScripts="searchScripts" />
    <batchAddScript v-model="batchScriptShow" @select="searchScripts" />
  </div>
</template>

<script setup lang="ts">
import axios from "@/utils/axios";
import editScript from "./components/editScript.vue";
import addScript from "./components/addScript.vue";
import batchAddScript from "./components/batchAddScript.vue";
import projectStore from "@/stores/project";
import settingStore from "@/stores/setting";
import imageListCacheStore from "@/stores/imageListCache";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Card from "@/components/ui/Card.vue"
import CardHeader from "@/components/ui/CardHeader.vue"
import CardTitle from "@/components/ui/CardTitle.vue"
import CardContent from "@/components/ui/CardContent.vue";
import Badge from "@/components/ui/Badge.vue";
import Checkbox from "@/components/ui/Checkbox.vue";
import { Plus, Trash2, Search, Download, Layers, FileText } from "lucide-vue-next";

const { clearScriptCache } = imageListCacheStore();
const { otherSetting } = storeToRefs(settingStore());
const { project } = storeToRefs(projectStore());

interface ScriptAsset { id: number; name: string; describe: string; prompt: string; type: "role" | "tool" | "scene" | "clip"; }
interface Script { id: number; name: string; content: string; createTime?: number; extractState?: -1 | 0 | 1 | 2; errorReason?: string; relatedAssets?: ScriptAsset[]; }

const scripts = ref<Script[]>([]);
const searchQuery = ref("");
const addScriptShow = ref(false);
const selectedIds = ref<number[]>([]);
const scriptLoad = ref(false);
const batchScriptShow = ref(false);
const isAllSelected = computed(() => scripts.value.length > 0 && selectedIds.value.length === scripts.value.length);

function toggleSelect(id: number) {
  const idx = selectedIds.value.indexOf(id);
  if (idx === -1) selectedIds.value.push(id);
  else selectedIds.value.splice(idx, 1);
}
function toggleSelectAll(checked: boolean) {
  selectedIds.value = checked ? scripts.value.map((s) => s.id) : [];
}

async function searchScripts() {
  try {
    const res = await axios.post("/script/getScrptApi", { projectId: project.value?.id, name: searchQuery.value });
    scripts.value = res.data;
  } catch (error) {
    window.$message.error($t("workbench.script.msg.searchFailed"));
  }
}
onMounted(searchScripts);

function onChange() { searchScripts(); }
function handleAddScript() { addScriptShow.value = true; }
function handleBatchAddScript() { batchScriptShow.value = true; }

async function handleExportScript() {
  if (!selectedIds.value.length) { window.$message.warning($t("workbench.script.msg.selectsExport")); return; }
  try {
    const res = await axios.post("/script/exportScript", { id: selectedIds.value }, { responseType: "blob" });
    const blob = new Blob([res as any], { type: "application/zip" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url; link.download = `script_${new Date().toISOString().slice(0, 10)}.zip`;
    document.body.appendChild(link); link.click(); document.body.removeChild(link); URL.revokeObjectURL(url);
    window.$message.success($t("workbench.script.msg.exportSuccess"));
  } catch (error) { window.$message.error((error as Error).message ?? $t("workbench.script.msg.exportFailed")); }
}

const selectedScript = ref<Script>({ id: 0, name: "", content: "" });
const detailsShow = ref(false);

function handleScriptClick(item: Script) { selectedScript.value = { ...item }; detailsShow.value = true; }

async function handleDeleteScript(scriptId: number) {
  const dialog = DialogPlugin.confirm({
    header: $t("workbench.script.msg.deleteHeader"),
    body: $t("workbench.script.msg.deleteBody"),
    confirmBtn: $t("workbench.script.msg.deleteConfirm"),
    cancelBtn: $t("workbench.script.msg.cancel"),
    theme: "warning",
    onConfirm: async () => {
      try {
        await axios.post("/script/delScript", { ids: [scriptId] });
        window.$message.success($t("workbench.script.msg.deleteSuccess"));
        clearScriptCache(project.value!.id, scriptId);
        searchScripts();
        selectedIds.value = selectedIds.value.filter((i) => i !== scriptId);
      } catch { window.$message.error($t("workbench.script.msg.deleteFailed")); }
      finally { dialog.destroy(); }
    },
    onClose: () => { dialog.destroy(); },
  });
}

async function handleExtractAssets() {
  if (!project.value) return window.$message.error($t("workbench.script.msg.projectNotFound"));
  scriptLoad.value = true;
  try {
    await axios.post("/script/extractAssets", { scriptIds: selectedIds.value, projectId: project.value!.id, groupSize: otherSetting.value.assetsBatchGenereateSize });
    searchScripts(); selectedIds.value = [];
  } catch (e) { window.$message.error((e as any)?.message || $t("workbench.script.msg.extractFailed")); }
  finally { scriptLoad.value = false; }
}

async function handleBatchDelete() {
  if (!selectedIds.value.length) { window.$message.warning($t("workbench.script.msg.selectDelScript")); return; }
  const extractingIds = new Set(notCompletedData.value.map((s) => s.id));
  if (selectedIds.value.some((id) => extractingIds.has(id))) return window.$message.error($t("workbench.script.msg.extractingInProgress"));
  const dialog = DialogPlugin.confirm({
    header: $t("workbench.script.msg.batchDeleteHeader"),
    body: $t("workbench.script.msg.batchDeleteBody", { count: selectedIds.value.length }),
    confirmBtn: $t("workbench.script.msg.deleteConfirm"),
    cancelBtn: $t("workbench.script.msg.cancel"),
    theme: "warning",
    onConfirm: async () => {
      try {
        await axios.post("/script/delScript", { ids: selectedIds.value });
        window.$message.success($t("workbench.script.msg.batchDeleteSuccess"));
        for (const item of selectedIds.value) clearScriptCache(project.value!.id, item);
        searchScripts();
      } catch { window.$message.error($t("workbench.script.msg.deleteFailed")); }
      finally { selectedIds.value = []; dialog.destroy(); }
    },
    onClose: () => { dialog.destroy(); },
  });
}

const notCompletedData = computed(() => scripts.value.filter((s) => s.extractState == 0));
let pollingTimer: ReturnType<typeof setInterval> | null = null;

async function pollScriptAssets() {
  if (notCompletedData.value.length === 0) return;
  const ids = notCompletedData.value.map((item) => item.id);
  try {
    const { data } = await axios.post("/script/pollScriptAssets", { ids });
    if (data.length) searchScripts();
  } catch (e) { console.error("轮询事件状态失败:", e); }
}

function startPolling() {
  if (pollingTimer) return;
  pollingTimer = setInterval(async () => {
    if (notCompletedData.value.length === 0) { stopPolling(); return; }
    await pollScriptAssets();
  }, 3000);
}
function stopPolling() { if (pollingTimer) { clearInterval(pollingTimer); pollingTimer = null; } }

watch(() => notCompletedData.value, (newVal) => { if (newVal.length > 0) startPolling(); else stopPolling(); });
onUnmounted(() => { stopPolling(); });
</script>
