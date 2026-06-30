<template>
  <div class="min-h-screen flex flex-col lg:flex-row" :style="{ height: isElectron ? 'calc(100vh - 32px)' : '100vh' }">
    <!-- Left branding panel (hidden on mobile) -->
    <div class="hidden lg:flex lg:w-1/2 bg-gradient-to-br from-slate-900 to-slate-800 dark:from-slate-950 dark:to-slate-900 items-center justify-center p-12">
      <div class="flex flex-col items-center text-center space-y-8">
        <!-- Logo -->
        <div class="flex items-center gap-4">
          <div class="w-20 h-20 bg-white dark:bg-slate-100 rounded-lg flex items-center justify-center">
            <div class="w-12 h-12 bg-gradient-to-br from-blue-600 to-blue-400 rounded-md"></div>
          </div>
          <div class="text-left">
            <h1 class="text-4xl font-bold text-white">ToonFlow</h1>
            <p class="text-slate-300 text-sm">{{ $t("login.slogan") }}</p>
          </div>
        </div>

        <!-- Features -->
        <div class="space-y-4 text-left">
          <p class="text-slate-300 text-lg font-medium">{{ $t("login.features") }}</p>
          <ul class="space-y-3">
            <li class="flex items-center gap-3 text-slate-200">
              <div class="w-2 h-2 bg-blue-400 rounded-full"></div>
              <span>{{ $t("login.feature1") }}</span>
            </li>
            <li class="flex items-center gap-3 text-slate-200">
              <div class="w-2 h-2 bg-blue-400 rounded-full"></div>
              <span>{{ $t("login.feature2") }}</span>
            </li>
            <li class="flex items-center gap-3 text-slate-200">
              <div class="w-2 h-2 bg-blue-400 rounded-full"></div>
              <span>{{ $t("login.feature3") }}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <!-- Right form panel -->
    <div class="flex-1 flex flex-col items-center justify-center p-8 bg-gradient-to-br from-white to-slate-50 dark:from-slate-950 dark:to-slate-900">
      <div class="w-full max-w-md">
        <!-- Mobile logo (visible on mobile) -->
        <div class="flex lg:hidden items-center justify-center gap-3 mb-8">
          <div class="w-16 h-16 bg-gradient-to-br from-blue-600 to-blue-400 rounded-lg flex items-center justify-center">
            <div class="w-10 h-10 bg-white rounded-md"></div>
          </div>
          <h1 class="text-2xl font-bold">ToonFlow</h1>
        </div>

        <!-- Card -->
        <Card class="dark:bg-slate-900 dark:border-slate-800">
          <CardHeader>
            <CardTitle class="text-center text-2xl">{{ $t("login.login") }}</CardTitle>
            <CardDescription class="text-center">
              {{ $t("login.enterCredentials") }}
            </CardDescription>
          </CardHeader>
          <CardContent>
            <form @submit.prevent="handleLogin" class="space-y-4">
              <div class="space-y-2">
                <Label :for="'username'">{{ $t("login.username") }}</Label>
                <Input
                  id="username"
                  v-model="state.user.username"
                  :placeholder="$t('login.usernamePlaceholder')"
                  autocomplete="username"
                  class="dark:bg-slate-800 dark:border-slate-700 dark:text-white"
                />
              </div>

              <div class="space-y-2">
                <Label :for="'password'">{{ $t("login.password") }}</Label>
                <Input
                  id="password"
                  v-model="state.user.password"
                  type="password"
                  :placeholder="$t('login.passwordPlaceholder')"
                  autocomplete="current-password"
                  class="dark:bg-slate-800 dark:border-slate-700 dark:text-white"
                />
              </div>

              <Button
                type="submit"
                class="w-full mt-6 h-10"
                :disabled="state.loginLoading"
              >
                <template v-if="state.loginLoading">
                  <span class="flex items-center gap-2">
                    <svg class="animate-spin h-4 w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                      <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                      <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    {{ $t("login.loggingIn") }}
                  </span>
                </template>
                <template v-else>
                  {{ $t("login.login") }}
                </template>
              </Button>
            </form>

            <p class="text-xs text-muted-foreground text-center mt-4 opacity-60">
              {{ $t("login.tips") }}
            </p>
          </CardContent>
        </Card>

        <!-- Settings section -->
        <div class="mt-8 pt-8 border-t border-slate-200 dark:border-slate-800">
          <details class="cursor-pointer group">
            <summary class="flex items-center gap-2 text-sm font-medium text-slate-600 dark:text-slate-400 hover:text-slate-900 dark:hover:text-slate-200">
              <svg class="w-4 h-4 transform group-open:rotate-90 transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
              </svg>
              {{ $t("login.settings") }}
            </summary>
            <div class="mt-4 space-y-3">
              <div>
                <Label :for="'baseUrl'">{{ $t("login.requestAddress") }}</Label>
                <Input
                  id="baseUrl"
                  v-model="tempBaseUrl"
                  type="text"
                  placeholder="http://localhost:10588"
                  class="mt-2 dark:bg-slate-800 dark:border-slate-700 dark:text-white"
                />
              </div>
              <Button
                @click="handleSaveSetting"
                variant="outline"
                size="sm"
                class="w-full dark:border-slate-700 dark:hover:bg-slate-800"
              >
                {{ $t("login.save") }}
              </Button>
            </div>
          </details>
        </div>
      </div>
    </div>
  </div>

  <!-- Language and settings buttons (floating) -->
  <div class="fixed bottom-8 right-8 flex flex-col gap-3 z-50">
    <!-- Language dropdown -->
    <div class="relative group">
      <button
        class="w-12 h-12 rounded-lg bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 flex items-center justify-center hover:bg-slate-50 dark:hover:bg-slate-700 transition-colors shadow-lg"
        @click="showLangMenu = !showLangMenu"
      >
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
          <path d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.949V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 6V5z"></path>
        </svg>
      </button>
      <div
        v-if="showLangMenu"
        class="absolute bottom-full right-0 mb-2 w-40 bg-white dark:bg-slate-800 rounded-lg shadow-lg border border-slate-200 dark:border-slate-700 overflow-hidden"
      >
        <button
          v-for="lang in languageList"
          :key="lang.value"
          @click="handleChangeLang({ value: lang.value }); showLangMenu = false"
          class="w-full text-left px-4 py-2 hover:bg-slate-100 dark:hover:bg-slate-700 transition-colors text-sm"
        >
          {{ lang.label }}
        </button>
      </div>
    </div>

    <!-- Settings button -->
    <button
      @click="showSettingModal = true"
      class="w-12 h-12 rounded-lg bg-primary text-white flex items-center justify-center hover:bg-primary/90 transition-colors shadow-lg"
    >
      <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"></path>
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path>
      </svg>
    </button>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";
import axios from "@/utils/axios";
import settingStore from "@/stores/setting";
import { storeToRefs } from "pinia";
import { languageList, cachedLocale } from "@/locales";
import Card from "@/components/ui/Card.vue";
import CardHeader from "@/components/ui/CardHeader.vue";
import CardTitle from "@/components/ui/CardTitle.vue";
import CardDescription from "@/components/ui/CardDescription.vue";
import CardContent from "@/components/ui/CardContent.vue";
import Button from "@/components/ui/Button.vue";
import Input from "@/components/ui/Input.vue";
import Label from "@/components/ui/Label.vue";
import { Sun, Moon } from "lucide-vue-next";

const Router = useRouter();

const { locale, t: $t } = useI18n();

const store = settingStore();
const { baseUrl, isElectron } = storeToRefs(store);

const showSettingModal = ref(false);
const showLangMenu = ref(false);
const tempBaseUrl = ref(baseUrl.value);

// 保存设置
const handleSaveSetting = () => {
  baseUrl.value = tempBaseUrl.value;
  showSettingModal.value = false;
  window.$message.success($t("login.settingsSaved"));
};

const handleChangeLang = (data: { value: string }) => {
  locale.value = data.value;
  cachedLocale.value = data.value;
};

const state = ref({
  show: true,
  loginLoading: false,
  user: {
    username: "",
    password: "",
  },
  rules: {
    username: [{ required: true, message: $t("login.usernameRequired") }],
    password: [{ required: true, message: $t("login.passwordRequired") }],
  },
});

const handleLogin = () => {
  if (!state.value.user.username || !state.value.user.password) {
    window.$message.warning($t("login.enterUsernameAndPassword"));
    return;
  }
  state.value.loginLoading = true;
  const obj = { ...state.value.user };
  axios
    .post("/login/login", obj)
    .then(({ data }) => {
      localStorage.setItem("token", data.token);
      localStorage.setItem("userId", data.id);
      Router.push("/project");
      window.$message.success($t("login.loginSuccess"));
      state.value.loginLoading = false;
    })
    .catch((e) => {
      state.value.loginLoading = false;
      window.$message.error(e.message);
    });
};
</script>

<style scoped>
::-webkit-scrollbar {
  width: 6px;
}

::-webkit-scrollbar-track {
  background: transparent;
}

::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 3px;
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 0, 0, 0.3);
}

.dark ::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.2);
}

.dark ::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.3);
}
</style>
