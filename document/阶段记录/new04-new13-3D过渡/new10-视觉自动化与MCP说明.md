# new(10) 视觉自动化与 MCP 接入说明

## 目标

这轮调整把原本只能靠人工截图描述的 3D 页面，转换成了可被 Codex CLI 直接调用的自动化入口：

1. 固定脚本截图
2. 运行时对象诊断
3. `.splinecode` 文件静态分析
4. 本地 MCP 工具封装
5. 可选的 computer use 本地代理

## 入口命令

```sh
npm run vis:check
npm run vis:baseline
npm run scene:report
npm run scene:strings
npm run scene:snap -- --label close --zoom 1.2 --wait 3000
npm run mcp:visual
npm run computer:local -- --prompt "查看 3D 页面并说明遮挡问题"
```

## 输出目录

- `artifacts/visual/current/`
- `artifacts/visual/baseline/`
- `artifacts/scene/captures/`
- `artifacts/scene/scene-report.json`
- `artifacts/scene/scene-strings.json`

## Codex CLI 使用建议

### 1. 日常视觉回归

让 Codex 直接执行：

```sh
npm run vis:check
```

然后读取：

- `artifacts/visual/current/manifest.json`
- `artifacts/visual/current/*.png`

### 2. 只看 3D 页

```sh
npm run scene:report
```

Codex 后续只需要读取：

- `artifacts/scene/scene-report.json`
- `artifacts/scene/captures/*.png`

### 3. 挂载 MCP

```sh
codex mcp add localVisualInspector -- node /home/just_monika/win_share/event/new(10)/mcp/visual-inspector-server.mjs
```

重启 Codex CLI 后，模型可以直接调用本地工具抓截图、生成报告，而不需要你先手工截图。

## 边界

### 已解决

- Codex 无法直接看宿主机屏幕的问题
- 3D 页面缺少可复现截图入口的问题
- 缺少统一 CLI / MCP 工具的问题

### 仍然保留

- `computer use` 依赖 `OPENAI_API_KEY`
- 首次运行 Playwright 可能需要安装 Chromium
- Spline runtime 仍可能从 CDN 拉取内部 wasm 资源
