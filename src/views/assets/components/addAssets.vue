<template>
  <div class="addAssets">
    <Dialog :open="addAssetsShow" @update:open="(v) => { if (!v) handleCancel() }">
      <DialogContent class="max-w-[40vw]">
        <DialogHeader>
          <DialogTitle>{{ props.title }}</DialogTitle>
        </DialogHeader>
        <div class="data">
          <div class="space-y-4">
            <div class="space-y-2">
              <label class="text-sm font-medium">{{ $t('workbench.assets.add.name') }}</label>
              <Input v-model="props.formData.name" :placeholder="$t('workbench.assets.add.namePh')" />
            </div>
            <div class="space-y-2">
              <label class="text-sm font-medium">{{ $t('workbench.assets.add.describe') }}</label>
              <textarea
                v-model="props.formData.describe"
                :placeholder="$t('workbench.assets.add.describePh')"
                class="flex min-h-[80px] w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50" />
            </div>
            <div class="space-y-2">
              <label class="text-sm font-medium">{{ $t('workbench.assets.add.remark') }}</label>
              <Input v-model="props.formData.remark" :placeholder="$t('workbench.assets.add.remarkPh')" />
            </div>
            <div v-if="props.type !== 'clip'" class="space-y-2">
              <label class="text-sm font-medium">{{ $t('workbench.assets.add.prompt') }}</label>
              <textarea
                v-model="props.formData.prompt"
                :placeholder="$t('workbench.assets.add.promptPh')"
                class="flex min-h-[100px] w-full rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background placeholder:text-muted-foreground focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:cursor-not-allowed disabled:opacity-50" />
            </div>
          </div>
        </div>
        <DialogFooter>
          <Button variant="outline" @click="handleCancel">{{ $t('workbench.assets.cancelBtn') }}</Button>
          <Button @click="onConfirm">{{ $t('workbench.assets.confirmBtn') }}</Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  </div>
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

const { project } = storeToRefs(projectStore());

const props = defineProps<{
  type: "role" | "tool" | "scene" | "clip" | "audio";
  title: string;
  formData: {
    id: number;
    name: string;
    describe: string;
    remark: string;
    prompt: string;
  };
}>();

const addAssetsShow = defineModel<boolean>({
  default: false,
});

function handleCancel() {
  addAssetsShow.value = false;
}

const emit = defineEmits(["getFilteredData"]);

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

  if (props.formData.id !== 0) {
    axios
      .post(`/assets/updateAssets`, {
        id: props.formData.id,
        name: props.formData.name,
        describe: props.formData.describe,
        remark: props.formData.remark,
        prompt: props.formData.prompt,
      })
      .then(() => {
        window.$message.success($t("workbench.assets.add.updateSuccess"));
        emit("getFilteredData");
        addAssetsShow.value = false;
      });
  } else {
    axios
      .post(`/assets/addAssets`, {
        name: props.formData.name,
        describe: props.formData.describe,
        remark: props.formData.remark,
        type: props.type,
        projectId: project.value?.id,
        prompt: props.formData.prompt,
      })
      .then(() => {
        window.$message.success($t("workbench.assets.add.addSuccess"));
        emit("getFilteredData");
        addAssetsShow.value = false;
      });
  }
}
</script>

<style lang="scss" scoped>
.addAssets {
  .data {
    width: 100%;
  }
}
</style>
