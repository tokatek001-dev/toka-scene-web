<template>
  <Dialog v-model:open="addScriptShow">
    <DialogContent class="max-w-2xl max-h-[90vh] flex flex-col overflow-hidden">
      <DialogHeader>
        <DialogTitle>{{ $t('workbench.script.add.title') }}</DialogTitle>
      </DialogHeader>

      <div class="flex-1 overflow-y-auto flex flex-col gap-5 py-2">
        <div class="space-y-2">
          <Label>{{ $t("workbench.script.add.scriptName") }}</Label>
          <Input v-model="scriptName" :placeholder="$t('workbench.script.add.scriptNamePh')" />
        </div>

        <div class="space-y-2">
          <Label>{{ $t("workbench.script.add.uploadFile") }}</Label>
          <div
            class="border-2 border-dashed border-border rounded-lg p-8 text-center cursor-pointer transition-colors hover:border-primary"
            @click="triggerUpload"
            @dragover.prevent
            @drop.prevent="handleDrop">
            <input ref="fileInputRef" type="file" accept=".txt,.docx" class="hidden" @change="handleFileChange" />
            <Upload :size="28" class="mx-auto mb-3 text-muted-foreground" />
            <p class="text-sm font-medium">{{ $t("workbench.script.add.dragUpload") }}</p>
            <p class="text-xs text-muted-foreground mt-1">{{ $t("workbench.script.add.uploadHint") }}</p>
          </div>
        </div>

        <div class="space-y-2">
          <Label>{{ $t("workbench.script.add.scriptContent") }}</Label>
          <Textarea
            v-model="scriptData"
            :placeholder="$t('workbench.script.add.scriptContentPh')"
            class="min-h-[200px] resize-none"
          />
          <p class="text-xs text-right" :class="scriptData.length > otherSetting.scriptEpisodeLength ? 'text-destructive' : 'text-muted-foreground'">
            {{ scriptData.length }}/{{ otherSetting.scriptEpisodeLength }}
          </p>
        </div>

        <div class="space-y-2">
          <div class="flex items-center justify-between">
            <Label>{{ $t("workbench.script.add.relatedAssets") }}</Label>
            <Button size="sm" variant="outline" @click="handleSelectAssets">
              <Plus :size="14" class="mr-1" />
              {{ $t("workbench.script.add.selectAssets") }}
            </Button>
          </div>
          <div v-if="selectedAssets.length" class="flex flex-wrap gap-2">
            <Badge
              v-for="asset in selectedAssets"
              :key="asset.id"
              variant="secondary"
              class="gap-1 cursor-pointer"
              @click="removeAsset(asset.id)">
              {{ asset.name }}
              <X :size="12" />
            </Badge>
          </div>
          <p v-else class="text-sm text-muted-foreground">{{ $t("workbench.script.add.noAssets") }}</p>
        </div>
      </div>

      <DialogFooter>
        <Button variant="outline" @click="handleCancel">{{ $t("workbench.script.add.cancel") }}</Button>
        <Button :disabled="scriptData.length > otherSetting.scriptEpisodeLength" @click="handleConfirm">
          {{ $t("workbench.script.add.confirm") }}
        </Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import mammoth from "mammoth";
import axios from "@/utils/axios";
import projectStore from "@/stores/project";
import openAssetsSelector from "@/utils/assetsCheck";
import settingStore from "@/stores/setting";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import Textarea from "@/components/ui/Textarea.vue";
import Badge from "@/components/ui/Badge.vue";
import Dialog from "@/components/ui/Dialog.vue"
import DialogContent from "@/components/ui/DialogContent.vue"
import DialogHeader from "@/components/ui/DialogHeader.vue"
import DialogTitle from "@/components/ui/DialogTitle.vue"
import DialogFooter from "@/components/ui/DialogFooter.vue";
import { Plus, Upload, X } from "lucide-vue-next";

const { otherSetting } = storeToRefs(settingStore());
const { project } = storeToRefs(projectStore());

const addScriptShow = defineModel<boolean>({ default: false });

const fileInputRef = ref<HTMLInputElement | null>(null);
const scriptData = ref("");
const scriptName = ref("");
const keepLoading = ref(false);

interface SelectedAsset { id: number; name: string; }
const selectedAssets = ref<SelectedAsset[]>([]);

function triggerUpload() { fileInputRef.value?.click(); }

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
  if (rawFile.type === "application/msword") { window.$message.warning($t("workbench.script.add.msg.docNotSupported")); return; }
  if (!allowTypes.includes(rawFile.type)) { window.$message.error($t("workbench.script.add.msg.unsupportedType")); return; }
  if (rawFile.size > 10 * 1024 * 1024) { window.$message.error($t("workbench.script.add.msg.fileTooLarge")); return; }
  try {
    const buffer = await rawFile.arrayBuffer();
    scriptData.value = rawFile.type === "text/plain"
      ? new TextDecoder().decode(buffer)
      : (await mammoth.extractRawText({ arrayBuffer: buffer })).value;
  } catch { window.$message.error($t("workbench.script.add.msg.parseFailed")); }
}

async function handleSelectAssets() {
  const assets = await openAssetsSelector({ title: $t("workbench.script.add.msg.selectAssetsTitle"), types: ["role", "tool", "scene"] });
  if (assets.length) {
    const existing = new Set(selectedAssets.value.map((a) => a.id));
    for (const a of assets) { if (!existing.has(a.id)) selectedAssets.value.push({ id: a.id, name: a.name }); }
  }
}

function removeAsset(id: number) { selectedAssets.value = selectedAssets.value.filter((a) => a.id !== id); }

function handleCancel() { addScriptShow.value = false; scriptData.value = ""; scriptName.value = ""; selectedAssets.value = []; }

const emit = defineEmits(["searchScripts"]);

async function handleConfirm() {
  if (!scriptData.value.trim()) { window.$message.warning($t("workbench.script.add.msg.enterContent")); return; }
  if (!scriptName.value.trim()) { window.$message.warning($t("workbench.script.add.msg.enterName")); return; }
  keepLoading.value = true;
  try {
    await axios.post("/script/addScript", { name: scriptName.value, content: scriptData.value, projectId: project.value?.id, assets: selectedAssets.value.map((a) => a.id) });
    window.$message.success($t("workbench.script.add.msg.addSuccess"));
    handleCancel(); emit("searchScripts");
  } catch (error) { window.$message.error((error as any).message ?? $t("workbench.script.add.msg.addFailed")); }
  finally { keepLoading.value = false; }
}
</script>
