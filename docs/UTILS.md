# Utilities & Composables

## src/utils/axios.ts

Configured axios instance with:

- **baseURL** — dynamically sourced from `settingStore.baseUrl`
- **Request interceptor** — injects `Authorization: Bearer <token>` header using the token from `localStorage`
- **Response interceptor:**
  - `401` — clears localStorage and redirects to `/login`
  - Network errors — shows NotifyPlugin notification with VC++ runtime download links for offline environments

---

## src/utils/useSocket.ts

Socket.io composable returning `{ connected, socket }`.

**Features:**
- Automatic reconnection logic: 10 attempts with exponential backoff (1s–5s delay)
- Connection timeout: 10 seconds
- Methods: `connect(url, auth)`, `disconnect()`, `send(event, data)`, `on(event, cb)`, `off(event, cb)`

---

## src/utils/useChat.ts

Full-featured AI chat composable (~763 lines) built on `useSocket`.

**Core features:**
- `messages` ref — fully compatible with TDesign chat component (message list view)
- **XML tag streaming extraction** with `keepInMessage` option to preserve tags in displayed text
- `onXmlTag(tag, cb)` — callback fires when an XML tag begins streaming and again when complete
- **Socket events:**
  - `message` — new chat message received
  - `message:update` — existing message updated
  - `content:add` — append to the current message
  - `content:update` — update message content by ID
- `manageLifecycle` flag — automatically connects on component mount and disconnects on unmount
- **Think level support** — optionally strips `<think>` tags from displayed output when `thinkLevel=0`

**Methods:**
- `chat(text)` — send a user message
- `stopGenerate()` — emit `stop` event to halt streaming
- `regenerate()` — resend the last user message
- `clearMessages()` — reset local message history
- `removeMessage(id)` — delete a specific message from the history

---

## src/utils/theme.ts

Theme system utilities:

- `generateColorPalette(hex)` → generates a 10-step TDesign-compatible brand palette from a HEX color code
- `applyThemeMode(mode: "auto" | "light" | "dark")` → applies the theme mode via DOM class and attribute
- `applyThemeColor(hex)` → sets TDesign CSS variables `--td-brand-color-*` to the specified color
- `toggleThemeWithTransition(event, cb)` → uses the View Transition API for smooth animated theme switching
- `initTheme()` → called on app startup to apply stored theme settings and listen for system preference changes
- `useTheme()` — composable for use in settings UI components

---

## src/utils/global.ts

Attaches utility objects to the global `window` scope for access outside Vue components:

- `window.$message` — TDesign MessagePlugin instance for toast notifications
- `window.$t` — i18n translation function for multi-language strings
- `window.$port` — Electron port reference (if running in Electron)

---

## src/utils/parseNovel.ts

Novel text parser:

- `parseChapters(text, regex)` → splits raw novel text into a chapter array
- **Regex-configurable:** the split pattern is stored in `settingStore.otherSetting.chapterReg`
- **Output:** array of `{ reel, chapter, chapterData }` objects

---

## src/utils/parseScript.ts

Script content parser:

- Parses raw script text into a structured episode array
- Respects the `scriptEpisodeLength` setting (max characters per episode)

---

## src/utils/videoPolling.ts

Video generation polling utility:

- Periodically polls `/production/workbench/checkVideoStateList` every N seconds
- Updates `VideoResult` state: `0` = pending, `1` = done, `-1` = error
- Automatically stops polling when all pending results are resolved

---

## src/utils/scanSkills.ts

Skill file scanner composable:

- Fetches the skill list from `/setting/skillManagement/getSkillList`
- Used by the Skill Management settings tab to display available skills

---

## src/utils/providersLogo.ts

Provider logo mapper:

- Maps vendor/provider IDs to their corresponding logo image paths in `src/assets/providers/`
- Used throughout the app to display AI provider branding

---

## src/utils/imageOptimizer.ts

Vue plugin for lazy image loading and optimization:

- Installed globally in `main.ts`
- Provides directives and composables for efficient image rendering

---

## src/utils/assetsCheck.ts

Asset validation utility:

- Validates asset state before performing operations (generation, deletion, export)
- Prevents invalid state transitions

---

## src/lib/vendorTemplate.ts

Vendor configuration form templates:

- Defines input field configurations for each AI provider type (e.g. API key, endpoint URL, model ID)
- Used by the Vendor Config settings tab to dynamically build configuration forms

---

## src/locales/index.ts

vue-i18n setup with lazy-loaded locale message modules.

**Supported locales:**
- `zh-CN` (Simplified Chinese, default)
- `zh-TW` (Traditional Chinese)
- `en` (English)
- `ja_JP` (Japanese)
- `ru_RU` (Russian)
- `th_TH` (Thai)
- `vi-VN` (Vietnamese)

**Usage:**
- Initialized in `main.ts` and attached to `window.$t` via `src/utils/global.ts`
- Auto-detected from `navigator.language` on first app load
- User can override in Settings → Language
