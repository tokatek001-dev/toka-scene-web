# Pinia Stores

## settingStore (`src/stores/setting.ts`)

**Persisted keys:** `baseUrl`, `otherSetting`, `themeSetting`, `language`

### State

- `showSetting: boolean` — toggles the settings drawer open/closed
- `isElectron: boolean` — true when running inside an Electron shell
- `baseUrl: string` — backend API base URL (default: `http://localhost:10588/api`)
- `needUpdate: boolean` — flag indicating an app update is available
- `canvasWheelEvent: "zoom" | "scroll"` — controls mouse wheel behavior on the production canvas
- `activeMenu: string` — currently active settings tab
- `language: string` — current locale code (default: `"zh-CN"`)

### `otherSetting`

- `axiosTimeOut: number` — HTTP request timeout in ms (default: `600000`, 10 minutes)
- `assetsBatchGenereateSize: number` — number of concurrent batch generation requests (default: `5`)
- `chapterReg: string` — regex string used to split novel text into chapters
- `interacting: boolean` — enables interaction mode
- `scriptEpisodeLength: number` — max characters per script episode (default: `5000`)

### `themeSetting`

- `mode: "auto" | "light" | "dark"` — theme mode
- `primaryColor: string` — brand color as hex (default: `#0052D9`)
- `fontSize: number` — base font size (default: `16`)

---

## indexStore (`src/stores/index.ts`)

Not persisted.

### State

- `version: string` — current app version string (e.g. `"v1.0.7"`)
- `activeMenu: string` — active sidebar navigation item
- `project: Project | null` — the currently open project object
- `projectId: ComputedRef<number>` — computed from `project.id`; `-1` if no project is open
- `currentScriptId: number | null` — ID of the script currently in context

### Methods

- `setProjectById(id)` — fetches the project and its current script from the API and populates state

---

## projectStore (`src/stores/project.ts`)

Persisted.

### State

- `allProject: Project[]` — full list of the user's projects
- `project: Project | null` — the currently selected project

### Project Type Fields

- `id`, `name`, `intro`, `type`, `artStyle`, `videoRatio`, `createTime`, `updatedAt`
- `imageModel`, `videoModel`, `projectType`
- `imageQuality: "1K" | "2K" | "4K" | ""` — output image resolution
- `mode: string` — video generation mode; JSON array string for multi-reference mode
- `directorManual: string` — director's notes / manual instructions

---

## userStore (`src/stores/user.ts`)

Persisted.

### State

- `token: string | null` — JWT authentication token

---

## scriptAgentStore (`src/stores/scriptAgent.ts`)

Per-project store factory. Connects via `useChat` composable to `/api/socket/scriptAgent`.

### State

- `messages` — chat message history (TDesign chat-compatible format)
- `planData` — structured plan output:
  - `storySkeleton` — overall story outline
  - `adaptationStrategy` — adaptation approach text
  - `scripts[]` — array of generated script items
- `thinkLevel: 0 | 1 | 2 | 3` — controls AI reasoning depth
- `loading: boolean` — true while awaiting a response

### XML Parsing (from socket stream)

The store extracts structured data from XML tags in the AI's streaming output:

- `<storySkeleton>` → `planData.storySkeleton`
- `<adaptationStrategy>` → `planData.adaptationStrategy`
- `<scriptItem name="...">` → appended to `planData.scripts[]`

### Methods

- `chat(text)` — send a user message
- `stopGenerate()` — interrupt the current generation
- `clearMessages()` — reset the local chat history
- `updateThinkConfig(level)` — emit think level change to the socket
- `setPlanData()` — persist the current plan data to `/scriptAgent/setPlanData`

---

## productionAgentStore (`src/stores/productionAgent.ts`)

Per-project + per-script store factory. Connects via `useChat` to `/api/socket/productionAgent`.

### State

- `messages` — chat history
- `flowData` — complete production flow data:
  - `script` — script text
  - `scriptPlan` — director plan content
  - `storyboardTable` — storyboard overview
  - `assets[]` — project assets
  - `storyboard[]` — storyboard frames
  - `workbench` — workbench configuration
- `thinkLevel: 0 | 1 | 2 | 3`

### Socket Events Handled

- `getFlowData` — fetch and populate initial flow data
- `addDeriveAsset` — add a derived asset to the flow
- `delDeriveAsset` — remove a derived asset
- `generateDeriveAsset` — trigger generation for a derived asset
- `generateStoryboard` — trigger storyboard image generation
- `addStoryboard` — append a new storyboard frame

### Polling

- Periodically polls asset image generation state (every N seconds)
- Periodically polls storyboard image generation state

---

## videoStore (`src/stores/video.ts`)

Not persisted.

### State

- `VideoConfig[]` — video generation configuration per track
- `VideoResult[]` — list of generated video results

### Methods

- `setCurrentScript(scriptId)` — set the active script context
- `fetchVideoConfigs()` — load video configs from backend
- `fetchVideoData()` — load existing video results
- `addConfig()` — add a new video config track
- `removeConfig()` — remove a config track
- `updateConfig()` — partial update of a config
- `updateConfigFull()` — full replacement update of a config
- `generateVideo(config)` — call the backend video generation API
- `selectResult(id)` — mark a video result as selected
- `startPolling()` / `stopPolling()` — start/stop polling every 10 seconds for results with `state=0` (pending)

---

## imageListCacheStore (`src/stores/imageListCache.ts`)

Cache store for image lists keyed by asset ID. Used by the workbench generate component to avoid redundant fetches.

---

## loadingStore (`src/stores/loadingStore.ts`)

Global loading state management. Used to show/hide full-page loading indicators across the app.
