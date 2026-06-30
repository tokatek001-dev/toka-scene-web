# Views & Components

## Layout

### pages/workbench/index.vue

Main application shell layout containing:

- Left sidebar navigation (icon-based menu)
- Top bar with project selector
- Main content area with `<router-view>`
- Settings drawer (component: `setting/index.vue`)
- Electron-specific TitleBar component

### pages/login/index.vue

Login page:

- Username + password form → POST `/login/login`
- On success: stores `token` and `userId` in localStorage → redirects to `/project`
- Inline `baseUrl` configuration input for pointing to a custom backend
- Language switcher for locale selection before login

---

## Views

### /project — `views/project/index.vue`

Project management grid view.

**Features:**
- Card grid displaying all user projects
- Open project: validates `imageModel` and `videoModel` via `/modelSelect/getModelDetail` before proceeding
- Routes to `/novel` if `projectType=novel`, or `/script` if `projectType=script`
- Edit project via `projectDialog.vue`
- Delete project with confirmation dialog; clears cached project data via `clearProjectCache()`
- Add new project via `projectDialog.vue`

**Local components:**
- `addProject.vue` — legacy add form (name, type, artStyle, videoRatio, intro)
- `projectDialog.vue` — current unified add/edit dialog

---

### /novel — `views/novel/index.vue`

Novel chapter management with paginated table.

**Features:**
- Paginated table (10 rows/page) with columns: index, reel, chapter, chapterData, event
- Import chapters via `importNovel.vue` dialog (file upload, paste text, or drag-and-drop)
- Edit individual chapters via `editNodel.vue` dialog
- Batch delete selected rows
- AI event extraction: triggers `/novel/event/generateEvents`; polls every 3 seconds on `eventState`
- Preview dialog for long text content (chapterData / event columns)
- Full-text search across chapters

**Event states:**
- `0` — generating (in progress)
- `-1` — failed
- Other — completed

**Local components:**
- `importNovel.vue` — file upload and chapter parsing using regex-based text splitting
- `editNodel.vue` — edit chapter name, event content, and chapter body
- `eventAnalysis.vue` — collapsible event analysis panels

---

### /script — `views/script/index.vue`

Script management with card grid layout.

**Features:**
- Script cards displaying name, status badge, and creation time
- Add script via `addScript.vue` (upload .txt/.docx, or paste text; attach related assets)
- Edit script via `editScript.vue` (name, content, related assets)
- Batch delete
- Export script content to a file
- AI asset extraction: calls `/script/extractAssets`; polls for extraction completion state
- Batch import via `batchAddScript.vue`

**Local components:**
- `addScript.vue` — upload or paste script content
- `editScript.vue` — edit script name, content, and linked assets
- `batchAddScript.vue` — batch import multiple scripts

---

### /scriptAgent — `views/scriptAgent/index.vue`

Script Agent AI chat interface (~817 lines).

**Layout:** Splitpanes with resizable left and right panels.

**Left panel tabs:**
- **Story Skeleton** — markdown preview of the generated story outline
- **Adaptation Strategy** — markdown preview of the adaptation approach
- **Scripts** — list of generated script cards with per-card edit and delete actions

**Right panel:**
- Chat message thread (TDesign chat component)
- Message input/sender
- Settings menu: reconnect, clear memory by type, think level selector
- Force-generate overlay for manual override

**Features:**
- Real-time XML tag extraction from socket streaming responses
- Collapse/expand all scripts in the left panel
- Think level control (0–3)
- Memory management: clear different memory types independently

---

### /production — `views/production/index.vue`

Production VueFlow canvas for the full AI-to-video pipeline (~618 lines).

**Node flow (left → right):**
1. **script** — displays script content with MdPreview / MdEditor toggle
2. **scriptPlan** — director plan / storyboard plan content
3. **storyboardTable** — storyboard overview table
4. **storyboard** — storyboard frame grid with generated images
5. **assets** — asset cards (parent and derived assets)
6. **workbench** — video generation entry point

**Canvas features:**
- Space-drag to pan the canvas
- Auto-layout powered by dagre (left-to-right direction)
- Episode selector to switch between scripts
- Right-side resizable AI chat panel (Production Agent)
- FPS counter display
- Guided tour steps for onboarding

**Node components:**
- `node/script.vue` — MdPreview with inline edit dialog
- `node/scriptPlan.vue` — plan content display
- `node/storyboard.vue` — frame grid, batch image generation, per-frame editing
- `node/assets.vue` — asset cards with image generation via editImage flow
- `node/workbench.vue` — video generation placeholder; opens full-screen workbench dialog

**Production utilities:**
- `utils/flowBuilder.ts` — `useFlowBuilder` composable; assembles `nodes[]` and `edges[]` for VueFlow
- `utils/dagre.ts` — auto-layout algorithm implementation
- `utils/editImageType.ts` — type definitions for editImage node variants

**Sub-components:**
- `components/rightChatBox/index.vue` — resizable AI chat panel with drag handle and agent settings
- `components/workbench/index.vue` — full-screen workbench dialog with tabs: Preview / Generate / Edit Video
- `components/workbench/generate/index.vue` — video generation UI: reference image upload, mode selector, prompt input, result polling
- `components/workbench/editVideo/index.vue` — video timeline editor
- `components/workbench/editVideo/videoPreview.vue` — inline video preview player
- `components/editImage/index.vue` — node-based image editing flow

---

### /assets — `views/assets/index.vue`

Comprehensive asset management view (~1629 lines).

**Tabs:** Role | Tool | Scene | Clip | Audio

**Features (per tab):**
- Expandable table rows for detail view
- Batch generate prompt using AI
- Batch generate images
- Batch delete assets
- Individual asset editing (name, describe, remark, prompt)
- Image generation polling for pending state assets
- Media preview dialog (images and video clips)
- Full image viewer

**Local components:**
- `addAssets.vue` — add or edit a single asset
- `addAudioAssets.vue` — add an audio asset
- `generateImage.vue` — image generation dialog: reference image upload, prompt, model/resolution selector, gallery selection
- `batchGeneration.vue` — batch operations UI wrapper

---

### /cornerScape — `views/cornerScape/index.vue`

Corner scene management — binds audio tracks to assets.

**Features:**
- Lists all assets in the current project
- Batch bind audio to multiple assets
- Polls for audio generation completion state
- Update asset audio binding after generation

---

### /task — `views/task/index.vue`

Task queue monitor — displays background AI task status and progress.

### /taskList — `views/taskList/index.vue`

Detailed task list view with per-task-class status breakdown.

---

## Shared Components

### components/setting/index.vue

Settings drawer with the following tabs:

- **UI** — theme mode (auto/light/dark), primary color picker, font size
- **Vendor Config** — configure AI provider credentials (API keys, endpoints)
- **Agent Config** — assign models to each agent role
- **Model Map** — bind specific prompts to specific models
- **Prompt Manage** — view and edit system prompts
- **Skill Management** — view and edit agent skill files
- **Memory Config** — configure memory parameters
- **DB Config** — database info panel with export, import, and clear actions
- **Login Config** — change account password
- **Language** — locale selector
- **Dev** — toggle developer tools
- **File Management** — open the local data folder
- **About** — app version display and update check

---

### components/modelSelect.vue

Reusable AI model selector component:

- Fetches vendors and models from `/modelSelect/getModelList`
- Filters by capability type: `text`, `image`, `video`, `tts`
- Displays provider logo alongside model name

---

### components/titleBar.vue

Custom Electron window title bar with minimize, maximize, and close buttons.

---

### components/imageTools.vue

Floating image toolbar providing: zoom in/out, rotate, download, and copy to clipboard.

---

### components/promptEditor.vue

Prompt text editor with support for variable insertion (e.g. `{{variableName}}`).

---

### components/storyboardImageCheck.vue

Storyboard image validation dialog — verifies image state before publishing or exporting.

---

### components/editMdPreivew.vue

Combined component that toggles between `MdEditor` (edit mode) and `MdPreview` (read mode) for markdown content.
