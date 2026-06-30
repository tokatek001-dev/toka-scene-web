<template>
  <div
    class="min-h-screen flex"
    :style="{ height: isElectron ? 'calc(100vh - 32px)' : '100vh' }"
  >
    <!-- Left branding panel -->
    <div
      class="hidden lg:flex lg:w-[52%] relative overflow-hidden flex-col items-center justify-center p-14"
      style="background: linear-gradient(135deg, #0f0c29 0%, #1a1560 45%, #24243e 100%)"
    >
      <!-- Ambient glow orbs -->
      <div
        class="absolute top-[-80px] left-[-80px] w-[400px] h-[400px] rounded-full opacity-20 blur-3xl pointer-events-none"
        style="background: radial-gradient(circle, #6366f1, transparent)"
      />
      <div
        class="absolute bottom-[-60px] right-[-60px] w-[350px] h-[350px] rounded-full opacity-15 blur-3xl pointer-events-none"
        style="background: radial-gradient(circle, #8b5cf6, transparent)"
      />

      <!-- Content -->
      <div class="relative z-10 max-w-sm w-full space-y-10">
        <!-- Logo + brand -->
        <div class="space-y-3">
          <div class="flex items-center gap-4">
            <div
              class="w-14 h-14 rounded-2xl flex items-center justify-center shadow-xl"
              style="background: linear-gradient(135deg, #6366f1, #8b5cf6)"
            >
              <svg class="w-7 h-7 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M15 10l4.553-2.069A1 1 0 0121 8.882v6.236a1 1 0 01-1.447.894L15 14M3 8a2 2 0 012-2h8a2 2 0 012 2v8a2 2 0 01-2 2H5a2 2 0 01-2-2V8z"
                />
              </svg>
            </div>
            <div>
              <h1 class="text-3xl font-bold text-white tracking-tight">ToonFlow</h1>
              <p class="text-indigo-300 text-sm mt-0.5">{{ $t("login.slogan") }}</p>
            </div>
          </div>
        </div>

        <!-- Divider -->
        <div class="h-px" style="background: linear-gradient(90deg, transparent, rgba(99,102,241,0.4), transparent)" />

        <!-- Features -->
        <div class="space-y-4">
          <p class="text-xs font-semibold uppercase tracking-widest text-indigo-400">
            {{ $t("login.features") }}
          </p>
          <ul class="space-y-3">
            <li
              v-for="(feat, i) in features"
              :key="i"
              class="flex items-start gap-3"
            >
              <div class="mt-0.5 w-5 h-5 rounded-md flex items-center justify-center shrink-0"
                style="background: rgba(99,102,241,0.2)"
              >
                <svg class="w-3 h-3 text-indigo-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M5 13l4 4L19 7" />
                </svg>
              </div>
              <span class="text-slate-200 text-sm leading-relaxed">{{ feat }}</span>
            </li>
          </ul>
        </div>

        <!-- Decorative bottom tag -->
        <div class="flex items-center gap-2 pt-4">
          <div class="w-2 h-2 rounded-full bg-indigo-400 animate-pulse" />
          <span class="text-xs text-slate-500">AI-powered · Automated · Professional</span>
        </div>
      </div>
    </div>

    <!-- Right form panel -->
    <div class="flex-1 flex flex-col items-center justify-center p-8 bg-white">
      <div class="w-full max-w-[380px] space-y-8">

        <!-- Mobile logo -->
        <div class="flex lg:hidden items-center gap-3">
          <div
            class="w-10 h-10 rounded-xl flex items-center justify-center"
            style="background: linear-gradient(135deg, #6366f1, #8b5cf6)"
          >
            <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M15 10l4.553-2.069A1 1 0 0121 8.882v6.236a1 1 0 01-1.447.894L15 14M3 8a2 2 0 012-2h8a2 2 0 012 2v8a2 2 0 01-2 2H5a2 2 0 01-2-2V8z"
              />
            </svg>
          </div>
          <span class="text-xl font-bold text-slate-900">ToonFlow</span>
        </div>

        <!-- Header text -->
        <div class="space-y-1">
          <h2 class="text-2xl font-bold text-slate-900 tracking-tight">{{ $t("login.login") }}</h2>
          <p class="text-sm text-slate-500">{{ $t("login.enterCredentials") }}</p>
        </div>

        <!-- Form -->
        <form @submit.prevent="handleLogin" class="space-y-5">
          <!-- Username -->
          <div class="space-y-1.5">
            <label class="text-sm font-medium text-slate-700" for="username">
              {{ $t("login.username") }}
            </label>
            <div class="relative">
              <div class="absolute inset-y-0 left-3 flex items-center pointer-events-none">
                <svg class="w-4 h-4 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"
                  />
                </svg>
              </div>
              <input
                id="username"
                v-model="state.user.username"
                type="text"
                :placeholder="$t('login.usernamePlaceholder')"
                autocomplete="username"
                class="w-full pl-10 pr-4 py-2.5 rounded-lg border border-slate-200 text-sm text-slate-900 placeholder-slate-400 outline-none transition-all focus:border-indigo-400 focus:ring-2 focus:ring-indigo-100 bg-slate-50/50"
              />
            </div>
          </div>

          <!-- Password -->
          <div class="space-y-1.5">
            <label class="text-sm font-medium text-slate-700" for="password">
              {{ $t("login.password") }}
            </label>
            <div class="relative">
              <div class="absolute inset-y-0 left-3 flex items-center pointer-events-none">
                <svg class="w-4 h-4 text-slate-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"
                  />
                </svg>
              </div>
              <input
                id="password"
                v-model="state.user.password"
                :type="showPassword ? 'text' : 'password'"
                :placeholder="$t('login.passwordPlaceholder')"
                autocomplete="current-password"
                class="w-full pl-10 pr-10 py-2.5 rounded-lg border border-slate-200 text-sm text-slate-900 placeholder-slate-400 outline-none transition-all focus:border-indigo-400 focus:ring-2 focus:ring-indigo-100 bg-slate-50/50"
              />
              <button
                type="button"
                class="absolute inset-y-0 right-3 flex items-center text-slate-400 hover:text-slate-600"
                @click="showPassword = !showPassword"
              >
                <svg v-if="!showPassword" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
                  />
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
                  />
                </svg>
                <svg v-else class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21"
                  />
                </svg>
              </button>
            </div>
          </div>

          <!-- Submit button -->
          <button
            type="submit"
            :disabled="state.loginLoading"
            class="w-full py-2.5 rounded-lg text-sm font-semibold text-white transition-all disabled:opacity-60 disabled:cursor-not-allowed"
            style="background: linear-gradient(135deg, #6366f1, #8b5cf6)"
            :style="state.loginLoading ? {} : {}"
          >
            <span v-if="state.loginLoading" class="flex items-center justify-center gap-2">
              <svg class="animate-spin w-4 h-4" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z" />
              </svg>
              {{ $t("login.loggingIn") }}
            </span>
            <span v-else>{{ $t("login.login") }}</span>
          </button>
        </form>

        <!-- Server settings (collapsible, subtle) -->
        <div class="border-t border-slate-100 pt-5">
          <button
            type="button"
            class="flex items-center gap-1.5 text-xs text-slate-400 hover:text-slate-600 transition-colors"
            @click="showServerSettings = !showServerSettings"
          >
            <svg
              class="w-3.5 h-3.5 transition-transform"
              :class="showServerSettings ? 'rotate-90' : ''"
              fill="none" stroke="currentColor" viewBox="0 0 24 24"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
            </svg>
            {{ $t("login.settings") }}
          </button>

          <div v-if="showServerSettings" class="mt-3 space-y-2">
            <div class="relative">
              <input
                v-model="tempBaseUrl"
                type="text"
                placeholder="http://localhost:10588"
                class="w-full px-3 py-2 rounded-lg border border-slate-200 text-xs text-slate-700 placeholder-slate-400 outline-none focus:border-indigo-400 focus:ring-2 focus:ring-indigo-100 bg-slate-50"
              />
            </div>
            <button
              type="button"
              @click="handleSaveSetting"
              class="w-full py-2 rounded-lg border border-slate-200 text-xs font-medium text-slate-600 hover:bg-slate-50 transition-colors"
            >
              {{ $t("login.save") }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Floating controls: language + (hidden settings already in form) -->
  <div class="fixed bottom-6 right-6 flex flex-col gap-2 z-50">
    <div class="relative">
      <button
        class="w-10 h-10 rounded-xl bg-white border border-slate-200 flex items-center justify-center hover:bg-slate-50 transition-colors shadow-md text-slate-500 hover:text-slate-700"
        @click="showLangMenu = !showLangMenu"
        :title="'Language'"
      >
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M3 5h12M9 3v2m1.048 9.5A18.022 18.022 0 016.412 9m6.088 9h7M11 21l5-10 5 10M12.751 5C11.783 10.77 8.07 15.61 3 18.129"
          />
        </svg>
      </button>
      <div
        v-if="showLangMenu"
        class="absolute bottom-full right-0 mb-2 w-36 bg-white rounded-xl shadow-lg border border-slate-100 overflow-hidden"
      >
        <button
          v-for="lang in languageList"
          :key="lang.value"
          @click="handleChangeLang({ value: lang.value }); showLangMenu = false"
          class="w-full text-left px-3 py-2 hover:bg-slate-50 transition-colors text-xs text-slate-700"
        >
          {{ lang.label }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed } from "vue";
import { useI18n } from "vue-i18n";
import { useRouter } from "vue-router";
import axios from "@/utils/axios";
import settingStore from "@/stores/setting";
import { storeToRefs } from "pinia";
import { languageList, cachedLocale } from "@/locales";

const Router = useRouter();
const { locale, t: $t } = useI18n();

const store = settingStore();
const { baseUrl, isElectron } = storeToRefs(store);

const showLangMenu = ref(false);
const showServerSettings = ref(false);
const showPassword = ref(false);
const tempBaseUrl = ref(baseUrl.value);

const features = computed(() => [
  $t("login.feature1"),
  $t("login.feature2"),
  $t("login.feature3"),
]);

const handleSaveSetting = () => {
  baseUrl.value = tempBaseUrl.value;
  showServerSettings.value = false;
  window.$message.success($t("login.settingsSaved"));
};

const handleChangeLang = (data: { value: string }) => {
  locale.value = data.value;
  cachedLocale.value = data.value;
};

const state = ref({
  loginLoading: false,
  user: {
    username: "",
    password: "",
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
