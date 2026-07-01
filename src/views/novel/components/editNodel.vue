<template>
  <Dialog v-model:open="editNodelShow">
    <DialogContent class="max-w-xl max-h-[85vh] flex flex-col overflow-hidden">
      <DialogHeader>
        <DialogTitle>{{ $t('workbench.novel.editDialog.title') }}</DialogTitle>
      </DialogHeader>

      <div class="flex-1 overflow-y-auto flex flex-col gap-4 py-2">
        <div class="space-y-2">
          <Label>{{ $t('workbench.novel.editDialog.chapterName') }}</Label>
          <Input :placeholder="$t('workbench.novel.editDialog.chapterNamePh')" v-model="formData.chapter" />
        </div>
        <div class="space-y-2">
          <Label>{{ $t('workbench.novel.editDialog.eventContent') }}</Label>
          <Textarea
            v-model="formData.event"
            :placeholder="$t('workbench.novel.editDialog.eventContentPh')"
            class="min-h-[80px] resize-none"
          />
        </div>
        <div class="space-y-2">
          <Label>{{ $t('workbench.novel.editDialog.chapterContent') }}</Label>
          <Textarea
            :placeholder="$t('workbench.novel.editDialog.chapterContentPh')"
            v-model="formData.chapterData"
            class="min-h-[200px] resize-none"
          />
        </div>
      </div>

      <DialogFooter>
        <Button variant="outline" @click="editNodelShow = false">
          {{ $t('workbench.novel.editDialog.cancel') }}
        </Button>
        <Button @click="saveChanges">
          {{ $t('workbench.novel.editDialog.save') }}
        </Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>

<script setup lang="ts">
import axios from "@/utils/axios";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import Textarea from "@/components/ui/Textarea.vue";
import Dialog from "@/components/ui/Dialog.vue"
import DialogContent from "@/components/ui/DialogContent.vue"
import DialogHeader from "@/components/ui/DialogHeader.vue"
import DialogTitle from "@/components/ui/DialogTitle.vue"
import DialogFooter from "@/components/ui/DialogFooter.vue";

const editNodelShow = defineModel<boolean>();
const props = defineProps<{
  formData: {
    id: number;
    index: number;
    reel: string;
    chapter: string;
    chapterData: string;
    event: string;
  };
}>();
const emit = defineEmits(["select"]);

async function saveChanges() {
  try {
    await axios.post("/novel/updateNovel", {
      id: props.formData.id,
      index: props.formData.index,
      reel: props.formData.reel,
      chapter: props.formData.chapter,
      chapterData: props.formData.chapterData,
      event: props.formData.event,
    });
    emit("select");
    window.$message.success($t('workbench.novel.editDialog.msg.updateSuccess'));
  } catch (e) {
    window.$message.error((e as Error).message);
  } finally {
    editNodelShow.value = false;
  }
}
</script>
