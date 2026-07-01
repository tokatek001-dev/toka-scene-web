<template>
  <Dialog v-model:open="purgeNovelShow">
    <DialogContent class="max-w-2xl h-[80vh] flex flex-col overflow-hidden p-0">
      <DialogHeader class="px-6 pt-6 pb-0 shrink-0">
        <DialogTitle>{{ $t('workbench.novel.import.title') }}</DialogTitle>
      </DialogHeader>

      <!-- Step tabs header -->
      <div class="flex border-b border-border mx-6 mt-4 shrink-0">
        <button
          v-for="(step, i) in steps"
          :key="step"
          class="px-4 py-2 text-sm font-medium border-b-2 transition-colors"
          :class="activeKey === step
            ? 'border-primary text-primary'
            : 'border-transparent text-muted-foreground opacity-50 cursor-default'"
        >
          {{ i + 1 }}. {{ $t(step === 'To1' ? 'workbench.novel.import.step1' : 'workbench.novel.import.step2') }}
        </button>
      </div>

      <!-- Step 1 -->
      <div v-if="activeKey === 'To1'" class="flex-1 overflow-y-auto px-6 py-4 flex flex-col gap-4">
        <!-- Drop area -->
        <div
          class="border-2 border-dashed border-border rounded-lg p-10 text-center cursor-pointer transition-colors hover:border-primary"
          @click="triggerUpload"
          @dragover.prevent
          @drop.prevent="handleDrop">
          <input ref="fileInputRef" type="file" accept=".txt,.docx" class="hidden" @change="handleFileChange" />
          <Upload :size="32" class="mx-auto mb-3 text-muted-foreground" />
          <p class="text-sm font-medium">{{ $t("workbench.novel.import.dragUpload") }}</p>
          <p class="text-xs text-muted-foreground mt-1">{{ $t("workbench.novel.import.uploadHint") }}</p>
        </div>

        <div class="flex items-center gap-3">
          <div class="flex-1 h-px bg-border" />
          <span class="text-xs text-muted-foreground">{{ $t("workbench.novel.import.or") }}</span>
          <div class="flex-1 h-px bg-border" />
        </div>

        <div class="flex flex-col gap-2">
          <Label>{{ $t("workbench.novel.import.pasteLabel") }}</Label>
          <Textarea
            v-model="content"
            :placeholder="$t('workbench.novel.import.pastePlaceholder')"
            class="min-h-[200px] resize-none"
          />
          <div class="flex items-center justify-between text-xs text-muted-foreground">
            <div class="flex items-center gap-2">
              <span>{{ content.length }} {{ $t("workbench.novel.import.chars") }}</span>
              <span v-if="content.length > 0 && content.length < 100" class="text-yellow-500">
                {{ $t("workbench.novel.import.tooShort") }}
              </span>
            </div>
            <span>{{ $t("workbench.novel.import.parsedChapters", { count: tableData.length }) }}</span>
          </div>
        </div>

        <div class="flex justify-end">
          <Button :disabled="!content || !tableData.length" @click="activeKey = 'To2'">
            {{ $t("workbench.novel.import.nextStep") }}
            <ChevronRight :size="16" class="ml-1" />
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
                <TableHead class="w-16">{{ $t("workbench.novel.import.col.chapter") }}</TableHead>
                <TableHead class="w-20">{{ $t("workbench.novel.import.col.reel") }}</TableHead>
                <TableHead class="w-40">{{ $t("workbench.novel.import.col.chapterName") }}</TableHead>
                <TableHead>{{ $t("workbench.novel.import.col.chapterData") }}</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              <TableRow v-for="row in tableData" :key="row.index" class="hover:bg-muted/50">
                <TableCell>
                  <Checkbox
                    :checked="selectedRowKeys.includes(row.index)"
                    @update:checked="(v) => {
                      if (v) selectedRowKeys.push(row.index);
                      else selectedRowKeys = selectedRowKeys.filter(k => k !== row.index);
                    }"
                  />
                </TableCell>
                <TableCell class="text-xs">{{ row.index }}</TableCell>
                <TableCell class="text-sm">{{ row.reel }}</TableCell>
                <TableCell class="text-sm truncate max-w-[160px]">{{ row.chapter }}</TableCell>
                <TableCell class="text-xs text-muted-foreground truncate max-w-[200px]">{{ row.chapterData }}</TableCell>
              </TableRow>
            </TableBody>
          </Table>
        </div>
        <p class="text-sm text-muted-foreground shrink-0">
          {{ $t("workbench.novel.import.selectedInfo", { count: selectedTextLength }) }}
        </p>
        <div class="flex justify-between shrink-0">
          <Button variant="outline" @click="activeKey = 'To1'">
            <ChevronLeft :size="16" class="mr-1" />
            {{ $t("workbench.novel.import.prevStep") }}
          </Button>
          <Button :loading="nextLoading" @click="keep">保存</Button>
        </div>
      </div>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import axios from "@/utils/axios";
import parseNovel from "@/utils/parseNovel";
import mammoth from "mammoth";
import Button from "@/components/ui/Button.vue";
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
import projectStore from "@/stores/project";

const { project } = storeToRefs(projectStore());

interface ChapterItem {
  index: number;
  reel: string;
  chapter: string;
  chapterData: string;
}

const purgeNovelShow = defineModel<boolean>();

const steps = ["To1", "To2"];
const activeKey = ref("To1");
const fileInputRef = ref<HTMLInputElement | null>(null);
const content = ref("");
const selectedRowKeys = ref<number[]>([]);
const nextLoading = ref(false);

const tableData = computed<ChapterItem[]>(() => {
  if (!content.value) return [];
  try {
    return parseNovel(content.value).flatMap((reel) =>
      reel.chapters.map((chapter) => ({
        index: chapter.index,
        reel: reel.reel,
        chapter: chapter.chapter,
        chapterData: chapter.text,
      })),
    );
  } catch (e) {
    console.error("解析小说内容出错:", e);
    return [];
  }
});

const selectedRows = computed(() => tableData.value.filter((item) => selectedRowKeys.value.includes(item.index)));
const selectedTextLength = computed(() => selectedRows.value.reduce((sum, item) => sum + item.chapterData.length, 0));

function triggerUpload() {
  fileInputRef.value?.click();
}

async function handleFileChange(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (file) await processFile(file);
}

async function handleDrop(e: DragEvent) {
  const file = e.dataTransfer?.files?.[0];
  if (file) await processFile(file);
}

async function processFile(rawFile: File) {
  const allowTypes = ["text/plain", "application/vnd.openxmlformats-officedocument.wordprocessingml.document"];
  if (rawFile.type === "application/msword") {
    window.$message.warning($t("workbench.novel.import.msg.docNotSupported"));
    return;
  }
  if (!allowTypes.includes(rawFile.type)) {
    window.$message.error($t("workbench.novel.import.msg.unsupportedType"));
    return;
  }
  if (rawFile.size > 10 * 1024 * 1024) {
    window.$message.error($t("workbench.novel.import.msg.fileTooLarge"));
    return;
  }
  try {
    const buffer = await rawFile.arrayBuffer();
    if (rawFile.type === "text/plain") {
      content.value = new TextDecoder().decode(buffer);
    } else {
      const result = await mammoth.extractRawText({ arrayBuffer: buffer });
      content.value = result.value;
    }
  } catch {
    window.$message.error($t("workbench.novel.import.msg.parseFailed"));
  }
}

const emit = defineEmits(["select"]);

async function keep() {
  nextLoading.value = true;
  if (!selectedRows.value.length) {
    window.$message.warning($t("workbench.novel.import.msg.selectChapters"));
    nextLoading.value = false;
    return;
  }
  try {
    await axios.post("/novel/addNovel", { projectId: project.value?.id, data: selectedRows.value });
    emit("select");
    window.$message.success($t("workbench.novel.import.msg.saveSuccess"));
  } catch (e) {
    window.$message.error((e as Error).message);
  } finally {
    nextLoading.value = false;
    purgeNovelShow.value = false;
  }
}

watch(purgeNovelShow, (newVal) => {
  if (!newVal) {
    content.value = "";
    selectedRowKeys.value = [];
    activeKey.value = "To1";
  }
});
</script>
