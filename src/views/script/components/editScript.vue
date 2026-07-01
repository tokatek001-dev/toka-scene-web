<template>
  <Dialog v-model:open="detailsShow">
    <DialogContent class="max-w-2xl max-h-[90vh] flex flex-col overflow-hidden">
      <DialogHeader>
        <DialogTitle>{{ $t("workbench.script.edit.title") }}</DialogTitle>
      </DialogHeader>

      <div class="flex-1 overflow-y-auto flex flex-col gap-4 py-2">
        <div class="space-y-2">
          <Label>{{ $t('workbench.script.edit.scriptName') }}</Label>
          <Input v-model="props.item.name" :maxlength="10" :placeholder="$t('workbench.script.edit.scriptNamePh')" />
        </div>
        <div class="space-y-2">
          <Label>{{ $t('workbench.script.edit.scriptContent') }}</Label>
          <Textarea
            v-model="props.item.content"
            :placeholder="$t('workbench.script.edit.scriptContentPh')"
            class="min-h-[300px] resize-none"
          />
          <p class="text-xs text-right" :class="props.item.content.length > otherSetting.scriptEpisodeLength ? 'text-destructive' : 'text-muted-foreground'">
            {{ props.item.content.length }}/{{ otherSetting.scriptEpisodeLength }}
          </p>
        </div>
        <div class="space-y-2">
          <div class="flex items-center justify-between">
            <Label>{{ $t('workbench.script.edit.relatedAssets') }}</Label>
            <Button size="sm" variant="outline" @click="handleSelectAssets">
              <Plus :size="14" class="mr-1" />
              {{ $t("workbench.script.edit.selectAssets") }}
            </Button>
          </div>
          <div v-if="selectedAssets.length" class="flex flex-wrap gap-2">
            <Badge
              v-for="asset in selectedAssets"
              :key="asset.id"
              variant="secondary"
              class="gap-1 cursor-pointer"
              @click="removeAsset(asset.id)">
              {{ asset.name }}<X :size="12" />
            </Badge>
          </div>
          <p v-else class="text-sm text-muted-foreground">{{ $t("workbench.script.edit.noAssets") }}</p>
        </div>
      </div>

      <DialogFooter>
        <Button variant="outline" @click="detailsShow = false">{{ $t("workbench.novel.import.prevStep") }}</Button>
        <Button :disabled="props.item.content.length > otherSetting.scriptEpisodeLength" @click="onConfirm">保存</Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import axios from "@/utils/axios";
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
import { Plus, X } from "lucide-vue-next";

const { otherSetting } = storeToRefs(settingStore());

interface ScriptAsset { id: number; name: string; }
interface ScriptItem { id: number; name: string; content: string; relatedAssets?: ScriptAsset[]; }

const detailsShow = defineModel<boolean>({ default: false });
const props = defineProps<{ item: ScriptItem }>();

const selectedAssets = ref<ScriptAsset[]>([]);
watch(() => props.item?.relatedAssets, (relatedAssets) => {
  selectedAssets.value = relatedAssets?.map((a) => ({ id: a.id, name: a.name })) ?? [];
}, { immediate: true });

async function handleSelectAssets() {
  const assets = await openAssetsSelector({ title: $t("workbench.script.edit.msg.selectAssetsTitle"), types: ["role", "tool", "scene"] });
  if (assets.length) {
    const existing = new Set(selectedAssets.value.map((a) => a.id));
    for (const a of assets) { if (!existing.has(a.id)) selectedAssets.value.push({ id: a.id, name: a.name }); }
  }
}
function removeAsset(id: number) { selectedAssets.value = selectedAssets.value.filter((a) => a.id !== id); }

const emit = defineEmits(["searchScripts"]);
async function onConfirm() {
  try {
    await axios.post("/script/updateScript", { id: props.item.id, name: props.item.name, content: props.item.content, assets: selectedAssets.value.map((a) => a.id) });
    emit("searchScripts"); detailsShow.value = false;
    window.$message.success($t("workbench.script.edit.msg.updateSuccess"));
  } catch (error) { window.$message.error((error as any)?.message ?? $t("workbench.script.edit.msg.updateFailed")); }
}
</script>
