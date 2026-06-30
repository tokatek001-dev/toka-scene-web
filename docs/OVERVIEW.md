# toka-scene-web — Frontend Overview

## Tech Stack

- Vue 3 (Composition API + `<script setup>`)
- TypeScript
- Vite (build tool)
- Pinia (state management, with pinia-plugin-persistedstate)
- Vue Router (hash history)
- TDesign Vue Next (UI component library)
- @icon-park/vue-next (icons)
- socket.io-client (WebSocket for AI agents)
- @vue-flow/core (node-based canvas for production workflow)
- @webav/av-cliper (video editing)
- md-editor-v3 (markdown editor/preview)
- Splitpanes (resizable panels)
- dayjs (date formatting)
- axios (HTTP client)

## App Entry

- `src/main.ts` — creates app, installs Pinia, Router, i18n, TDesign plugins, imageOptimizer
- `src/App.vue` — detects Electron, initializes theme, configures TDesign i18n, shows titleBar for Electron

## Theme System

`src/utils/theme.ts` provides the full theming API:

- `generateColorPalette(hex)` — generates a 10-step brand palette from a HEX color
- `applyThemeMode(mode)` — sets the `theme-mode` attribute and `.dark` class on the document
- `applyThemeColor(color)` — sets TDesign CSS variables `--td-brand-color-*`
- `toggleThemeWithTransition(event, cb)` — uses the View Transition API for smooth theme switching
- `initTheme()` — applies the stored theme on app start and listens for system dark/light preference changes

Theme settings are stored in `settingStore.themeSetting` (persisted to localStorage):

- `mode: "auto" | "light" | "dark"`
- `primaryColor: string` — hex color code
- `fontSize: number`

## i18n

- `src/locales/` — 7 languages: `zh-CN`, `zh-TW`, `en`, `ja_JP`, `ru_RU`, `th_TH`, `vi-VN`
- Language is auto-detected from `navigator.language` on first load
- Configurable in Settings → Language

## Routing

- Hash-based history (`createWebHashHistory`)
- Auth guard: if no `localStorage.token` → redirect to `/login`
- Layout: `/workbench` wraps all main views as nested children

### Route Map

| Path | View |
|---|---|
| `/project` | Project list |
| `/novel` | Novel chapter management |
| `/script` | Script management |
| `/scriptAgent` | Script Agent chat |
| `/production` | Production VueFlow canvas |
| `/assets` | Asset management |
| `/cornerScape` | Corner scene |
| `/task` | Task view |
| `/login` | Login page |
| `/*` | 404 not found |

## Global Utilities

- `src/utils/global.ts` — attaches `window.$message`, `window.$t` to the global scope for use outside Vue components
- `src/utils/axios.ts` — axios instance with dynamic `baseURL` (from `settingStore`), JWT auth header injection, 401 redirect handling, and network error notifications

## Static Assets

- `src/assets/providers/` — 200+ AI provider logo images (.webp)
- `src/assets/main.scss` — global styles, CSS variables, TDesign component overrides, and utility classes
- `src/assets/bg.png`, `logo.png`, `logo.svg` — app branding assets
