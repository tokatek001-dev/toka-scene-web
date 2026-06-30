# ToonFlow — Shadcn UI Migration Plan

**Goal:** Chuyển toàn bộ UI toka-scene-web sang shadcn/vue thuần, light mode mặc định, loại bỏ TDesign khỏi các screen chính.

**Architecture:**
- Giữ nguyên logic, store, router — chỉ thay UI layer
- Mỗi screen được refactor độc lập, theo thứ tự từ ngoài vào trong
- TDesign (`t-*`) được thay bằng shadcn components (`@/components/ui/*`) + Tailwind
- Light mode là default, dark mode vẫn hoạt động qua `applyThemeMode`

**Tech Stack:** Vue 3 + Vite + TypeScript + Tailwind v3 + shadcn-vue (radix-vue) + lucide-vue-next

**Thứ tự ưu tiên:**
1. Global (theme default, globals.css, main.scss)
2. Screen 1: Login
3. Screen 2: Workbench Shell (sidebar)
4. Screen 3: Project List
5. Screen 4: Task Center
6. Screen 5: Novel (Văn bản gốc)
7. Screen 6: Script (Kịch bản)
8. Screen 7: Assets (Tài nguyên)
9. Screen 8: Production (Flow canvas — node panels)
10. Screen 9: Settings Dialog

---

## TASK 0 — Global: Force light mode default + cleanup globals

**Objective:** Light mode là default khi user chưa chọn theme. TDesign dark vars không leak vào shadcn layout.

**Files:**
- Modify: `src/stores/setting.ts`
- Modify: `src/assets/globals.css`
- Modify: `src/assets/main.scss`
- Modify: `src/utils/theme.ts`

**Step 1: Đổi default mode sang `light` trong store**

File: `src/stores/setting.ts`

```ts
// Trước
const themeSetting = ref({
  mode: 'auto',
  primaryColor: '#0052D9',
  fontSize: 16,
})

// Sau
const themeSetting = ref({
  mode: 'light',           // ← đổi từ 'auto' sang 'light'
  primaryColor: '#0052D9',
  fontSize: 16,
})
```

**Step 2: Thêm TDesign light override vào `globals.css`**

Append vào cuối `src/assets/globals.css`:

```css
/* ── TDesign force-light override ────────────────────────── */
/* Khi không có class .dark, TDesign dùng light vars mặc định.
   Override các token hay bị leak sang nền tối. */
:root,
[theme-mode='light'],
html:not([theme-mode='dark']) {
  --td-bg-color-container: #ffffff;
  --td-bg-color-page:       #f5f6fa;
  --td-text-color-primary:  #1a1a1a;
  --td-text-color-secondary:#666666;
  --td-component-border:    #dcdfe6;
  --td-bg-color-component:  #ffffff;
  --td-gray-color-1:        #f5f6fa;
  --td-gray-color-2:        #ededf0;
  --td-gray-color-3:        #e3e6eb;
  --bgc:                    #f5f6fa;
}
```

**Step 3: Cleanup `main.scss` — xoá `--bgc` override, giữ electron + dialog fix**

```scss
/* Xoá dòng này trong main.scss */
background-color: var(--bgc);      /* ← xoá */
color: var(--td-text-color-primary); /* ← xoá */

/* Thay bằng — map sang shadcn token */
background-color: hsl(var(--background));
color: hsl(var(--foreground));
```

**Step 4: `initTheme` không apply dark nếu mode = light**

`src/utils/theme.ts` — `initTheme` hiện chạy `applyThemeMode(themeSetting.value.mode)`.
Đảm bảo nếu mode = `'light'` thì **không** add class `.dark` và không set `theme-mode="dark"`.
Hàm `applyThemeMode` đã làm đúng — không cần sửa thêm. Chỉ cần verify bằng cách log.

**Verify:**
```bash
cd toka-scene-web && npm run dev
# Mở http://localhost:5174, F12 → xem <html> không có class "dark" và không có attribute theme-mode="dark"
```

---

## TASK 1 — Screen: Login (`src/pages/login/index.vue`)

**Objective:** Strip toàn bộ `dark:*` hardcode class. Left panel dùng `bg-primary` thay gradient tối. Form panel dùng `bg-background`.

**Files:**
- Modify: `src/pages/login/index.vue`

**Current problems:**
- Left panel: `bg-gradient-to-br from-slate-900 to-slate-800` — hardcode tối, không theo theme
- Right panel: `dark:from-slate-950 dark:to-slate-900` — hardcode dark class
- Card: `dark:bg-slate-900 dark:border-slate-800`
- Input: `dark:bg-slate-800 dark:border-slate-700 dark:text-white`
- Lang menu dropdown: `dark:bg-slate-800 dark:border-slate-700`
- Details/settings section: `dark:border-slate-800`, `dark:text-slate-400`, `dark:hover:text-slate-200`

**Step 1: Thay left branding panel**

```vue
<!-- Trước -->
<div class="hidden lg:flex lg:w-1/2 bg-gradient-to-br from-slate-900 to-slate-800 dark:from-slate-950 dark:to-slate-900 ...">
  <h1 class="text-4xl font-bold text-white">ToonFlow</h1>
  <p class="text-slate-300 text-sm">...</p>
  <li class="... text-slate-200">...</li>

<!-- Sau — dùng primary token, không hardcode màu -->
<div class="hidden lg:flex lg:w-1/2 bg-primary items-center justify-center p-12">
  <h1 class="text-4xl font-bold text-primary-foreground">ToonFlow</h1>
  <p class="text-primary-foreground/70 text-sm">...</p>
  <li class="... text-primary-foreground/90">...</li>
```

**Step 2: Thay right form panel**

```vue
<!-- Trước -->
<div class="flex-1 flex flex-col items-center justify-center p-8 bg-gradient-to-br from-white to-slate-50 dark:from-slate-950 dark:to-slate-900">

<!-- Sau -->
<div class="flex-1 flex flex-col items-center justify-center p-8 bg-background">
```

**Step 3: Strip dark: khỏi Card, Input, details section, lang dropdown**

Regex tìm: `dark:[a-zA-Z0-9:_/-]+` → xoá hết.

Cụ thể:
```vue
<!-- Card -->
<Card>  <!-- xoá class="dark:bg-slate-900 dark:border-slate-800" -->

<!-- Input -->
<Input class="..." />  <!-- xoá dark:* classes -->

<!-- Details border -->
<div class="mt-8 pt-8 border-t border-border">  <!-- đổi border-slate-200 → border-border -->

<!-- Lang dropdown -->
<div class="... bg-card rounded-lg shadow-lg border border-border overflow-hidden">
```

**Step 4: Feature dots — đổi màu cố định sang accent**

```vue
<!-- Trước -->
<div class="w-2 h-2 bg-blue-400 rounded-full"></div>

<!-- Sau -->
<div class="w-2 h-2 bg-primary-foreground/60 rounded-full"></div>
```

**Verify:** `npm run build` không lỗi. Mở `/` → login page trắng sáng, không có vùng tối.

---

## TASK 2 — Screen: Workbench Shell (`src/pages/workbench/index.vue`)

**Objective:** Sidebar icon-only đã dùng shadcn tokens đúng. Chỉ cần strip `dark:` thừa (nếu còn) và đảm bảo `bg-card border-border` là chuẩn.

**Files:**
- Modify: `src/pages/workbench/index.vue`

**Current state:** Workbench shell đã khá clean — dùng `bg-card`, `border-border`, `hover:bg-accent`, `text-muted-foreground`. Ít `dark:` class.

**Step 1: Kiểm tra và strip dark: còn sót**

```bash
grep -n "dark:" src/pages/workbench/index.vue
```

Strip bất kỳ `dark:*` nào còn lại.

**Step 2: Project top bar**

```vue
<!-- Hiện tại — đã OK -->
<div class="h-12 border-b border-border flex items-center justify-between px-6 shrink-0 bg-card/50 backdrop-blur-sm">

<!-- Không cần sửa -->
```

**Step 3: Main content area**

```vue
<!-- Hiện tại — đã OK -->
<div class="flex-1 flex flex-col overflow-hidden bg-background">
```

**Verify:** Sidebar hiện ra trắng/xám nhạt, icon active có bg-accent highlight đúng màu.

---

## TASK 3 — Screen: Project List (`src/views/project/index.vue`)

**Objective:** Cards dùng shadcn Card, strip `dark:bg-card dark:border-border` (redundant), empty state dùng shadcn pattern.

**Files:**
- Modify: `src/views/project/index.vue`
- Modify: `src/views/project/components/addProject.vue`
- Modify: `src/views/project/components/projectDialog.vue`

**Step 1: Strip dark: redundant khỏi Card**

```vue
<!-- Trước -->
<Card class="group relative cursor-pointer hover:shadow-xl hover:-translate-y-0.5 transition-all duration-200 dark:bg-card dark:border-border overflow-hidden">

<!-- Sau — dark:bg-card dark:border-border đã là default của Card, xoá đi -->
<Card class="group relative cursor-pointer hover:shadow-xl hover:-translate-y-0.5 transition-all duration-200 overflow-hidden">
```

**Step 2: Empty state — thay TDesign `<t-empty>` nếu có**

Project page chưa dùng `t-empty`, nhưng `addProject.vue` và `projectDialog.vue` có thể dùng `t-dialog`, `t-form`, `t-input`.

```vue
<!-- Thay t-dialog → dùng shadcn Dialog -->
<Dialog v-model:open="dialogShow">
  <DialogContent class="max-w-lg">
    <DialogHeader>
      <DialogTitle>{{ $t("workbench.project.newProject") }}</DialogTitle>
    </DialogHeader>
    <!-- form content -->
  </DialogContent>
</Dialog>
```

**Step 3: Thêm shadcn Dialog imports nếu chưa có**

```ts
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/Dialog'
// Nếu chưa có file → tạo từ shadcn-vue snippet
```

**Step 4: Form inputs trong addProject/projectDialog — đổi `t-input`, `t-select`, `t-textarea` → shadcn**

```vue
<!-- t-input → shadcn Input -->
<Input v-model="form.name" :placeholder="$t('...')" />

<!-- t-textarea → shadcn Textarea -->
<Textarea v-model="form.intro" rows="3" />

<!-- t-select → shadcn Select -->
<Select v-model="form.projectType">
  <SelectTrigger><SelectValue /></SelectTrigger>
  <SelectContent>
    <SelectItem value="novel">...</SelectItem>
    <SelectItem value="script">...</SelectItem>
  </SelectContent>
</Select>
```

**Verify:** Click "Dự án mới" → dialog trắng sáng, form sạch, không có TDesign widget nào.

---

## TASK 4 — Screen: Task Center (`src/views/task/index.vue`)

**Objective:** Toàn bộ screen này dùng `t-table`, `t-select`, `t-button`, `t-pagination` — cần thay hết sang shadcn + custom table.

**Files:**
- Modify: `src/views/task/index.vue`
- Remove: `src/views/taskList/index.vue` (legacy, dùng vxe-table — không dùng nữa nếu task/index.vue đã đủ)

**Step 1: Header section**

```vue
<!-- Trước -->
<div class="header">
  <div class="headerInfo fc">
    <span class="title">{{ $t("workbench.task.title") }}</span>
    <span class="sub">{{ $t("workbench.task.subtitle") }}</span>
  </div>
  <t-button @click="getTaskList">...</t-button>
</div>

<!-- Sau -->
<div class="flex items-center justify-between mb-6">
  <div>
    <h1 class="text-2xl font-bold text-foreground">{{ $t("workbench.task.title") }}</h1>
    <p class="text-sm text-muted-foreground mt-1">{{ $t("workbench.task.subtitle") }}</p>
  </div>
  <Button variant="outline" @click="getTaskList">
    <RefreshCw :size="16" class="mr-2" />
    {{ $t("workbench.task.refresh") }}
  </Button>
</div>
```

**Step 2: Filter selects — đổi `t-select` → shadcn Select**

```vue
<div class="flex items-center gap-4 mb-4">
  <Select v-model="projectId" @update:modelValue="onFilterChange">
    <SelectTrigger class="w-48">
      <SelectValue :placeholder="$t('workbench.task.project')" />
    </SelectTrigger>
    <SelectContent>
      <SelectItem v-for="opt in projectData" :key="opt.value" :value="opt.value">
        {{ opt.label }}
      </SelectItem>
    </SelectContent>
  </Select>
  <!-- repeat cho taskClass, taskState -->
</div>
```

**Step 3: Data table — đổi `t-table` → shadcn Table**

```vue
<div class="rounded-md border border-border overflow-hidden">
  <Table>
    <TableHeader>
      <TableRow>
        <TableHead class="w-32">{{ $t("workbench.task.col.taskClass") }}</TableHead>
        <TableHead>{{ $t("workbench.task.col.relatedObjects") }}</TableHead>
        <TableHead class="w-40">{{ $t("workbench.task.col.model") }}</TableHead>
        <TableHead>{{ $t("workbench.task.col.describe") }}</TableHead>
        <TableHead class="w-28">{{ $t("workbench.task.col.state") }}</TableHead>
        <TableHead class="w-40">{{ $t("workbench.task.col.startTime") }}</TableHead>
      </TableRow>
    </TableHeader>
    <TableBody>
      <TableRow v-for="row in taskList" :key="row.id" class="hover:bg-muted/50">
        <TableCell>{{ row.taskClass }}</TableCell>
        <TableCell>{{ row.relatedObjects }}</TableCell>
        <TableCell class="truncate max-w-[120px]">{{ row.model }}</TableCell>
        <TableCell class="truncate max-w-[200px]">{{ row.describe }}</TableCell>
        <TableCell>
          <Badge :variant="row.state === 'running' ? 'default' : row.state === 'failed' ? 'destructive' : 'secondary'">
            {{ row.state }}
          </Badge>
        </TableCell>
        <TableCell class="text-muted-foreground text-xs">
          {{ dayjs(row.startTime).format("YYYY-MM-DD HH:mm") }}
        </TableCell>
      </TableRow>
    </TableBody>
  </Table>
</div>
```

**Step 4: Pagination — đổi `t-pagination` → simple shadcn pagination**

```vue
<div class="flex items-center justify-between mt-4">
  <p class="text-sm text-muted-foreground">
    Tổng {{ pagination.total }} tác vụ
  </p>
  <div class="flex items-center gap-2">
    <Button variant="outline" size="sm" :disabled="pagination.page <= 1" @click="pagination.page--; getTaskList()">
      <ChevronLeft :size="16" />
    </Button>
    <span class="text-sm">{{ pagination.page }} / {{ Math.ceil(pagination.total / pagination.limit) }}</span>
    <Button variant="outline" size="sm" :disabled="pagination.page >= Math.ceil(pagination.total / pagination.limit)" @click="pagination.page++; getTaskList()">
      <ChevronRight :size="16" />
    </Button>
  </div>
</div>
```

**Step 5: Thêm Table components nếu chưa có**

```bash
# Kiểm tra
ls src/components/ui/ | grep -i table

# Nếu chưa có, tạo: Table.vue, TableHeader.vue, TableBody.vue, TableRow.vue, TableHead.vue, TableCell.vue
# Theo pattern của shadcn-vue: https://www.shadcn-vue.com/docs/components/table
```

**Verify:** Task list hiển thị đúng dữ liệu, table trắng sáng, badge màu theo trạng thái.

---

## TASK 5 — Screen: Novel / Văn bản gốc (`src/views/novel/index.vue`)

**Objective:** Thay `t-table`, `t-button`, `t-space`, `t-input` sang shadcn. Giữ nguyên pagination logic.

**Files:**
- Modify: `src/views/novel/index.vue`
- Modify: `src/views/novel/components/importNovel.vue`
- Modify: `src/views/novel/components/editNodel.vue`
- Modify: `src/views/novel/components/eventAnalysis.vue`

**Step 1: Toolbar buttons — thay `t-space` + `t-button`**

```vue
<!-- Trước -->
<t-space>
  <t-button theme="primary" @click="importNovelFn">
    <template #icon><t-icon name="add" /></template>
    {{ $t("workbench.novel.importText") }}
  </t-button>
  <t-button theme="danger" :disabled="...">...</t-button>
  <t-button @click="startEventAnalysis" :disabled="...">...</t-button>
</t-space>

<!-- Sau -->
<div class="flex items-center gap-2">
  <Button @click="importNovelFn">
    <Plus :size="16" class="mr-2" />
    {{ $t("workbench.novel.importText") }}
  </Button>
  <Button variant="destructive" :disabled="selectedRowKeys.length === 0" @click="handleBatchDelete">
    <Trash2 :size="16" class="mr-2" />
    {{ $t("workbench.novel.batchDelete") }}
    <span v-if="selectedRowKeys.length > 0" class="ml-1">({{ selectedRowKeys.length }})</span>
  </Button>
  <Button variant="outline" @click="startEventAnalysis" :disabled="selectedRowKeys.length === 0">
    <BarChart2 :size="16" class="mr-2" />
    {{ $t("workbench.novel.eventAnalysis") }}
    <span v-if="selectedRowKeys.length > 0" class="ml-1">({{ selectedRowKeys.length }})</span>
  </Button>
</div>
```

**Step 2: Search bar**

```vue
<!-- Trước -->
<t-input v-model="searchText" :placeholder="..." clearable style="width: 260px" />
<t-button @click="onChange">
  <template #icon><t-icon name="search" /></template>
  {{ $t("workbench.novel.search") }}
</t-button>

<!-- Sau -->
<div class="relative">
  <Search :size="16" class="absolute left-3 top-1/2 -translate-y-1/2 text-muted-foreground" />
  <Input v-model="searchText" :placeholder="$t('workbench.novel.searchPlaceholder')" class="pl-9 w-64" />
</div>
<Button variant="outline" @click="onChange">
  {{ $t("workbench.novel.search") }}
</Button>
```

**Step 3: Table — tương tự TASK 4**

Dùng shadcn `Table` + `TableHeader/Body/Row/Head/Cell`. Thêm checkbox column cho select.

**Step 4: `importNovel.vue` dialog — đổi `t-dialog` → shadcn Dialog**

**Step 5: `eventAnalysis.vue` — xem và đổi t-* components tương tự**

**Verify:** Novel list load đúng, select row, batch delete không bị lỗi.

---

## TASK 6 — Screen: Script / Kịch bản (`src/views/script/index.vue`)

**Objective:** Thay card grid `t-card` sang shadcn Card. Toolbar buttons đổi sang shadcn Button. Dialog addScript/editScript đổi sang shadcn.

**Files:**
- Modify: `src/views/script/index.vue`
- Modify: `src/views/script/components/addScript.vue`
- Modify: `src/views/script/components/editScript.vue`
- Modify: `src/views/script/components/batchAddScript.vue`

**Step 1: Toolbar — pattern giống Novel (xem TASK 5 Step 1)**

**Step 2: Script card grid — đổi `t-card` → shadcn Card**

```vue
<!-- Trước -->
<t-card shadow hover-shadow :style="{ width: '400px', cursor: 'pointer' }">
  <template #header>
    <div class="cardHeader">
      <span class="cardTitle">{{ item.name }}</span>
      <t-checkbox :checked="selectedIds.includes(item.id)" />
    </div>
  </template>
  <span class="content">{{ item.content }}</span>
</t-card>

<!-- Sau -->
<Card class="w-96 cursor-pointer hover:shadow-lg transition-shadow" @click="handleScriptClick(item)">
  <CardHeader class="pb-2">
    <div class="flex items-center justify-between">
      <CardTitle class="text-base font-semibold truncate">{{ item.name }}</CardTitle>
      <Checkbox
        :checked="selectedIds.includes(item.id)"
        @click.stop
        @update:checked="toggleSelect(item.id)"
      />
    </div>
  </CardHeader>
  <CardContent>
    <p class="text-sm text-muted-foreground line-clamp-4">{{ item.content }}</p>
  </CardContent>
</Card>
```

**Step 3: Empty state**

```vue
<!-- Thay t-empty -->
<div class="flex flex-col items-center justify-center py-24 text-muted-foreground">
  <FileText :size="48" class="mb-4 opacity-30" />
  <p class="text-sm">{{ $t("workbench.script.empty") }}</p>
</div>
```

**Step 4: addScript/editScript/batchAddScript dialogs — đổi t-dialog, t-form, t-input → shadcn**

Pattern: `Dialog > DialogContent > DialogHeader > form > Input/Textarea > DialogFooter > Button`

**Verify:** Script grid hiển thị cards sáng, select checkbox hoạt động, open dialog edit OK.

---

## TASK 7 — Screen: Assets / Tài nguyên (`src/views/assets/index.vue`)

**Objective:** Đổi `t-tabs` → shadcn Tabs, `t-button` → shadcn Button, `t-popup` → shadcn DropdownMenu, `t-input` → shadcn Input. Giữ nguyên asset grid logic.

**Files:**
- Modify: `src/views/assets/index.vue`
- Modify: `src/views/assets/components/addAssets.vue`
- Modify: `src/views/assets/components/addAudioAssets.vue`
- Modify: `src/views/assets/components/generateImage.vue`
- Modify: `src/views/assets/components/batchGeneration.vue`

**Step 1: Đổi `t-tabs` → shadcn Tabs**

```vue
<!-- Trước -->
<t-tabs v-model="assetOptions" @change="selectAssetOptions">
  <t-tab-panel v-for="item in themeData" :value="item.value">
    <template #label>
      <div class="tabLabel">
        <component :is="item.icon" size="20" />
        <span>{{ item.name }}</span>
      </div>
    </template>
    <!-- panel content -->
  </t-tab-panel>
</t-tabs>

<!-- Sau -->
<Tabs v-model="assetOptions" @update:modelValue="selectAssetOptions">
  <TabsList class="mb-4">
    <TabsTrigger v-for="item in themeData" :key="item.value" :value="item.value">
      <component :is="item.icon" :size="16" class="mr-2" />
      {{ item.name }}
    </TabsTrigger>
  </TabsList>
  <TabsContent v-for="item in themeData" :key="item.value" :value="item.value">
    <!-- panel content -->
  </TabsContent>
</Tabs>
```

**Step 2: Toolbar — `t-button`, `t-popup` (batch generate dropdown)**

```vue
<!-- t-popup → DropdownMenu -->
<DropdownMenu>
  <DropdownMenuTrigger asChild>
    <Button variant="outline">
      <AlignLeft :size="16" class="mr-2" />
      {{ $t("workbench.assets.batchGenerate") }}
      <ChevronDown :size="14" class="ml-2" />
    </Button>
  </DropdownMenuTrigger>
  <DropdownMenuContent>
    <DropdownMenuItem @click="batchGeneration(1)">
      {{ $t("workbench.assets.generatePrompt") }}
    </DropdownMenuItem>
    <DropdownMenuItem @click="batchGeneration(2)">
      {{ $t("workbench.assets.generateImage") }}
    </DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

**Step 3: Thêm Tabs components nếu chưa có**

```bash
ls src/components/ui/ | grep -i tab
# Nếu thiếu → tạo theo shadcn-vue Tabs pattern
```

**Step 4: Asset dialog — addAssets/addAudioAssets đổi t-dialog → shadcn Dialog**

**Verify:** Tab switch đúng, batch dropdown hiện menu, asset grid load ảnh OK.

---

## TASK 8 — Screen: Production Flow (`src/views/production/`)

**Objective:** Canvas (VueFlow) không cần đổi. Các node panels bên trong dùng TDesign — cần đổi từng panel sang shadcn. Đây là task phức tạp nhất.

**Files:**
- Modify: `src/views/production/node/script.vue`
- Modify: `src/views/production/node/scriptPlan.vue`
- Modify: `src/views/production/node/storyboardTable.vue`
- Modify: `src/views/production/node/storyboard.vue`
- Modify: `src/views/production/node/assets.vue`
- Modify: `src/views/production/node/workbench.vue`
- Modify: `src/views/production/components/rightChatBox/index.vue`
- Modify: `src/views/production/components/workbench/generate/index.vue`

**Step 1: Xác định TDesign usage trong từng node**

```bash
grep -rn "t-button\|t-input\|t-select\|t-table\|t-dialog\|t-card\|t-loading" \
  src/views/production/ --include="*.vue"
```

**Step 2: Node panels — wrapper class**

Mỗi node panel thường là `<div class="nodePanel">`. Đổi sang shadcn:

```vue
<!-- Wrapper -->
<div class="bg-card border border-border rounded-lg shadow-sm overflow-hidden">
  <!-- Header bar -->
  <div class="flex items-center justify-between px-4 py-2 border-b border-border bg-muted/30">
    <span class="text-sm font-semibold text-foreground">{{ title }}</span>
    <!-- action buttons -->
  </div>
  <!-- Content -->
  <div class="p-4">
    <!-- content -->
  </div>
</div>
```

**Step 3: Buttons trong node — đổi `t-button` → shadcn Button size="sm"**

Node panels thường nhỏ, dùng `size="sm"` và `size="icon"` cho icon-only buttons.

**Step 4: Loading state — đổi `t-loading` → Tailwind spinner**

```vue
<!-- Thay t-loading -->
<div v-if="loading" class="flex items-center justify-center py-8">
  <svg class="animate-spin h-6 w-6 text-muted-foreground" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
  </svg>
</div>
```

**Step 5: rightChatBox — đổi `t-input` và `t-button`**

Chat input:
```vue
<div class="flex items-center gap-2 p-3 border-t border-border">
  <Input v-model="message" :placeholder="$t('...')" class="flex-1" @keydown.enter="sendMessage" />
  <Button size="icon" @click="sendMessage">
    <Send :size="16" />
  </Button>
</div>
```

**Verify:** Mở 1 project → flow canvas load → click node → panel hiển thị đúng, không có TDesign widget.

---

## TASK 9 — Screen: Settings Dialog (`src/components/setting/index.vue` + sub-components)

**Objective:** Đổi `t-dialog` bao ngoài → shadcn Dialog. Sidebar menu `t-menu` → shadcn vertical nav. 16 sub-panel đổi từng cái.

**Files:**
- Modify: `src/components/setting/index.vue`
- Modify: `src/components/setting/components/*.vue` (16 files)

**Step 1: Wrapper dialog**

```vue
<!-- Trước -->
<t-dialog :header="$t('settings.title')" :footer="false" placement="center" width="1200px" v-model:visible="showSetting">
  <div class="settingPanel">
    <t-menu class="settingMenu" v-model:value="activeMenu" ...>

<!-- Sau -->
<Dialog v-model:open="showSetting">
  <DialogContent class="max-w-5xl h-[80vh] p-0 flex overflow-hidden">
    <DialogTitle class="sr-only">{{ $t('settings.title') }}</DialogTitle>

    <!-- Left nav -->
    <nav class="w-52 border-r border-border flex flex-col py-4 gap-1 shrink-0 overflow-y-auto">
      <button
        v-for="item in menuItems"
        :key="item.key"
        class="flex items-center gap-3 px-4 py-2 text-sm rounded-md mx-2 transition-colors hover:bg-accent hover:text-accent-foreground"
        :class="{ 'bg-accent text-accent-foreground font-medium': activeMenu === item.key }"
        @click="activeMenu = item.key"
      >
        <component :is="item.icon" :size="16" />
        {{ $t(item.label) }}
        <span v-if="needUpdate && item.key === 'about'" class="ml-auto w-2 h-2 bg-destructive rounded-full" />
      </button>
    </nav>

    <!-- Right content -->
    <div class="flex-1 overflow-y-auto p-6">
      <h2 class="text-lg font-semibold mb-4">{{ currentMenuItem ? $t(currentMenuItem.label) : "" }}</h2>
      <uiConfig v-if="activeMenu === 'ui'" />
      <!-- ... rest of sub-panels -->
    </div>
  </DialogContent>
</Dialog>
```

**Step 2: uiConfig.vue — đổi t-form, t-radio-group, t-color-picker**

```vue
<!-- Color mode radio -->
<div class="space-y-2">
  <Label>{{ $t('settings.ui.colorMode') }}</Label>
  <div class="flex gap-2">
    <Button v-for="mode in ['auto','light','dark']" :key="mode"
      :variant="themeSetting.mode === mode ? 'default' : 'outline'"
      size="sm"
      @click="themeSetting.mode = mode">
      {{ mode === 'auto' ? 'Tự động' : mode === 'light' ? 'Sáng' : 'Tối' }}
    </Button>
  </div>
</div>

<!-- Font size -->
<div class="space-y-2 mt-4">
  <Label>Cỡ chữ</Label>
  <div class="flex flex-wrap gap-2">
    <Button v-for="size in [12,13,14,16,18,20,22]" :key="size"
      :variant="themeSetting.fontSize === size ? 'default' : 'outline'"
      size="sm"
      @click="themeSetting.fontSize = size">
      {{ size }}px
    </Button>
  </div>
</div>
```

**Step 3: vendorConfig, requestConfig, loginConfig — đổi t-form + t-input**

Pattern chung:
```vue
<div class="space-y-4">
  <div class="space-y-2">
    <Label for="xxx">{{ $t('...') }}</Label>
    <Input id="xxx" v-model="value" placeholder="..." />
    <p class="text-xs text-muted-foreground">{{ $t('...hint') }}</p>
  </div>
  <Button @click="save">{{ $t('save') }}</Button>
</div>
```

**Step 4: vendorConfig — provider list (phức tạp nhất)**

VendorConfig thường có danh sách providers với logo. Giữ nguyên logic, chỉ đổi UI wrapper:
```vue
<!-- Provider card -->
<div class="border border-border rounded-lg p-4 flex items-center gap-4 hover:bg-muted/50">
  <img :src="logo" class="w-8 h-8 rounded" />
  <div class="flex-1 min-w-0">
    <p class="font-medium text-sm">{{ name }}</p>
    <p class="text-xs text-muted-foreground truncate">{{ description }}</p>
  </div>
  <Button variant="ghost" size="sm" @click="configure(provider)">Cấu hình</Button>
</div>
```

**Step 5: Các sub-panels còn lại (promptManage, skillManagement, memoryConfig, fileManagement, dbConfig, devConfig, modelMap, about, logoutConfig)**

Mỗi panel: strip `t-*` → shadcn tương ứng. Pattern lặp lại như các bước trên.

**Verify:** Settings dialog mở ra, nav menu click đúng, các form input nhập được, không có TDesign styling nào lọt ra.

---

## TASK 10 — Final: Cleanup + Build

**Objective:** Xoá TDesign khỏi dependencies nếu không còn dùng. Build production. Verify toàn bộ.

**Files:**
- Modify: `package.json`
- Modify: `src/main.ts`
- Modify: `vite.config.ts` (nếu có TDesign plugin)

**Step 1: Kiểm tra còn t-* nào sót không**

```bash
grep -rn "<t-" src/ --include="*.vue" | grep -v "node_modules"
```

**Step 2: Nếu còn sót** → fix từng chỗ theo pattern trên

**Step 3: Nếu sạch hoàn toàn — xoá TDesign import trong main.ts**

```ts
// Xoá
import TDesign from 'tdesign-vue-next'
import 'tdesign-vue-next/es/style/index.css'
app.use(TDesign)
```

**Step 4: Xoá khỏi package.json**

```bash
npm uninstall tdesign-vue-next
```

**Step 5: Build và check**

```bash
npm run build
# Expected: no errors
# Check dist/ size — nên nhỏ hơn đáng kể sau khi bỏ TDesign
```

**Step 6: Run dev + navigate hết screens**

```bash
npm run dev
# Login → Project → click project → Novel/Script/Assets/Production → Settings
# Mỗi screen: không có t-* widget nào, toàn bộ light mode, font Inter
```

**Step 7: Commit**

```bash
git add -A
git commit -m "feat: migrate all UI to shadcn, drop TDesign"
```

---

## Shadcn Components cần đảm bảo có trong `src/components/ui/`

| Component | Dùng ở đâu |
|-----------|-----------|
| Button | Tất cả screens |
| Input | Login, Task, Novel, Script, Settings |
| Label | Form fields |
| Card + CardHeader/Content/Footer | Project, Script |
| Badge | Task, Project |
| Dialog + DialogContent/Header/Title/Footer | Project, Novel, Script, Assets, Settings |
| Select + SelectTrigger/Content/Item | Task, Novel, Assets, Settings |
| Tabs + TabsList/Trigger/Content | Assets |
| Table + TableHeader/Body/Row/Head/Cell | Task, Novel |
| Checkbox | Novel, Script |
| Textarea | addScript, addProject |
| DropdownMenu + Trigger/Content/Item | Assets (batch), Project (3-dot) |
| Tooltip + Provider/Trigger/Content | Workbench sidebar (đã có) |

Kiểm tra nhanh:
```bash
ls src/components/ui/
```

Bất kỳ component nào thiếu → tạo theo [shadcn-vue docs](https://www.shadcn-vue.com/docs/components).
