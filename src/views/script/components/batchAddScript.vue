<template>
  <Dialog v-model:open="purgeNovelShow">
    <DialogContent class="max-w-2xl h-[80vh] flex flex-col overflow-hidden p-0">
      <DialogHeader class="px-6 pt-6 pb-0 shrink-0">
        <DialogTitle>{{ $t('workbench.script.import.batchTitle') }}</DialogTitle>
      </DialogHeader>

      <!-- Step tabs -->
      <div class="flex border-b border-border mx-6 mt-4 shrink-0">
        <button
          v-for="(step, i) in ['To1', 'To2']"
          :key="step"
          class="px-4 py-2 text-sm font-medium border-b-2 transition-colors"
          :class="activeKey === step ? 'border-primary text-primary' : 'border-transparent text-muted-foreground opacity-50 cursor-default'"
        >
          {{ i + 1 }}. {{ $t(i === 0 ? 'workbench.novel.import.step1' : 'workbench.novel.import.step2') }}
        </button>
      </div>

      <!-- Step 1 -->
      <div v-if="activeKey === 'To1'" class="flex-1 overflow-y-auto px-6 py-4 flex flex-col gap-4">
        <!-- Regex row -->
        <div class="flex items-center gap-2">
          <span class="text-sm font-medium shrink-0">{{ $t("workbench.script.import.episodeRegex") }}</span>
          <div class="flex-1 relative">
            <Input
              v-model="customRegStr"
              :placeholder="$t('workbench.script.import.episodeRegexPh')"
              :disabled="aiRegexLoading"
              :class="regexError ? 'border-destructive' : ''"
            />
            <p v-if="regexError" class="text-xs text-destructive mt-1">{{ regexError }}</p>
          </div>
          <Button size="sm" variant="outline" @click="getAiRegex" :disabled="aiRegexLoading">
            <svg v-if="aiRegexLoading" class="animate-spin h-4 w-4 mr-1" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            </svg>
            {{ $t("workbench.script.import.getAiRegex") }}
          </Button>
        </div>

        <!-- Drop area -->
        <div
          class="border-2 border-dashed border-border rounded-lg p-8 text-center cursor-pointer transition-colors hover:border-primary"
          @click="triggerUpload"
          @dragover.prevent
          @drop.prevent="handleDrop">
          <input ref="fileInputRef" type="file" accept=".txt,.docx" class="hidden" @change="handleFileChange" />
          <Upload :size="28" class="mx-auto mb-3 text-muted-foreground" />
          <p class="text-sm font-medium">{{ $t("workbench.script.add.dragUpload") }}</p>
          <p class="text-xs text-muted-foreground mt-1">{{ $t("workbench.novel.import.uploadHint") }}</p>
        </div>

        <div class="flex items-center gap-3">
          <div class="flex-1 h-px bg-border" /><span class="text-xs text-muted-foreground">{{ $t("workbench.novel.import.or") }}</span><div class="flex-1 h-px bg-border" />
        </div>

        <div class="flex flex-col gap-2">
          <Label>{{ $t("workbench.script.import.pasteLabel") }}</Label>
          <Textarea v-model="content" :placeholder="$t('workbench.script.add.scriptContentPh')" class="min-h-[180px] resize-none" />
          <div class="flex items-center justify-between text-xs text-muted-foreground">
            <div class="flex items-center gap-2">
              <span>{{ content.length }} {{ $t("workbench.novel.import.chars") }}</span>
              <span v-if="content.length > 0 && content.length < 100" class="text-yellow-500">{{ $t("workbench.novel.import.tooShort") }}</span>
            </div>
            <span>{{ $t("workbench.script.import.parsedChapters", { count: tableData.length }) }}</span>
          </div>
        </div>

        <div class="flex justify-end">
          <Button :disabled="!content || !tableData.length" @click="activeKey = 'To2'">
            {{ $t("workbench.novel.import.nextStep") }}<ChevronRight :size="16" class="ml-1" />
          </Button>
        </div>
      </div>

      <!-- Step 2 -->
      <div v-if="activeKey === 'To2'" class="flex-1 overflow-hidden flex flex-col px-6 py-4 gap-3">
        <div class="rounded-md border border-border overflow-auto flex-1">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead class="w-12">
                  <Checkbox
                    :checked="selectedRowKeys.length === tableData.length && tableData.length > 0"
                    @update:checked="(v) => (selectedRowKeys = v ? tableData.map(r => r.index) : [])"
                  />
                </TableHead>
                <TableHead class="w-16">{{ $t("workbench.script.import.col.chapter") }}</TableHead>
                <TableHead class="w-48">{{ $t("workbench.script.import.col.scriptName") }}</TableHead>
                <TableHead>{{ $t("workbench.script.import.col.scriptData") }}</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              <TableRow v-for="row in tableData" :key="row.index" class="hover:bg-muted/50">
                <TableCell>
                  <Checkbox
                    :checked="selectedRowKeys.includes(row.index)"
                    @update:checked="(v) => { if (v) selectedRowKeys.push(row.index); else selectedRowKeys = selectedRowKeys.filter(k => k !== row.index); }"
                  />
                </TableCell>
                <TableCell class="text-xs">{{ row.index }}</TableCell>
                <TableCell class="text-sm truncate max-w-[192px]">{{ row.scriptName }}</TableCell>
                <TableCell class="text-xs text-muted-foreground truncate max-w-[200px]">{{ row.scriptData }}</TableCell>
              </TableRow>
            </TableBody>
          </Table>
        </div>
        <p class="text-sm text-muted-foreground shrink-0">{{ $t("workbench.novel.import.selectedInfo", { count: selectedTextLength }) }}</p>
        <div class="flex justify-between shrink-0">
          <Button variant="outline" @click="activeKey = 'To1'"><ChevronLeft :size="16" class="mr-1" />{{ $t("workbench.novel.import.prevStep") }}</Button>
          <Button :disabled="selectedTextLength > otherSetting.scriptEpisodeLength" :loading="nextLoading" @click="keep">保存</Button>
        </div>
      </div>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import settingStore from "@/stores/setting";
const { otherSetting } = storeToRefs(settingStore());
import axios from "@/utils/axios";
import parseScript from "@/utils/parseScript";
import mammoth from "mammoth";
import projectStore from "@/stores/project";
const { project } = storeToRefs(projectStore());
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import Textarea from "@/components/ui/Textarea.vue";
import Dialog from "@/components/ui/Dialog.vue"
import DialogContent from "@/components/ui/DialogContent.vue"
import DialogHeader from "@/components/ui/DialogHeader.vue"
import DialogTitle from "@/components/ui/DialogTitle.vue";
import Table from "@/components/ui/Table.vue"
import TableHeader from "@/components/ui/TableHeader.vue"
import TableBody from "@/components/ui/TableBody.vue"
import TableRow from "@/components/ui/TableRow.vue"
import TableHead from "@/components/ui/TableHead.vue"
import TableCell from "@/components/ui/TableCell.vue";
import Checkbox from "@/components/ui/Checkbox.vue";
import { Upload, ChevronRight, ChevronLeft } from "lucide-vue-next";

interface ChapterItem { index: number; scriptName: string; scriptData: string; }

const purgeNovelShow = defineModel<boolean>();
const activeKey = ref("To1");
const fileInputRef = ref<HTMLInputElement | null>(null);
const content = ref("");
const selectedRowKeys = ref<number[]>([]);
const nextLoading = ref(false);
const customRegStr = ref("");
const regexError = ref("");
const aiRegexLoading = ref(false);

watch(customRegStr, (val) => {
  if (!val.trim()) { regexError.value = ""; return; }
  try {
    const m = val.match(/^\/(.*)\/([ igmuy]*)$/);
    new RegExp(m ? m[1] : val); regexError.value = "";
  } catch { regexError.value = $t("workbench.script.import.regexInvalid"); }
});

const tableData = computed<ChapterItem[]>(() => {
  if (!content.value) return [];
  try {
    return parseScript(content.value, customRegStr.value || undefined).map((ep) => ({ index: ep.index, scriptName: ep.chapter, scriptData: ep.text }));
  } catch { return []; }
});

const selectedRows = computed(() => tableData.value.filter((item) => selectedRowKeys.value.includes(item.index)));
const selectedTextLength = computed(() => selectedRows.value.reduce((sum, item) => sum + item.scriptData.length, 0));

function triggerUpload() { fileInputRef.value?.click(); }
async function handleFileChange(e: Event) { const file = (e.target as HTMLInputElement).files?.[0]; if (file) await processFile(file); }
async function handleDrop(e: DragEvent) { const file = e.dataTransfer?.files?.[0]; if (file) await processFile(file); }

async function processFile(rawFile: File) {
  const allowTypes = ["text/plain", "application/vnd.openxmlformats-officedocument.wordprocessingml.document"];
  if (rawFile.type === "application/msword") { window.$message.warning($t("workbench.novel.import.msg.docNotSupported")); return; }
  if (!allowTypes.includes(rawFile.type)) { window.$message.error($t("workbench.novel.import.msg.unsupportedType")); return; }
  if (rawFile.size > 10 * 1024 * 1024) { window.$message.error($t("workbench.novel.import.msg.fileTooLarge")); return; }
  try {
    const buffer = await rawFile.arrayBuffer();
    content.value = rawFile.type === "text/plain" ? new TextDecoder().decode(buffer) : (await mammoth.extractRawText({ arrayBuffer: buffer })).value;
  } catch { window.$message.error($t("workbench.novel.import.msg.parseFailed")); }
}

const emit = defineEmits(["select"]);
async function keep() {
  nextLoading.value = true;
  if (!selectedRows.value.length) { window.$message.warning($t("workbench.script.import.msg.selectChapters")); nextLoading.value = false; return; }
  try {
    await axios.post("/script/batchAddScript", { projectId: project.value?.id, data: selectedRows.value });
    emit("select"); window.$message.success($t("workbench.script.import.msg.saveSuccess")); purgeNovelShow.value = false;
  } catch (e) { window.$message.error((e as Error).message); }
  finally { nextLoading.value = false; }
}

watch(purgeNovelShow, (newVal) => {
  if (!newVal) { content.value = ""; selectedRowKeys.value = []; activeKey.value = "To1"; customRegStr.value = ""; regexError.value = ""; }
});

async function getAiRegex() {
  if (!content.value.trim()) { window.$message.warning($t("workbench.script.import.msg.selectChapters")); return; }
  aiRegexLoading.value = true;
  try {
    const { data } = await axios.post("/script/getAiRegex", { content: content.value.slice(0, 2000) });
    if (data) customRegStr.value = data;
  } catch (e) { window.$message.error((e as Error).message); }
  finally { aiRegexLoading.value = false; }
}
</script>
