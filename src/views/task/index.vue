<template>
  <div class="flex flex-col h-full px-8 py-8">
    <!-- Header -->
    <div class="flex items-center justify-between mb-8 shrink-0">
      <div>
        <h1 class="text-3xl font-bold text-foreground">{{ $t("workbench.task.title") }}</h1>
        <p class="text-sm text-muted-foreground mt-1">{{ $t("workbench.task.subtitle") }}</p>
      </div>
      <Button variant="outline" @click="getTaskList" :disabled="pagination.loading">
        <RefreshCw :size="15" class="mr-2" :class="pagination.loading ? 'animate-spin' : ''" />
        {{ $t("workbench.task.refresh") }}
      </Button>
    </div>

    <!-- Filters -->
    <div class="flex items-center gap-3 mb-4 shrink-0">
      <select
        v-model="projectId"
        @change="onFilterChange"
        class="px-3 py-2 rounded-lg border border-input bg-background text-sm outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 min-w-[160px]"
      >
        <option v-for="opt in projectData" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
      </select>

      <select
        v-model="taskClass"
        @change="onFilterChange"
        class="px-3 py-2 rounded-lg border border-input bg-background text-sm outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 min-w-[140px]"
      >
        <option v-for="opt in categoryOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
      </select>

      <select
        v-model="taskState"
        @change="onFilterChange"
        class="px-3 py-2 rounded-lg border border-input bg-background text-sm outline-none focus:border-primary focus:ring-2 focus:ring-primary/20 min-w-[130px]"
      >
        <option v-for="opt in stateOptions" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
      </select>
    </div>

    <!-- Table -->
    <div class="flex-1 flex flex-col min-h-0">
      <div class="rounded-lg border border-border overflow-hidden">
        <Table>
          <TableHeader>
            <TableRow class="hover:bg-transparent">
              <TableHead class="w-[120px]">{{ $t("workbench.task.col.taskClass") }}</TableHead>
              <TableHead class="w-[120px]">{{ $t("workbench.task.col.relatedObjects") }}</TableHead>
              <TableHead class="w-[200px]">{{ $t("workbench.task.col.model") }}</TableHead>
              <TableHead>{{ $t("workbench.task.col.describe") }}</TableHead>
              <TableHead>{{ $t("workbench.task.col.reason") }}</TableHead>
              <TableHead class="w-[100px]">{{ $t("workbench.task.col.state") }}</TableHead>
              <TableHead class="w-[180px]">{{ $t("workbench.task.col.startTime") }}</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            <!-- Loading skeleton -->
            <template v-if="pagination.loading">
              <TableRow v-for="i in 5" :key="i">
                <TableCell v-for="j in 7" :key="j">
                  <div class="h-4 bg-muted rounded animate-pulse" />
                </TableCell>
              </TableRow>
            </template>

            <!-- Empty -->
            <template v-else-if="taskList.length === 0">
              <TableRow>
                <TableCell colspan="7" class="text-center py-16 text-muted-foreground">
                  <div class="flex flex-col items-center gap-2">
                    <ClipboardList :size="36" class="opacity-30" />
                    <p>{{ $t("workbench.task.noTasks") }}</p>
                  </div>
                </TableCell>
              </TableRow>
            </template>

            <!-- Data rows -->
            <template v-else>
              <TableRow v-for="row in taskList" :key="row.id">
                <TableCell class="font-medium truncate max-w-[120px]">{{ row.taskClass }}</TableCell>
                <TableCell class="truncate max-w-[120px] text-muted-foreground">{{ row.relatedObjects }}</TableCell>
                <TableCell class="truncate max-w-[200px] text-xs text-muted-foreground">{{ row.model }}</TableCell>
                <TableCell class="truncate max-w-[200px] text-sm">{{ row.describe }}</TableCell>
                <TableCell class="truncate max-w-[160px]">
                  <span v-if="row.reason" class="text-xs text-destructive cursor-pointer" :title="row.reason">
                    {{ row.reason }}
                  </span>
                  <span v-else class="text-muted-foreground/40">—</span>
                </TableCell>
                <TableCell>
                  <span
                    class="inline-flex items-center gap-1.5 px-2 py-0.5 rounded-full text-xs font-medium"
                    :class="{
                      'bg-blue-50 text-blue-600': row.state === '进行中',
                      'bg-green-50 text-green-600': row.state === '已完成',
                      'bg-red-50 text-red-600': row.state === '生成失败',
                    }"
                  >
                    <span
                      class="w-1.5 h-1.5 rounded-full"
                      :class="{
                        'bg-blue-400 animate-pulse': row.state === '进行中',
                        'bg-green-400': row.state === '已完成',
                        'bg-red-400': row.state === '生成失败',
                      }"
                    />
                    {{ row.state }}
                  </span>
                </TableCell>
                <TableCell class="text-xs text-muted-foreground whitespace-nowrap">
                  {{ dayjs(row.startTime).format("YYYY-MM-DD HH:mm:ss") }}
                </TableCell>
              </TableRow>
            </template>
          </TableBody>
        </Table>
      </div>

      <!-- Pagination -->
      <div class="flex items-center justify-between mt-4 shrink-0">
        <p class="text-sm text-muted-foreground">
          {{ $t("workbench.task.total", { total: pagination.total }) }}
        </p>
        <div class="flex items-center gap-2">
          <select
            v-model="pagination.limit"
            @change="getTaskList"
            class="px-2 py-1.5 rounded-md border border-input bg-background text-xs outline-none focus:border-primary"
          >
            <option :value="10">10 / {{ $t("workbench.task.page") }}</option>
            <option :value="20">20 / {{ $t("workbench.task.page") }}</option>
            <option :value="50">50 / {{ $t("workbench.task.page") }}</option>
          </select>
          <Button
            variant="outline"
            size="sm"
            :disabled="pagination.page <= 1"
            @click="pagination.page--; getTaskList()"
          >
            <ChevronLeft :size="15" />
          </Button>
          <span class="text-sm text-muted-foreground min-w-[80px] text-center">
            {{ pagination.page }} / {{ Math.max(1, Math.ceil(pagination.total / pagination.limit)) }}
          </span>
          <Button
            variant="outline"
            size="sm"
            :disabled="pagination.page >= Math.ceil(pagination.total / pagination.limit)"
            @click="pagination.page++; getTaskList()"
          >
            <ChevronRight :size="15" />
          </Button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import dayjs from "dayjs";
import axios from "@/utils/axios";
import projectStore from "@/stores/project";
import { RefreshCw, ChevronLeft, ChevronRight, ClipboardList } from "lucide-vue-next";
import Button from "@/components/ui/Button.vue";
import Table from "@/components/ui/Table.vue";
import TableHeader from "@/components/ui/TableHeader.vue";
import TableBody from "@/components/ui/TableBody.vue";
import TableRow from "@/components/ui/TableRow.vue";
import TableHead from "@/components/ui/TableHead.vue";
import TableCell from "@/components/ui/TableCell.vue";

const { project } = storeToRefs(projectStore());

interface TaskItem {
  id: number;
  taskClass: string;
  relatedObjects: string;
  model: string;
  projectName: string;
  episode: string;
  state: string;
  startTime: number;
  describe?: string;
  reason?: string;
}

const stateOptions = [
  { label: $t("workbench.task.stateAll"), value: "" },
  { label: $t("workbench.task.stateRunning"), value: "进行中" },
  { label: $t("workbench.task.stateCompleted"), value: "已完成" },
  { label: $t("workbench.task.stateFailed"), value: "生成失败" },
];

const pagination = ref({ page: 1, limit: 10, total: 0, loading: false });
const categoryOptions = ref<{ label: string; value: string }[]>([]);
const projectData = ref<{ label: string; value: string }[]>([]);
const taskClass = ref("");
const taskState = ref("");
const projectId = ref("");
const taskList = ref<TaskItem[]>([]);

onMounted(() => {
  getTaskList();
  getCategories();
  getProject();
});

function onFilterChange() {
  pagination.value.page = 1;
  getTaskList();
}

async function getCategories() {
  const { data } = await axios.post("/task/getTaskCategories").catch(() => ({ data: [] }));
  categoryOptions.value = [
    { label: $t("workbench.task.stateAll"), value: "" },
    ...data.map((i: any) => ({ label: i.taskClass, value: i.taskClass })),
  ];
}

async function getProject() {
  const { data } = await axios.post("/task/getProject").catch(() => ({ data: [] }));
  projectData.value = [{ label: $t("workbench.task.stateAll"), value: "" }, ...data.map((i: any) => ({ label: i.name, value: i.id }))];
}

async function getTaskList() {
  pagination.value.loading = true;
  try {
    const { data } = await axios.post("/task/getTaskApi", {
      page: pagination.value.page,
      limit: pagination.value.limit,
      taskClass: taskClass.value,
      state: taskState.value,
      projectId: projectId.value || project.value?.id,
    });
    taskList.value = data.data;
    pagination.value.total = data.total;
  } catch {
    window.$message.error($t("workbench.task.fetchFailed"));
  } finally {
    pagination.value.loading = false;
  }
}
</script>
