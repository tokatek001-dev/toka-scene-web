<template>
  <div
    class="flex bg-background"
    :style="{ height: isElectron ? 'calc(100vh - 32px)' : '100vh' }"
  >
    <!-- Narrow icon sidebar -->
    <TooltipProvider :delay-duration="200">
      <aside class="w-14 flex flex-col items-center py-4 gap-1 bg-card border-r border-border shrink-0">
        <!-- Logo -->
        <div class="mb-4 flex items-center justify-center w-10 h-10">
          <div class="w-7 h-7 bg-primary rounded-lg flex items-center justify-center">
            <div class="w-4 h-4 bg-primary-foreground rounded-sm opacity-80"></div>
          </div>
        </div>

        <!-- Primary nav items -->
        <nav class="flex flex-col items-center gap-1 flex-1">
          <!-- Project / Task (always visible) -->
          <template v-for="(menu, index) in menuList" :key="index">
            <Tooltip v-if="menu.type === 'btn'">
              <TooltipTrigger>
                <button
                  class="w-10 h-10 flex items-center justify-center rounded-lg transition-colors hover:bg-accent hover:text-accent-foreground"
                  :class="{ 'bg-accent text-accent-foreground': activeMenu === menu.path }"
                  @click="handleClick(menu)"
                >
                  <component :is="menu.icon" :size="20" />
                </button>
              </TooltipTrigger>
              <TooltipContent side="right">
                {{ menu.labelKey ? $t(menu.labelKey) : '' }}
              </TooltipContent>
            </Tooltip>
            <div v-else-if="menu.type === 'divider'" class="w-8 h-px bg-border my-1" />
          </template>

          <!-- Divider before project-specific routes -->
          <div v-if="project?.id" class="w-8 h-px bg-border my-1" />

          <!-- Project-specific routes (only when project is open) -->
          <template v-if="project?.id" v-for="(menu, index) in rightBtnList" :key="'right-' + index">
            <Tooltip v-if="menu.type === 'btn' && (project.projectType === 'novel' || !menu.nodelOnly)">
              <TooltipTrigger>
                <button
                  class="w-10 h-10 flex items-center justify-center rounded-lg transition-colors hover:bg-accent hover:text-accent-foreground"
                  :class="{ 'bg-accent text-accent-foreground': activeMenu === menu.path }"
                  @click="handleClick(menu)"
                >
                  <component :is="menu.icon" :size="20" />
                </button>
              </TooltipTrigger>
              <TooltipContent side="right">
                {{ menu.labelKey ? $t(menu.labelKey) : '' }}
              </TooltipContent>
            </Tooltip>
            <div v-else-if="menu.type === 'divider'" class="w-8 h-px bg-border my-1" />
          </template>
        </nav>

        <!-- Footer actions -->
        <div class="flex flex-col items-center gap-1 mt-auto">
          <!-- Feedback -->
          <Tooltip>
            <TooltipTrigger>
              <button
                class="w-10 h-10 flex items-center justify-center rounded-lg transition-colors hover:bg-accent hover:text-accent-foreground text-muted-foreground"
                @click="openFeedback"
              >
                <FileText :size="20" />
              </button>
            </TooltipTrigger>
            <TooltipContent side="right">
              {{ $t('workbench.menu.feedbackQuestions') }}
            </TooltipContent>
          </Tooltip>

          <!-- Settings with update badge -->
          <Tooltip>
            <TooltipTrigger>
              <button
                class="w-10 h-10 flex items-center justify-center rounded-lg transition-colors hover:bg-accent hover:text-accent-foreground text-muted-foreground relative"
                @click="showSetting = true"
              >
                <Settings :size="20" />
                <span v-if="needUpdate" class="absolute top-1.5 right-1.5 w-2 h-2 bg-destructive rounded-full" />
              </button>
            </TooltipTrigger>
            <TooltipContent side="right">
              {{ $t('workbench.menu.settings') }}
            </TooltipContent>
          </Tooltip>

          <!-- Theme toggle (Sun / Moon) -->
          <Tooltip>
            <TooltipTrigger>
              <button
                class="w-10 h-10 flex items-center justify-center rounded-lg transition-colors hover:bg-accent hover:text-accent-foreground text-muted-foreground"
                @click="handleThemeToggle"
              >
                <Sun v-if="isDark" :size="20" />
                <Moon v-else :size="20" />
              </button>
            </TooltipTrigger>
            <TooltipContent side="right">
              {{ isDark ? $t('workbench.menu.lightMode') : $t('workbench.menu.darkMode') }}
            </TooltipContent>
          </Tooltip>

          <!-- GitHub -->
          <Tooltip>
            <TooltipTrigger>
              <button
                class="w-10 h-10 flex items-center justify-center rounded-lg transition-colors hover:bg-accent hover:text-accent-foreground text-muted-foreground"
                @click="jumpGithub"
              >
                <Github :size="20" />
              </button>
            </TooltipTrigger>
            <TooltipContent side="right">
              {{ $t('workbench.menu.jumpGithub') }}
            </TooltipContent>
          </Tooltip>
        </div>
      </aside>
    </TooltipProvider>

    <!-- Main content area -->
    <div class="flex-1 flex flex-col overflow-hidden bg-background">
      <!-- Project top bar (only when a project is active) -->
      <div v-if="project?.id" class="h-12 border-b border-border flex items-center justify-between px-6 shrink-0 bg-card/50 backdrop-blur-sm">
        <h2 class="font-semibold text-foreground truncate">{{ project?.name || $t("workbench.selectProject") }}</h2>
      </div>

      <!-- Router view -->
      <div class="flex-1 overflow-auto scrollbar-thin">
        <router-view v-slot="{ Component }">
          <component :is="Component" :key="$route.fullPath" />
        </router-view>
      </div>
    </div>
  </div>

  <hello />
  <setting />
</template>

<script setup lang="ts">
import { ref, reactive, watch, onMounted, onUnmounted } from "vue";
import { storeToRefs } from "pinia";
import axios from "@/utils/axios";
import setting from "@/components/setting/index.vue";
import hello from "@/components/hello.vue";
import projectStore from "@/stores/project";
const { project } = storeToRefs(projectStore());
import settingStore from "@/stores/setting";
import { NotifyPlugin } from "tdesign-vue-next";
import {
  FolderOpen,
  BookOpen,
  FileText,
  Bot,
  Clapperboard,
  Image,
  LayoutGrid,
  ListTodo,
  Settings,
  Github,
  Sun,
  Moon,
} from "lucide-vue-next";
import TooltipProvider from "@/components/ui/TooltipProvider.vue";
import Tooltip from "@/components/ui/Tooltip.vue";
import TooltipTrigger from "@/components/ui/TooltipTrigger.vue";
import TooltipContent from "@/components/ui/TooltipContent.vue";
import { useTheme, applyThemeMode, toggleThemeWithTransition } from "@/utils/theme";

const { showSetting, isElectron, needUpdate } = storeToRefs(settingStore());
const { themeSetting } = useTheme();

const isDark = computed(() => {
  if (themeSetting.value.mode === "auto") {
    return window.matchMedia("(prefers-color-scheme: dark)").matches;
  }
  return themeSetting.value.mode === "dark";
});

function handleThemeToggle(event: MouseEvent) {
  const newMode = isDark.value ? "light" : "dark";
  toggleThemeWithTransition(event, () => {
    themeSetting.value.mode = newMode;
    applyThemeMode(newMode);
  });
}

const menuList = ref([
  { type: "btn", path: "/project", labelKey: "workbench.menu.myProject", icon: FolderOpen },
  { type: "btn", path: "/task", labelKey: "workbench.menu.taskCenter", icon: ListTodo },
]);

const rightBtnList = ref([
  { type: "btn", path: "/novel", labelKey: "workbench.menu.novel", icon: BookOpen, nodelOnly: true },
  { type: "btn", path: "/scriptAgent", labelKey: "workbench.menu.scriptAgent", icon: Bot, nodelOnly: true },
  { type: "btn", path: "/script", labelKey: "workbench.menu.scriptManage", icon: FileText },
  { type: "btn", path: "/cornerScape", labelKey: "workbench.menu.cornerScape", icon: LayoutGrid },
  { type: "btn", path: "/production", labelKey: "workbench.menu.production", icon: Clapperboard },
  { type: "divider" },
  { type: "btn", path: "/assets", labelKey: "workbench.menu.assetCenter", icon: Image },
]);

const router = useRouter();
const route = useRoute();
const activeMenu = ref(route.path);

watch(
  () => route.path,
  (newPath) => {
    activeMenu.value = newPath;
  },
);

function handleClick(menu: any) {
  if (menu.needProject && !project.value) return;
  router.push(menu.path);
  activeMenu.value = menu.path;
}

async function jumpGithub() {
  if (isElectron.value) {
    await fetch("toonflow://openurlwithbrowser?url=https://github.com/HBAI-Ltd/Toonflow-app");
  } else {
    window.open("https://github.com/HBAI-Ltd/Toonflow-app");
  }
}

async function openFeedback() {
  if (isElectron.value) {
    await fetch("toonflow://openurlwithbrowser?url=https://docs.qq.com/smartsheet/form/EmvmQBrmlPmr%2Fss_vsqk2v%2FvhiGzE?tab=ss_vsqk2v");
  } else {
    window.open("https://docs.qq.com/smartsheet/form/EmvmQBrmlPmr%2Fss_vsqk2v%2FvhiGzE?tab=ss_vsqk2v");
  }
}

async function checkVersion() {
  const { data } = await axios.post("/setting/about/checkUpdate", {
    source: "toonflow",
  });
  if (data.needUpdate) {
    needUpdate.value = true;
    const { activeMenu: settingActiveMenu } = storeToRefs(settingStore());
    const notifyInstance = NotifyPlugin.success({
      title: $t("version.newVersion") as string,
      content: () =>
        h(
          "div",
          { style: "text-align: right; padding-top: 4px;" },
          h(
            "span",
            {
              style: "color: #ed7b2f; font-size: 12px; cursor: pointer;",
              onClick: () => {
                settingActiveMenu.value = "about";
                showSetting.value = true;
                NotifyPlugin.close(notifyInstance);
              },
            },
            $t("skillScan.openSettings"),
          ),
        ),
      closeBtn: true,
      placement: "bottom-right",
    });
  } else {
    needUpdate.value = false;
  }
}

let checkVersionTimer: ReturnType<typeof setInterval> | null = null;

function startVersionCheck() {
  checkVersion();
  checkVersionTimer = setInterval(
    () => {
      checkVersion();
    },
    2 * 60 * 1000,
  );
}

function stopVersionCheck() {
  if (checkVersionTimer) {
    clearInterval(checkVersionTimer);
    checkVersionTimer = null;
  }
}

watch(needUpdate, (val) => {
  if (val) stopVersionCheck();
});

onMounted(() => {
  startVersionCheck();
});

onUnmounted(() => {
  stopVersionCheck();
});
</script>

<style scoped>
.scrollbar-thin {
  scrollbar-width: thin;
  scrollbar-color: hsl(var(--border)) transparent;
}

.scrollbar-thin::-webkit-scrollbar {
  width: 6px;
}

.scrollbar-thin::-webkit-scrollbar-track {
  background: transparent;
}

.scrollbar-thin::-webkit-scrollbar-thumb {
  background-color: hsl(var(--border));
  border-radius: 3px;
}

.scrollbar-thin::-webkit-scrollbar-thumb:hover {
  background-color: hsl(var(--muted-foreground));
}
</style>
