# 说明文档13-new(10)-视觉自动化改动说明

## 1. 本次改动目的

本轮改动的目标不是继续强化“人工截图后再描述问题”的旧流程，而是把 `new(10)` 改造成一个适合 Codex CLI 持续参与视觉开发的工程副本。重点解决以下问题：

1. 3D 页面是黑盒组件，模型无法直接拿到稳定的运行时信息。
2. 视觉问题没有固定截图入口，导致每次排查都依赖手工操作。
3. Spline 场景缺少结构化分析手段，难以快速判断对象、事件和运行状态。
4. 后续开发缺少统一规范，容易重新退回低效的截图往返流程。

## 2. 本次落地内容

### 2.1 3D 页面运行时改造

将原本的 `spline-viewer` 标签壳子改为基于项目内 vendoring 的 `@splinetool/runtime` 直接挂载到 `canvas` 上。

改造后新增能力：

- 支持调试面板显示加载状态、对象数、可见对象数、事件对象数、对象名样本。
- 支持通过 URL 查询参数控制场景：
  - `diagnostic`
  - `wait`
  - `zoom`
  - `background`
  - `event`
  - `target`
- 新增实验控制台双屏覆盖层：保留 Spline 里的控制台和屏幕几何区域，真正的数字显示改由 Vue/HTML 覆盖。
- 双屏覆盖层默认直接读取 Pinia 实验状态中的电流与电压，避免在 Spline 文本或贴图里重复维护一套显示状态。
- 双屏覆盖层支持以下 URL 查询参数，便于通过脚本微调对位：
  - `screenLeft`
  - `screenTop`
  - `screenWidth`
  - `screenRotate`
  - `screenSkewX`
  - `screenOpacity`
- 在浏览器运行时注入 `window.__VISUAL_INSPECTOR__`，供 Playwright、CLI 脚本、后续自动化工具复用。

主要文件：

- `src/components/SplineScene.vue`
- `src/views/ThreeDView.vue`

### 2.2 视觉检查流水线

新增一套固定场景、固定视口、固定等待时序的视觉检查脚本，用来替代零散截图。

已实现命令：

- `npm run vis:check`
- `npm run vis:baseline`

默认输出：

- `artifacts/visual/current/`
- `artifacts/visual/baseline/`

其中 `manifest.json` 会记录：

- 截图标签
- 页面 URL
- 视口类型
- 文件路径
- 图片哈希
- 3D 页运行时诊断结果

### 2.3 Headless 场景分析

新增两类面向 Spline 场景的分析入口：

1. 运行时分析
   - `npm run scene:report`
   - 输出 3D 场景截图与对象统计
2. 静态文件分析
   - `npm run scene:strings`
   - 直接解析 `resource/scene.splinecode` 中的字符串、UUID、URL 和对象命名线索

默认输出：

- `artifacts/scene/scene-report.json`
- `artifacts/scene/scene-strings.json`
- `artifacts/scene/captures/*.png`

### 2.4 本地 computer use 代理

新增 `npm run computer:local`，它使用：

- Playwright 打开本地页面
- Responses API 发送截图
- 模型返回浏览器动作
- 脚本继续执行点击、输入、滚动、等待

这条链路用于“让模型看页面并在隔离浏览器内操作”，而不是直接控制宿主机桌面。

### 2.5 本地 MCP 服务

新增 `mcp/visual-inspector-server.mjs`，作为本地视觉工具服务，暴露这些工具：

- `capture_visual_matrix`
- `capture_scene_snapshot`
- `build_scene_report`
- `inspect_scene_file`

后续 Codex CLI 可以把这份项目直接挂成 MCP，然后在会话里直接调工具抓图和拿报告。

## 3. 本次新增或重点修改文件

### 新增

- `AGENTS.md`
- `scripts/lib/project-paths.mjs`
- `scripts/lib/preview-server.mjs`
- `scripts/lib/scene-introspection.mjs`
- `scripts/lib/visual-workflows.mjs`
- `scripts/visual-check.mjs`
- `scripts/scene-report.mjs`
- `scripts/scene-strings.mjs`
- `scripts/scene-snap.mjs`
- `scripts/computer-use-runner.mjs`
- `mcp/visual-inspector-server.mjs`
- `document/阶段记录/new04-new13-3D过渡/new10-视觉自动化与MCP说明.md`
- `document/阶段记录/new04-new13-3D过渡/new10-视觉自动化改动说明.md`

### 重点修改

- `src/components/SplineScene.vue`
- `src/views/ThreeDView.vue`
- `package.json`
- `vite.config.js`
- `README.md`
- `.gitignore`

## 4. 已完成验证

本轮已经实际执行并确认以下命令可用：

```sh
npm run build
npm run scene:strings
npm run vis:check
npm run scene:report
```

验证结果包括：

- 视觉矩阵截图已生成
- 视觉基线已生成
- 场景报告已生成
- Spline 文件静态分析已生成

已得到的关键运行时信息：

- 3D 场景对象数：`46`
- 可见对象数：`46`
- 事件对象数：`1`
- 对象名样本：
  - `Point Light`
  - `Camera`
  - `all4`
  - `Room`
  - `Body_Main`
  - `Body_Main_1`
  - `Body_Main_2`
  - `Body_Main_3`

## 5. 后续开发建议

### 5.1 视觉改动的默认流程

后续涉及 3D 页面或视觉问题时，建议统一使用以下流程：

1. 修改代码
2. 运行 `npm run vis:check`
3. 读取 `artifacts/visual/current/manifest.json`
4. 如只关注 3D 页，再运行 `npm run scene:report`
5. 需要更新基线时，再执行 `npm run vis:baseline`

### 5.2 需要模型主动看页面时

优先使用：

```sh
npm run computer:local -- --prompt "说明 3D 页面当前的布局和遮挡问题"
```

前提：

- 已设置 `OPENAI_API_KEY`
- 本机可访问 OpenAI API

### 5.3 需要 Codex 直接调工具时

先挂载本地 MCP：

```sh
codex mcp add localVisualInspector -- node /home/just_monika/win_share/event/new(10)/mcp/visual-inspector-server.mjs
```

然后重启 Codex CLI。

## 6. 风险与边界

- `resource/scene.splinecode` 仍是二进制资产，结构可观测性有限。
- `computer use` 依赖外部 API 和安全检查，不能视为完全无人值守方案。
- Playwright 首次执行仍可能要求安装浏览器。
- 当前 MCP 服务脚本已落地，但实际接入时仍需在 Codex CLI 中显式添加并重启会话。

## 7. 结论

`new(10)` 已从“需要频繁人工截图协助”的 3D 项目，提升为“可以脚本化截图、可运行时诊断、可生成场景报告、可通过 MCP 或 computer use 继续扩展”的开发副本。后续开发应以这套流程为默认路径，而不是回到手工截图往返。
