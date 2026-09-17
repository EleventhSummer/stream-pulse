# Stream Pulse · 一周执行手册

这是你的第一份按实习标准走完的前端项目。功能很小，流程和文档按正式交付来。

对照计划总览：[一周看板](/Users/ashitaka/.cursor/projects/Users-ashitaka-02-Projects/canvases/stream-pulse-week-plan.canvas.tsx)（可在 Cursor 里打开在聊天旁边）。

- 需求合同：`docs/PRD.md`（范围已冻结，不要加需求）
- 接口合同：`docs/API.md`（先实现合同，再写页面）
- 每天结束对照「当天产物」，没有产物不算做完

建议节奏：每天 5–6 小时，第 7 天只收尾。卡在某工具上超过 40 分钟，先记进 `docs/DEVELOPMENT.md`，第二天再解决，不要用新功能逃避。

---

## 0. 为什么是这个项目

你的目标岗位是 TP-LINK / 腾讯 / 字节前端实习。现有能力是 Vue + JS + HTML + CSS + Python；缺口是 TypeScript、HTTP、Vite 工程化、Node、完整 Git 协作。

本项目用 **Vue 3 + TS + Vite** 把已会的框架用透，用一个约 80 行的 Express mock 补 Node 和 HTTP，用列表 + 详情 + 轮询 + 折线图覆盖组件开发、页面开发和可视化。

本周明确不做：

- React / Next.js（流程会了再换框架，否则两头不熟）
- 真 WebSocket、连麦、礼物、渲染引擎
- 注册、权限矩阵、后台 CRUD
- Webpack / Babel / Rollup 实操（面试能对比 Vite 即可）

---

## 1. 每天开始前的固定动作

1. 开 Issue（或继续昨天的 Issue），写清今天的验收标准。
2. 从 `main` 拉新分支：`feat/xxx` 或 `fix/xxx`。
3. 做完开 PR，用模板自审，合并后再开始下一天。
4. Commit 信息用 [Conventional Commits](https://www.conventionalcommits.org/)：

```
feat(list): 支持按直播状态筛选
fix(http): 401 时清除过期 token
docs(readme): 补充本地启动步骤
chore(ci): 增加 typecheck
```

---

## Day 1 · 脚手架与仓库（5h）

目标：空壳能跑，规范能卡住，仓库能给人看。

### 1.1 重建工程

当前目录里只有一次不完整的 Vite 缓存，从零建：

```bash
cd /Users/ashitaka/02_Projects/fronted-projects
# 先把旧的 stream-pulse 挪走或删掉 .vite 后再初始化
npm create vue@latest stream-pulse
```

交互选项：

- TypeScript：Yes
- Vue Router：Yes
- Pinia：Yes
- Vitest：Yes
- ESLint：Yes
- Prettier：Yes
- 不要选 JSX、不要选 Playwright（本周用不到）

然后：

```bash
cd stream-pulse
npm install
npm run dev
```

浏览器能打开默认 Vue 页即可。

### 1.2 初始化 Git 并推远程

```bash
git init
git add .
git commit -m "chore: init vue3 + ts + vite scaffold"
# 在 GitHub 新建私有或公开仓库 stream-pulse，再：
git branch -M main
git remote add origin <你的仓库 URL>
git push -u origin main
```

保护 `main`：GitHub → Settings → Branches → 要求 PR 才能合入（一个人练习也要开，强迫走流程）。

### 1.3 工程约定

补这些文件（内容按仓库模板抄，能跑就行）：

- `.editorconfig`
- `.vscode/settings.json`（保存时 format）
- `CONTRIBUTING.md`
- `.github/ISSUE_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `README.md` 骨架（项目名、技术栈、还没写的启动命令先占位）

确认三条命令都过：

```bash
npm run lint
npm run format
npx vue-tsc --noEmit
```

在 `package.json` 里加 `"typecheck": "vue-tsc --noEmit"`。

### 1.4 搞清这些东西是干什么的

打开文件，用自己的话写进 `docs/DEVELOPMENT.md`（各写 2–3 句）：

| 文件/目录 | 你要写明白的事 |
|---|---|
| `package.json` | scripts 和 dependencies 的区别 |
| `vite.config.ts` | 开发服务器和打包入口 |
| `tsconfig.json` | 为什么 `.vue` 能被 TS 检查 |
| `src/main.ts` | 应用怎么挂载 |
| `src/router` | 路由表 |
| `src/stores` | Pinia 放什么、不放什么 |
| `index.html` | 为什么入口是它而不是某个 `.vue` |

### Day 1 产物

- [ ] `npm run dev` 可访问
- [ ] GitHub 上有仓库和初始 commit
- [ ] lint / format / typecheck 能跑
- [ ] README 骨架 + CONTRIBUTING + Issue/PR 模板

---

## Day 2 · 类型、Mock、请求层（5h）

目标：页面还没有，但三条接口在 Chrome Network 里是绿的。

### 2.1 类型即合同

新建 `src/types/room.ts`、`src/types/auth.ts`，字段必须和 `docs/API.md` 一致，不要自己发明。

### 2.2 Mock 服务（补 Node.js）

在仓库里加 `server/index.mjs`（或 `.ts`），只用 `express` + `cors`。实现 API 文档里的 3 个接口，数据写死 12 个房间即可。

`package.json`：

```json
"scripts": {
  "dev": "vite",
  "mock": "node server/index.mjs",
  "typecheck": "vue-tsc --noEmit"
}
```

两个终端：

```bash
npm run mock   # 例如 http://127.0.0.1:3456
npm run dev    # 例如 http://127.0.0.1:5173
```

在 `vite.config.ts` 配 proxy，把 `/api` 转到 mock。这样浏览器只打同源 `/api`，你能亲眼看到一次 **没有 proxy 时的 CORS 报错**，再打开 proxy 对比。把这次对比写进 `docs/DEVELOPMENT.md`。

### 2.3 请求封装

`src/api/http.ts`：

- `baseURL`
- `timeout`（可用 `AbortController`）
- 从 `localStorage` 带 `Authorization`
- 非 2xx 抛出自定义 `HttpError`（带 `status` 和 `message`）
- 401 清除 token（可先 `console.warn`，Day 5 再接路由）

`src/api/rooms.ts`、`src/api/auth.ts` 只调用 `http`，不直接 `fetch`。

### 2.4 验证

临时在 `App.vue` 里 `onMounted` 调 `getRooms()`，打开 DevTools → Network：

1. `POST /api/login` 成功
2. `GET /api/rooms` 带回房间数组
3. `GET /api/rooms/:id` 成功
4. 把 mock 关掉，确认页面能拿到错误而不是白屏

验证完删掉 `App.vue` 里的临时调用。

### Day 2 产物

- [ ] mock 独立可启动
- [ ] 三条接口 Network 可截图
- [ ] `docs/DEVELOPMENT.md` 有 CORS 和 401 记录
- [ ] 页面仍可以是默认壳，但 `src/api` 已成型

---

## Day 3 · 列表页与基础组件（6h）

目标：搜索、筛选、分页、三种状态都能演示。

### 3.1 组件（每个文件只做一件事）

| 组件 | 职责 | 不要做 |
|---|---|---|
| `StatusBadge` | 直播中 / 未开播 / 封禁 | 不发请求 |
| `SearchBar` | 抛出 `submit` 事件 | 不调 API |
| `EmptyState` | 无数据文案 + 可选操作槽 | 不判断业务 |
| `ErrorState` | 错误文案 + 重试按钮 | 不自己 retry 逻辑 |
| `RoomCard` | 展示一个房间摘要 | 不拉列表 |

用 TS 写 `defineProps` / `defineEmits`。`StatusBadge` 第二天要测，逻辑放纯函数更好。

### 3.2 列表页

- 路由 `/rooms`
- Pinia：`query`、`status`、`page`，刷新不丢可用 `sessionStorage` 或保持在 store
- 进入页面和筛选变化时请求 `GET /api/rooms`
- 三种状态互斥：loading / empty / error / success
- 点击卡片 `router.push(/rooms/:id)`

### Day 3 产物

- [ ] 能搜到「星」字样的房间
- [ ] 筛「直播中」只剩 live
- [ ] mock 关掉出现 ErrorState，点重试能恢复
- [ ] 无匹配结果出现 EmptyState

---

## Day 4 · 详情、图表、轮询（6h）

目标：详情像一个真的运营看板，且离开页面后不再打接口。

### 4.1 详情页 `/rooms/:id`

- KPI：在线人数、点赞、开播时长
- 最近 10 条评论
- ECharts 折线图：`viewerTrend`（12 个点）
- 非法 id → 404 页

图表数据必须来自接口。窗口 resize 时图表要 `resize`，组件卸载时 `dispose`。

### 4.2 轮询

仅当 `status === 'live'` 时每 8 秒重新拉详情。

```ts
onUnmounted(() => {
  window.clearInterval(timer)
})
```

用 Network 证明：停在详情会持续出现请求；点回列表后请求停止。截图进 PR。

### Day 4 产物

- [ ] 直播中房间数字会变
- [ ] 离开详情后轮询停止
- [ ] 不存在的 id 进 404
- [ ] 图表随数据更新

---

## Day 5 · 登录、守卫、测试（5h）

### 5.1 鉴权

- `/login`：表单，成功后存 token，跳 `/rooms`
- 路由守卫：无 token 访问业务页 → `/login?redirect=...`
- 登录后跳回 redirect
- 退出登录清 token
- Day 2 的 401 处理改为真正跳登录页

账号见 `docs/API.md`：`demo / demo123`。

### 5.2 测试（至少 2 个）

```bash
npm run test
```

建议：

1. `StatusBadge`：传入 `live` 渲染出「直播中」
2. `HttpError` 或状态映射函数：`401` → 指定文案

不要求 E2E。单测的目的是让你知道 CI 为什么要跑 `test`。

### Day 5 产物

- [ ] 无 token 进 `/rooms` 会被踢回登录
- [ ] `npm run test` 通过
- [ ] 退出后再进详情进不了

---

## Day 6 · CI、文档、部署（5h）

### 6.1 GitHub Actions

`.github/workflows/ci.yml`：在 Node 20 上执行

```
npm ci
npm run lint
npm run typecheck
npm run test
npm run build
```

推到 PR 上必须看到绿勾。

### 6.2 文档写完

- `README.md`：项目简介、截图、技术栈、本地启动（两个命令）、在线 Demo 链接、目录说明
- `docs/ARCHITECTURE.md`：一张数据流（页面 → api → http → proxy → mock）
- `CHANGELOG.md`：按 Keep a Changelog，至少有 `0.1.0`

### 6.3 部署

前端 build 产物部署到 **GitHub Pages** 或 **Vercel**。mock 没有公网时：

- 方案 A（推荐练习）：用 Vite 的 `import.meta.env.PROD` 在生产环境走 `src/mocks/static.ts`（内存数据），开发环境走真实 HTTP。在 ARCHITECTURE 里写清楚为什么这样做。
- 方案 B：mock 也部署（Railway / Render），前端改 `VITE_API_BASE`。

面试时能讲「开发和生产数据源不同」比硬上一个随时挂的免费后端更加分。

### Day 6 产物

- [ ] PR 上 CI 全绿
- [ ] README 有截图和 Demo 链接
- [ ] 手机或另一台电脑能打开 Demo

---

## Day 7 · 打磨与复盘（4h）

只修边角，不加功能。

- 空态文案、按钮禁用、图表在窄屏下的高度
- 基础 a11y：按钮能 tab、图片有 alt
- 写 `docs/RETRO.md`（模板在下面）
- 录 60–90 秒屏幕：登录 → 筛选 → 详情轮询 → 断 mock 重试
- 准备三条口述（见文末）

### Day 7 产物

- [ ] `docs/RETRO.md`
- [ ] Demo 视频或 GIF
- [ ] 仓库 README 把 Demo 放在最上面

---

## 目录建议（Day 3 结束时应接近这样）

```
stream-pulse/
  .github/
    ISSUE_TEMPLATE.md
    PULL_REQUEST_TEMPLATE.md
    workflows/ci.yml
  docs/
    PRD.md
    API.md
    ARCHITECTURE.md
    DEVELOPMENT.md
    RETRO.md
  server/
    index.mjs          # mock API
  src/
    api/               # http + rooms + auth
    components/        # 纯 UI 组件
    mocks/             # 生产环境兜底数据（D6）
    router/
    stores/
    types/
    views/             # Login / RoomList / RoomDetail / NotFound
  HANDBOOK.md
  README.md
  CHANGELOG.md
  CONTRIBUTING.md
```

---

## 复盘模板（D7 复制到 `docs/RETRO.md`）

```markdown
# 复盘

## 我实际做完了什么
## 计划里没做完的（以及为什么）
## 每个工具我现在能怎么解释
## 踩过的两个坑（现象 → 原因 → 处理）
## 如果再给 3 天，我会改什么（仍然不要加需求，只改结构或测试）
## 面试 3 问自答
```

面试口述（对着仓库讲，不要背概念）：

1. **组件**：为什么 `StatusBadge` 不发请求，列表的 loading 状态放在哪一层。
2. **请求**：proxy 解决了什么；401 从拦截器到回到登录页的路径。
3. **工程化**：Vite 开发时做了什么；CI 四步各自挡住什么低级错误。

---

## 卡住时怎么问（包括问 AI）

先提供这四样：你要达成的验收标准、相关文件路径、报错原文、你已经试过的一步。不要问「帮我把项目做完」。本手册的目的就是让你自己走完流程。
