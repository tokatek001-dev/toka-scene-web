<template>
  <Dialog :open="generateImageShow" @update:open="(v) => { if (!v) handleCancel() }">
    <DialogContent class="max-w-[80vw] max-h-[92vh] flex flex-col overflow-hidden">
      <DialogHeader>
        <DialogTitle>{{ $t('workbench.assets.gen.header') }}</DialogTitle>
      </DialogHeader>

      <div class="flex flex-1 overflow-hidden gap-5 py-2">
        <!-- Left panel -->
        <div class="w-[40%] overflow-y-auto pr-2 flex flex-col gap-5">
          <!-- Upload reference -->
          <div class="space-y-2">
            <div class="flex items-center justify-between">
              <span class="text-base font-bold">{{ $t("workbench.assets.gen.uploadRef") }}</span>
              <Badge variant="secondary" class="text-xs">{{ $t("workbench.assets.gen.optional") }}</Badge>
            </div>
            <div
              class="border-2 border-dashed border-border rounded-lg p-6 text-center cursor-pointer hover:border-primary transition-colors"
              @click="refFileInput?.click()"
              @dragover.prevent
              @drop.prevent="handleRefDrop">
              <input ref="refFileInput" type="file" accept="image/*" class="hidden" @change="handleRefFileChange" />
              <template v-if="referenceFile">
                <ImageIcon :size="20" class="mx-auto mb-1 text-primary" />
                <p class="text-sm font-medium text-primary">{{ referenceFile.name }}</p>
                <button class="text-xs text-muted-foreground mt-1 hover:text-destructive" @click.stop="referenceFile = null">{{ $t("workbench.assets.gen.remove") }}</button>
              </template>
              <template v-else>
                <Upload :size="20" class="mx-auto mb-1 text-muted-foreground" />
                <p class="text-sm text-muted-foreground">{{ $t("workbench.assets.gen.uploadHint") }}</p>
              </template>
            </div>
          </div>

          <!-- Prompt -->
          <div class="space-y-2">
            <div class="flex items-center justify-between">
              <span class="text-base font-bold">{{ $t("workbench.assets.gen.promptLabel") }}</span>
              <button class="flex items-center gap-1 text-sm text-primary hover:opacity-80" @click="generatePrompt" :disabled="promptLoading">
                <Loader2 v-if="promptLoading" :size="14" class="animate-spin" />
                <Wand2 v-else :size="14" />
                {{ $t("workbench.assets.gen.smartGenerate") }}
              </button>
            </div>
            <div class="relative">
              <div v-if="promptLoading" class="absolute inset-0 flex flex-col items-center justify-center bg-background/80 z-10 rounded-md">
                <Loader2 :size="24" class="animate-spin text-primary mb-2" />
                <span class="text-sm text-muted-foreground">{{ $t('workbench.assets.gen.generatingPrompt') }}</span>
              </div>
              <Textarea
                v-model="promptValue"
                :placeholder="$t('workbench.assets.gen.promptPlaceholder')"
                :disabled="generateLoading"
                class="min-h-[220px] resize-none" />
            </div>
          </div>

          <!-- Model + resolution -->
          <div class="flex gap-3">
            <div class="flex-[3] space-y-1">
              <span class="text-base font-bold">{{ $t("workbench.assets.gen.selectModel") }}</span>
              <modelSelect v-model="selectValue" :type="`image`" />
            </div>
            <div class="flex-[2] space-y-1">
              <span class="text-base font-bold">{{ $t("workbench.assets.gen.selectResolution") }}</span>
              <select v-model="resolution" class="flex h-9 w-full rounded-md border border-input bg-transparent px-3 py-1 text-sm focus:outline-none focus:ring-1 focus:ring-ring mt-1">
                <option value="1K">1K</option>
                <option value="2K">2K</option>
                <option value="4K">4K</option>
              </select>
            </div>
          </div>

          <Button class="w-full" size="lg" :disabled="generateLoading" @click="handleGenerate">
            <Loader2 v-if="generateLoading" :size="16" class="mr-2 animate-spin" />
            {{ $t("workbench.assets.gen.generateBtn") }}
          </Button>
        </div>

        <!-- Divider -->
        <div class="w-px bg-border self-stretch shrink-0" />

        <!-- Right panel -->
        <div class="w-[60%] flex flex-col overflow-hidden gap-3">
          <div class="flex items-center justify-between">
            <span class="text-base font-bold">{{ $t('workbench.assets.gen.resultTitle') }}</span>
            <Badge v-if="resultImages.length" variant="secondary">{{ $t("workbench.assets.gen.generatedCount", { count: resultImages.length }) }}</Badge>
          </div>

          <div class="flex-1 overflow-y-auto">
            <div class="flex flex-wrap gap-3 pb-2">
              <div
                v-for="(img, index) in resultImages"
                :key="index"
                class="w-44 h-44 rounded-[20px] relative border-2 cursor-pointer transition-all"
                :class="{
                  'border-foreground': selectedImageIndex === index && img.state === '已完成',
                  'border-border': selectedImageIndex !== index,
                  'cursor-not-allowed opacity-80': img.state !== '已完成',
                }"
                @click="img.state === '已完成' ? selectImage(index) : null"
                @mouseenter="hoveredImageIndex = index"
                @mouseleave="hoveredImageIndex = null">
                <!-- Generating -->
                <div v-if="img.state === '生成中'" class="absolute inset-0 bg-white/95 rounded-[20px] flex flex-col items-center justify-center z-10 gap-2">
                  <Loader2 :size="24" class="animate-spin text-primary" />
                  <span class="text-sm text-muted-foreground">{{ $t('workbench.assets.gen.generatingLabel') }}</span>
                </div>
                <!-- Failed -->
                <div v-else-if="img.state === '生成失败' && !img.src" class="absolute inset-0 bg-white/95 rounded-[20px] flex flex-col items-center justify-center z-10 gap-2">
                  <XCircle :size="40" class="text-destructive" />
                  <span class="text-sm text-destructive font-bold">{{ $t("workbench.assets.gen.genFailed") }}</span>
                </div>
                <!-- Image -->
                <img v-else-if="img.src" :src="img.src" class="w-full h-full object-cover rounded-[20px]" />

                <!-- Overlay controls -->
                <div v-if="hoveredImageIndex === index && img.state === '已完成'" class="absolute top-2 left-2 z-20">
                  <button class="p-1 rounded-full bg-black/50 text-white hover:bg-black/70" @click.stop="handlePreview(img.src)">
                    <Eye :size="18" />
                  </button>
                </div>
                <div v-if="selectedImageIndex === index && img.state === '已完成'" class="absolute top-2 right-2 z-20">
                  <div class="p-1 rounded-full bg-foreground text-background">
                    <Check :size="18" />
                  </div>
                </div>
                <div v-if="hoveredImageIndex === index" class="absolute bottom-2 right-2 z-20">
                  <button class="p-1 rounded-full bg-black/50 text-white hover:bg-destructive/80" @click.stop="deleteImage(img.id, index)">
                    <Trash2 :size="16" />
                  </button>
                </div>
              </div>

              <!-- Custom upload tile -->
              <div
                class="w-44 h-44 rounded-[20px] border-2 border-dashed border-border flex items-center justify-center cursor-pointer hover:border-primary transition-colors"
                @click="customFileInput?.click()">
                <input ref="customFileInput" type="file" accept="image/*" class="hidden" @change="handleCustomUpload" />
                <Plus :size="24" class="text-muted-foreground" />
              </div>
            </div>
          </div>

          <Button class="w-full" size="lg" :disabled="selectedImageIndex === null" @click="onClick">
            {{ $t("workbench.assets.gen.confirmSelect") }}
          </Button>
        </div>
      </div>
    </DialogContent>
  </Dialog>

  <!-- Light image viewer -->
  <Dialog v-model:open="previewVisible">
    <DialogContent class="max-w-3xl bg-black/90 border-0">
      <img :src="previewSrc" class="w-full rounded-lg" />
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import modelSelect from "@/components/modelSelect.vue";
import projectStore from "@/stores/project";
const { project } = storeToRefs(projectStore());
import axios from "@/utils/axios";
import Dialog from "@/components/ui/Dialog.vue";
import DialogContent from "@/components/ui/DialogContent.vue";
import DialogHeader from "@/components/ui/DialogHeader.vue";
import DialogTitle from "@/components/ui/DialogTitle.vue";
import Button from "@/components/ui/Button.vue";
import Textarea from "@/components/ui/Textarea.vue";
import Badge from "@/components/ui/Badge.vue";
import { Loader2, ImageIcon, Upload, Wand2, Eye, Check, Trash2, XCircle, Plus } from "lucide-vue-next";

const props = defineProps<{
  formData: {
    id?: number;
    name?: string;
    describe?: string;
    type?: string;
    prompt?: string;
    src: string;
  };
}>();

const generateImageShow = defineModel({ type: Boolean, default: false });
const emit = defineEmits(["update"]);

const referenceFile = ref<File | null>(null);
const refFileInput = ref<HTMLInputElement | null>(null);
const customFileInput = ref<HTMLInputElement | null>(null);
const generateLoading = ref(false);
const selectValue = ref("");
const resolution = ref("1K");
const promptLoading = ref(false);
const selectedImageIndex = ref<number | null>(null);
const hoveredImageIndex = ref<number | null>(null);
const resultImages = ref<{ id: string; src: string; state: string; selected?: boolean }[]>([]);
const previewVisible = ref(false);
const previewSrc = ref("");

// sync prompt from parent
const promptValue = ref(props.formData.prompt ?? "");
watch(() => props.formData.prompt, (v) => { if (v !== undefined) promptValue.value = v; });
watch(promptValue, (v) => { if (props.formData) (props.formData as any).prompt = v; });

function handleRefFileChange(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (file) referenceFile.value = file;
}
function handleRefDrop(e: DragEvent) {
  const file = e.dataTransfer?.files?.[0];
  if (file && file.type.startsWith("image/")) referenceFile.value = file;
}

function handleCancel() {
  generateImageShow.value = false;
  generateLoading.value = false;
  stopPolling();
  emit("update");
}

async function generatePrompt() {
  promptLoading.value = true;
  try {
    const { data } = await axios.post("/assetsGenerate/polishAssetsPrompt", {
      projectId: project.value?.id,
      assetsId: props.formData.id,
      type: props.formData.type ?? "props",
      name: props.formData.name,
      describe: props.formData.describe ? props.formData.describe : $t("workbench.assets.noDescription"),
    });
    window.$message.success($t("workbench.assets.gen.promptSuccess"));
    if (data.assetsId === props.formData.id) promptValue.value = data.prompt;
  } catch (e: any) {
    window.$message.error(e.message ?? $t("workbench.assets.gen.promptFail"));
  } finally {
    promptLoading.value = false;
  }
}

async function handleGenerate() {
  if (!promptValue.value) { window.$message.error($t("workbench.assets.gen.fillPrompt")); return; }
  if (!resolution.value) { window.$message.error($t("workbench.assets.gen.pickResolution")); return; }
  if (!selectValue.value) { window.$message.error($t("workbench.assets.gen.pickModel")); return; }
  generateLoading.value = true;
  try {
    let referenceImageBase64 = "";
    if (referenceFile.value) {
      referenceImageBase64 = await new Promise<string>((resolve) => {
        const reader = new FileReader();
        reader.onload = (e) => resolve(e.target?.result as string);
        reader.readAsDataURL(referenceFile.value!);
      });
    }
    await axios.post("/assetsGenerate/generateAssets", {
      type: props.formData.type ?? "props",
      projectId: project.value?.id,
      name: props.formData.name ?? $t("workbench.assets.gen.unnamed"),
      base64: referenceImageBase64,
      prompt: promptValue.value,
      model: selectValue.value,
      id: props.formData.id,
      resolution: resolution.value,
    });
    window.$message.success($t("workbench.assets.gen.assetGenSuccess"));
    await fetchGeneratedImages();
  } catch (e: any) {
    window.$message.error(e.message ?? $t("workbench.assets.gen.assetGenFail"));
    fetchGeneratedImages();
  } finally {
    generateLoading.value = false;
  }
}

function handleCustomUpload(e: Event) {
  const file = (e.target as HTMLInputElement).files?.[0];
  if (!file) return;
  const reader = new FileReader();
  reader.onload = (ev) => {
    const base64 = ev.target?.result as string;
    resultImages.value.push({ id: "", src: base64, state: "已完成" });
    window.$message.success($t("workbench.assets.gen.uploadOk"));
    if (customFileInput.value) customFileInput.value.value = "";
  };
  reader.readAsDataURL(file);
}

let pollingTimer: ReturnType<typeof setTimeout> | null = null;
function stopPolling() { if (pollingTimer) { clearTimeout(pollingTimer); pollingTimer = null; } }

async function fetchGeneratedImages() {
  const { data } = await axios.post("/assets/getImage", { assetsId: props.formData.id });
  const images = data.tempAssets.map((item: any) => ({
    id: item.id, src: item.filePath, state: item.state, selected: item.selected ?? false,
  }));
  resultImages.value = images;
  const selectedIdx = images.findIndex((img: any) => img.selected);
  if (selectedIdx !== -1) selectedImageIndex.value = selectedIdx;
  const hasGenerating = images.some((img: any) => img.state === "生成中");
  stopPolling();
  if (hasGenerating && generateImageShow.value) {
    pollingTimer = setTimeout(() => fetchGeneratedImages(), 3000);
  }
}

watch(
  () => generateImageShow.value,
  (newVal) => {
    if (newVal) {
      referenceFile.value = null;
      selectedImageIndex.value = null;
      hoveredImageIndex.value = null;
      generateLoading.value = false;
      fetchGeneratedImages();
    }
  }
);

function selectImage(index: number) {
  if (resultImages.value[index].state === "已完成") {
    selectedImageIndex.value = index;
    window.$message.success($t("workbench.assets.gen.imageSelected"));
  }
}

function handlePreview(src: string) {
  previewSrc.value = src;
  previewVisible.value = true;
}

function deleteImage(id: string | number, index: number) {
  if (!confirm($t("workbench.assets.confirmDeleteBody"))) return;
  axios.post("/assets/delImage", { id });
  resultImages.value.splice(index, 1);
  if (selectedImageIndex.value === index) selectedImageIndex.value = null;
  else if (selectedImageIndex.value !== null && selectedImageIndex.value > index) selectedImageIndex.value--;
  window.$message.success($t("workbench.assets.deleteSuccess"));
}

async function onClick() {
  if (selectedImageIndex.value !== null) {
    const selectedImage = resultImages.value[selectedImageIndex.value];
    const isLocalUpload = !selectedImage.id;
    await axios.post("/assets/saveAssets", {
      id: props.formData.id,
      base64: isLocalUpload ? selectedImage.src : "",
      type: props.formData.type,
      prompt: promptValue.value,
      projectId: project.value?.id,
      imageId: isLocalUpload ? undefined : Number(selectedImage.id),
    });
    window.$message.success($t("workbench.assets.gen.imageSaved"));
    generateImageShow.value = false;
    emit("update");
  }
}
</script>
