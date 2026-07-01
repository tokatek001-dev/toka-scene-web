<template>
  <Dialog :open="addAssetsShow" @update:open="(v) => { if (!v) handleCancel() }">
    <DialogContent class="max-w-[40vw]">
      <DialogHeader>
        <DialogTitle>{{ props.formData.id ? '编辑' : '新增' }}</DialogTitle>
      </DialogHeader>
      <div class="data">
        <div class="space-y-4">
          <div class="space-y-2">
            <label class="text-sm font-medium">{{ $t('workbench.assets.add.audioName') }}</label>
            <Input v-model="props.formData.name" :placeholder="$t('workbench.assets.add.audioNamePh')" />
          </div>
          <div class="space-y-2">
            <label class="text-sm font-medium">{{ $t('workbench.assets.add.describe') }}</label>
            <textarea
              v-model="props.formData.describe"
              :placeholder="$t('workbench.assets.add.describePh')"
              class="flex min-h-[80px] w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50" />
          </div>
          <div class="space-y-2">
            <label class="text-sm font-medium">{{ $t('workbench.assets.add.sex') }}</label>
            <Input v-model="props.formData.sex" :placeholder="$t('workbench.assets.add.sexPh')" />
          </div>
          <div class="space-y-2">
            <label class="text-sm font-medium">{{ $t('workbench.assets.add.audioFile') }}</label>
            <div class="audio-list">
              <div v-for="(item, index) in audioItems" :key="index" class="audio-item">
                <div class="audio-upload-row">
                  <div class="audio-file-area" @click="triggerFileInput(index)" @dragover.prevent @drop.prevent="(e) => handleDrop(e, index)">
                    <template v-if="item.file">
                      <i-volume-notice size="16" />
                      <span class="audio-filename">{{ item.file.name }}</span>
                    </template>
                    <template v-else-if="item.src">
                      <i-volume-notice size="16" fill="var(--td-success-color)" />
                      <span class="audio-filename audio-filename--existing">{{ item.name }}</span>
                      <span class="ml-auto flex-shrink-0 text-xs px-2 py-0.5 rounded bg-green-100 text-green-700">已上传</span>
                    </template>
                    <template v-else>
                      <i-upload-one size="16" fill="var(--td-brand-color)" />
                      <span class="audio-upload-hint">点击或拖拽上传音频</span>
                    </template>
                    <input
                      :ref="(el) => (fileInputRefs[index] = el as HTMLInputElement)"
                      type="file"
                      accept="audio/*"
                      style="display: none"
                      @change="(e) => handleFileChange(e, index)" />
                  </div>
                  <Button variant="destructive" size="sm" class="rounded-full w-7 h-7 p-0 flex-shrink-0" @click="removeAudioItem(index)">
                    <X :size="12" />
                  </Button>
                </div>
                <Input v-model="item.text" placeholder="请输入该音频对应的文本内容" class="audio-text-input" />
                <Input v-model="item.describe" placeholder="请输入该音频的描述" class="audio-text-input" />
              </div>
              <Button variant="outline" size="sm" @click="addAudioItem">
                <Plus :size="14" class="mr-1" />
                添加音频
              </Button>
            </div>
          </div>
        </div>
      </div>
      <DialogFooter>
        <Button variant="outline" @click="handleCancel">{{ $t('workbench.assets.cancelBtn') }}</Button>
        <Button @click="onConfirm">{{ $t('workbench.assets.confirmBtn') }}</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import axios from "@/utils/axios";
import projectStore from "@/stores/project";
import Dialog from "@/components/ui/Dialog.vue";
import DialogContent from "@/components/ui/DialogContent.vue";
import DialogHeader from "@/components/ui/DialogHeader.vue";
import DialogTitle from "@/components/ui/DialogTitle.vue";
import DialogFooter from "@/components/ui/DialogFooter.vue";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import { Plus, X } from "lucide-vue-next";

const { project } = storeToRefs(projectStore());

interface AudioItem {
  id?: number;
  src?: string;
  file: File | null;
  text: string;
  name: string;
  describe: string;
}
const props = defineProps<{
  formData: {
    id?: number;
    name: string;
    describe: string;
    sex: string;
    sonAssets?: {
      id?: number;
      src?: string;
      prompt: string;
      name?: string;
      describe?: string;
    }[];
  };
}>();
const addAssetsShow = defineModel<boolean>({
  default: false,
});

function handleCancel() {
  addAssetsShow.value = false;
  // 重置音频列表
  audioItems.value = [{ file: null, text: "", name: "", describe: "" }];
}
const formRef = ref();
const emit = defineEmits(["getFilteredData"]);

const audioItems = ref<AudioItem[]>([{ file: null, text: "", name: "", describe: "" }]);
const fileInputRefs = ref<HTMLInputElement[]>([]);

// 初始化音频列表
watch(
  () => props.formData.sonAssets,
  (newSonAssets) => {
    if (newSonAssets && newSonAssets.length > 0) {
      audioItems.value = newSonAssets.map((asset) => ({
        id: asset.id,
        src: asset.src,
        file: null,
        text: asset.prompt,
        name: asset.name || "",
        describe: asset.describe || "",
      }));
    } else {
      audioItems.value = [{ file: null, text: "", name: "", describe: "" }];
    }
  },
  { immediate: true },
);

function addAudioItem() {
  audioItems.value.push({ file: null, text: "", name: "", describe: "" });
}

function removeAudioItem(index: number) {
  audioItems.value.splice(index, 1);
  if (audioItems.value.length === 0) {
    audioItems.value.push({ file: null, text: "", name: "", describe: "" });
  }
}

function triggerFileInput(index: number) {
  fileInputRefs.value[index]?.click();
}

function handleFileChange(e: Event, index: number) {
  const input = e.target as HTMLInputElement;
  const file = input.files?.[0];
  if (file) {
    audioItems.value[index].file = file;
    audioItems.value[index].src = undefined;
    if (!audioItems.value[index].name) {
      audioItems.value[index].name = file.name;
    }
  }
  input.value = "";
}

function handleDrop(e: DragEvent, index: number) {
  const file = e.dataTransfer?.files?.[0];
  if (file && file.type.startsWith("audio/")) {
    audioItems.value[index].file = file;
    if (!audioItems.value[index].name) {
      audioItems.value[index].name = file.name;
    }
  } else if (file) {
    window.$message.warning($t("workbench.assets.add.pleaseUploadAudio"));
  }
}

async function fileToBase64(file: File): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => {
      resolve(reader.result as string);
    };
    reader.onerror = reject;
    reader.readAsDataURL(file);
  });
}

function onConfirm() {
  // Simple validation
  if (!props.formData.name?.trim()) {
    window.$message.error($t("workbench.assets.add.nameRequired"));
    return;
  }
  if (!props.formData.describe?.trim()) {
    window.$message.error($t("workbench.assets.add.describeRequired"));
    return;
  }

  (async () => {
    const assetsItem = (
      await Promise.all(
        audioItems.value.map(async (item) => {
          if (item.id != null && item.src) {
            return {
              id: item.id,
              src: item.src,
              prompt: item.text || "",
              name: item.name || item.src.split("/").pop() || "",
              describe: item.describe || "",
            };
          }
          if (item.file) {
            return {
              base64: await fileToBase64(item.file),
              prompt: item.text || "",
              name: item.name || item.file.name,
              describe: item.describe || "",
            };
          }
          return null;
        }),
      )
    ).filter(
      (
        item,
      ): item is
        | { id: number; src: string; prompt: string; name: string; describe: string }
        | { base64: string; prompt: string; name: string; describe: string } => !!item,
    );

    const payload = {
      name: props.formData.name,
      describe: props.formData.sex + "|" + props.formData.describe,
      projectId: project.value?.id ?? 0,
      assetsItem,
    };
    console.log(props.formData.id);
    if (props.formData.id) {
      await axios
        .post(`/assets/updateAudioAssets`, {
          id: props.formData.id,
          ...payload,
        })
        .then(() => {
          window.$message.success($t("workbench.assets.add.updateSuccess"));
          emit("getFilteredData");
          addAssetsShow.value = false;
        });
    } else {
      await axios.post(`/assets/addAudioAssets`, payload).then(() => {
        window.$message.success($t("workbench.assets.add.addSuccess"));
        emit("getFilteredData");
        addAssetsShow.value = false;
      });
    }
  })();
}
</script>

<style lang="scss" scoped>
.data {
  width: 100%;
}

.audio-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
  width: 100%;

  .audio-item {
    display: flex;
    flex-direction: column;
    gap: 6px;
    padding: 10px;
    border: 1px solid hsl(var(--border));
    border-radius: 6px;
    background: hsl(var(--muted) / 0.3);

    .audio-upload-row {
      display: flex;
      align-items: center;
      gap: 8px;

      .audio-file-area {
        flex: 1;
        display: flex;
        align-items: center;
        gap: 8px;
        padding: 6px 10px;
        border: 1px dashed hsl(var(--border));
        border-radius: 4px;
        cursor: pointer;
        min-height: 34px;
        transition: border-color 0.2s;

        &:hover {
          border-color: hsl(var(--primary));
          background: hsl(var(--muted) / 0.5);
        }

        .audio-filename {
          font-size: 13px;
          color: hsl(var(--foreground));
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          max-width: 200px;
          flex: 1;

          &--existing {
            color: hsl(var(--primary));
          }
        }

        .audio-upload-hint {
          font-size: 12px;
          color: hsl(var(--muted-foreground));
        }
      }
    }

    .audio-text-input {
      width: 100%;
    }
  }
}
</style>
