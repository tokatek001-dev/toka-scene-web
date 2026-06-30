<template>
  <div class="flex flex-col h-full px-8 py-8">
    <!-- Top toolbar -->
    <div class="flex items-center justify-between mb-8">
      <div>
        <h1 class="text-3xl font-bold text-foreground">{{ $t("workbench.project.title") }}</h1>
        <p class="text-sm text-muted-foreground mt-1">{{ $t("workbench.project.subtitle") }}</p>
      </div>
      <Button
        @click="editProjectData = null; dialogShow = true"
        class="flex items-center gap-2"
      >
        <Plus :size="16" />
        {{ $t("workbench.project.newProject") }}
      </Button>
    </div>

    <!-- Project Grid -->
    <div v-if="allProject.length > 0" class="grid grid-cols-1 md:grid-cols-2 xl:grid-cols-3 gap-5">
      <Card
        v-for="project in allProject"
        :key="project.id"
        class="group relative cursor-pointer hover:shadow-xl hover:-translate-y-0.5 transition-all duration-200 dark:bg-card dark:border-border overflow-hidden"
      >
        <!-- Clickable body area -->
        <CardHeader class="pb-3" @click.stop="openProject(project.id)">
          <div class="flex items-start justify-between gap-2">
            <div class="flex items-center gap-2 min-w-0">
              <CardTitle class="text-lg font-bold truncate">
                {{ project.name }}
              </CardTitle>
            </div>
            <div class="flex items-center gap-2 shrink-0">
              <Badge variant="secondary" class="shrink-0">
                {{ project.projectType == "novel" ? $t(`workbench.project.type.novel`) : $t(`workbench.project.type.script`) }}
              </Badge>

              <!-- 3-dot dropdown menu -->
              <div @click.stop>
                <DropdownMenu>
                  <DropdownMenuTrigger>
                    <button class="p-1.5 rounded-md hover:bg-muted transition-colors opacity-0 group-hover:opacity-100 focus:opacity-100">
                      <MoreHorizontal :size="16" class="text-muted-foreground" />
                    </button>
                  </DropdownMenuTrigger>
                  <DropdownMenuContent class="min-w-[120px]" align="end">
                    <DropdownMenuItem @click="openEdit(project)" class="cursor-pointer flex items-center gap-2">
                      <Pencil :size="14" />
                      {{ $t("workbench.project.edit") }}
                    </DropdownMenuItem>
                    <DropdownMenuItem @click="delProjcer(project.id)" class="cursor-pointer flex items-center gap-2 text-destructive focus:text-destructive">
                      <Trash2 :size="14" />
                      {{ $t("workbench.project.delete") }}
                    </DropdownMenuItem>
                  </DropdownMenuContent>
                </DropdownMenu>
              </div>
            </div>
          </div>
          <Badge v-if="project.artStyle" variant="outline" class="self-start mt-1 text-xs">
            {{ project.artStyle }}
          </Badge>
        </CardHeader>

        <CardContent class="pt-0" @click.stop="openProject(project.id)">
          <p class="text-sm text-muted-foreground line-clamp-2 min-h-[2.5rem]">
            {{ project.intro || $t("workbench.project.noIntro") }}
          </p>
          <p class="text-xs text-muted-foreground/60 mt-3 flex items-center gap-1.5">
            <CalendarDays :size="12" />
            {{ dayjs(project?.createTime).format("YYYY-MM-DD HH:mm") }}
          </p>
        </CardContent>

        <CardFooter class="pt-0 gap-2 flex-wrap" @click.stop="openProject(project.id)">
          <div v-if="project.imageModel" class="flex items-center gap-1.5 text-xs px-2.5 py-1 bg-muted rounded-full text-muted-foreground">
            <Image :size="12" />
            <span class="truncate max-w-[100px]">{{ project.imageModel }}</span>
          </div>
          <div v-if="project.videoModel" class="flex items-center gap-1.5 text-xs px-2.5 py-1 bg-muted rounded-full text-muted-foreground">
            <Video :size="12" />
            <span class="truncate max-w-[100px]">{{ project.videoModel }}</span>
          </div>
        </CardFooter>
      </Card>
    </div>

    <!-- Empty state -->
    <div v-else class="flex flex-col items-center justify-center flex-1 gap-4 py-24 text-muted-foreground">
      <div class="w-20 h-20 rounded-full bg-muted flex items-center justify-center">
        <FolderOpen :size="36" class="text-muted-foreground/50" />
      </div>
      <div class="text-center">
        <p class="text-lg font-medium">{{ $t("workbench.project.noProjects") }}</p>
        <p class="text-sm opacity-60 mt-1">{{ $t("workbench.project.noProjectsHint") }}</p>
      </div>
      <Button @click="editProjectData = null; dialogShow = true" variant="outline" class="mt-2">
        <Plus :size="16" class="mr-2" />
        {{ $t("workbench.project.newProject") }}
      </Button>
    </div>
  </div>

  <projectDialog v-model="dialogShow" :projectData="editProjectData" @add="addProjectFn" @edit="editProjectFn" />
</template>

<script setup lang="ts">
import { DialogPlugin } from "tdesign-vue-next";
import projectDialog from "./components/projectDialog.vue";
import dayjs from "dayjs";
import axios from "@/utils/axios";
import projectStore from "@/stores/project";
import imageListCacheStore from "@/stores/imageListCache";
import { Plus, MoreHorizontal, Pencil, Trash2, CalendarDays, Image, Video, FolderOpen } from "lucide-vue-next";
import Card from "@/components/ui/Card.vue";
import CardHeader from "@/components/ui/CardHeader.vue";
import CardTitle from "@/components/ui/CardTitle.vue";
import CardContent from "@/components/ui/CardContent.vue";
import CardFooter from "@/components/ui/CardFooter.vue";
import Button from "@/components/ui/Button.vue";
import Badge from "@/components/ui/Badge.vue";
import DropdownMenu from "@/components/ui/DropdownMenu.vue";
import DropdownMenuTrigger from "@/components/ui/DropdownMenuTrigger.vue";
import DropdownMenuContent from "@/components/ui/DropdownMenuContent.vue";
import DropdownMenuItem from "@/components/ui/DropdownMenuItem.vue";

const { clearProjectCache } = imageListCacheStore();
const { allProject, project } = storeToRefs(projectStore());

const dialogShow = ref(false);
const editProjectData = ref<{
  id: string;
  name: string;
  intro: string;
  type: string;
  artStyle: string | null;
  videoRatio: string | null;
  imageModel: string;
  videoModel: string;
  projectType: string;
  imageQuality: "1K" | "2K" | "4K" | "";
  mode: string;
  directorManual: string;
} | null>(null);

async function getAllProject() {
  axios.post("/project/getProject").then(({ data }) => {
    allProject.value = data;
  });
}

onMounted(() => {
  project.value = null;
  getAllProject();
});

const router = useRouter();

async function openProject(projectId: string | undefined) {
  const item = allProject.value.find((p) => p.id === projectId);

  if (!item) return window.$message.error($t("workbench.project.msg.notFound"));

  if (!item.imageModel || !item.videoModel) {
    window.$message.warning($t("workbench.project.msg.modelProviderDisabled"));
    return openEdit(item);
  }

  try {
    if (item.imageModel) {
      await axios.post("/modelSelect/getModelDetail", {
        modelId: item.imageModel,
      });
    }
    if (item.videoModel) {
      await axios.post("/modelSelect/getModelDetail", {
        modelId: item.videoModel,
      });
    }
  } catch {
    window.$message.warning($t("workbench.project.msg.modelProviderDisabled"));
    return openEdit(item);
  }

  project.value = item;
  if (item.projectType === "novel") router.push(`/novel`);
  else if (item.projectType === "script") router.push(`/script`);
}

function openEdit(item: {
  id: string;
  name: string;
  intro: string;
  type: string;
  artStyle: string | null;
  directorManual: string;
  videoRatio: string | null;
  imageModel: string;
  videoModel: string;
  imageQuality: "1K" | "2K" | "4K" | "";
  projectType: string;
  mode: string;
}) {
  editProjectData.value = {
    ...item,
  };
  dialogShow.value = true;
}

function editProjectFn(data: {
  id: string;
  name: string;
  intro: string;
  type: string;
  artStyle: string;
  directorManual: string;
  videoRatio: string;
  imageModel: string;
  videoModel: string;
  imageQuality: "1K" | "2K" | "4K" | "";
  mode: string;
}) {
  axios
    .post("/project/editProject", data)
    .then(() => {
      window.$message.success($t("workbench.project.msg.editSuccess"));
      getAllProject();
    })
    .catch((e) => {
      window.$message.error(e.message ?? $t("workbench.project.msg.editFailed"));
    });
}

function addProjectFn(data: {
  projectType: string;
  name: string;
  intro: string;
  type: string;
  artStyle: string;
  directorManual: string;
  videoRatio: string;
  imageModel: string;
  videoModel: string;
  imageQuality: string;
  mode: string;
}) {
  axios
    .post("/project/addProject", data)
    .then(() => {
      window.$message.success($t("workbench.project.msg.addSuccess"));
      getAllProject();
    })
    .catch((e) => {
      window.$message.error(e.message ?? $t("workbench.project.msg.addFailed"));
    });
}

function delProjcer(projectId: string | undefined) {
  const dialog = DialogPlugin.confirm({
    header: $t("workbench.project.msg.deleteHeader"),
    body: $t("workbench.project.msg.deleteBody"),
    confirmBtn: $t("workbench.project.msg.deleteConfirm"),
    cancelBtn: $t("workbench.project.msg.deleteCancel"),
    onConfirm: () => {
      axios
        .post("/project/delProject", { id: projectId })
        .then(() => {
          clearProjectCache(projectId!);
          window.$message.success($t("workbench.project.msg.deleteSuccess"));
          getAllProject();
        })
        .catch((e) => {
          window.$message.error(e.message ?? $t("workbench.project.msg.deleteFailed"));
        })
        .finally(() => {
          dialog.destroy();
        });
    },
  });
}
</script>

<style scoped>
/* Smooth hover elevation */
.group:hover {
  box-shadow: 0 10px 40px -10px rgba(0, 0, 0, 0.15);
}

.dark .group:hover {
  box-shadow: 0 10px 40px -10px rgba(0, 0, 0, 0.5);
}
</style>
