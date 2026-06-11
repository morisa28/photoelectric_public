# 阶段目标

本阶段执行整合后的底层回归校验，确认阶段 01 到阶段 06 的页面升级、AI 辅助、2D 虚拟实验台、数据处理和后端独立工程没有破坏 `animation-demo` 的主物理模型、原理动画、3D/Spline 链路和页面导航。

# 校验范围

重点检查以下区域：

- `src/stores/experiment.js`
- `src/stores/principleAnimation.js`
- `src/features/experiment/photoelectricModel.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`
- `src/features/photoelectric-animation/*`
- `src/components/PrincipleAnimationSection.vue`
- `src/components/SplineScene.vue`
- `src/components/animation/ThreeDMicroAnimationPanel.vue`
- `src/components/VirtualLab.vue`
- `src/components/IntegratedVirtualConsole.vue`
- `src/components/PhotoelectricEffectGraph.vue`
- `src/components/ExperimentReport.vue`
- `src/components/HistoryRecords.vue`
- `src/components/AIAssistant.vue`
- `src/views/HomeView.vue`
- `backend/`

# 差异与红线检查

差异统计：

```bash
git diff --stat origin/animation-demo...HEAD -- src/stores src/features src/components src/views backend document/阶段记录/分支整合
```

结果：差异范围符合阶段 01 到阶段 06 的合入计划。未出现原理动画、3D 场景、Spline 运行时、实验物理模型文件被覆盖。

变更文件检查：

```bash
git diff --name-only origin/animation-demo...HEAD -- src/stores src/features src/components src/views | sort
```

结果：主实验红线相关变更仅集中在：

- `src/stores/experiment.js`：评分/题库/复测/反馈辅助状态。
- `src/features/experiment/helpers.js`：复测截止电压平均与标准差聚合。
- `src/views/HomeView.vue`：数据处理页局部挂载 AI 教学洞察，保留导航和 3D 入口。

未出现在变更列表中的关键文件包括：

- `src/features/experiment/photoelectricModel.js`
- `src/features/experiment/constants.js`
- `src/stores/principleAnimation.js`
- `src/features/photoelectric-animation/engine.js`
- `src/features/photoelectric-animation/mapping.js`
- `src/features/photoelectric-animation/voltAmpTrajectory.js`
- `src/components/PrincipleAnimationSection.vue`
- `src/components/SplineScene.vue`
- `src/components/animation/ThreeDMicroAnimationPanel.vue`
- `resource/scene.splinecode`
- `vendor/spline-runtime/`

红线符号扫描：

```bash
rg -n "tanh\(\(voltage \+ cutoff\)|PHOTOCURRENT_SATURATION_CURRENT_MIN_AMP|PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP|NON_ENCLOSING_PHOTOTUBE_MODEL|CUTOFF_MODE_CATHODE_TAIL|resolveAnodeParasiticPhotocurrentAmp|resolveCathodePhotocurrentAmp|resolveNormalizedLightFlux" src
```

结果：

- 未发现 `tanh((voltage + cutoff)` 旧模型。
- `NON_ENCLOSING_PHOTOTUBE_MODEL`、`CUTOFF_MODE_CATHODE_TAIL`、`resolveAnodeParasiticPhotocurrentAmp`、`resolveCathodePhotocurrentAmp`、`resolveNormalizedLightFlux` 均仍存在于主物理链路、原理动画映射和相关测试中。
- `src/utils/photoelectric.js` 中仍有 `PHOTOCURRENT_SATURATION_CURRENT_MIN_AMP` / `MAX`，但该文件不是本次整合改动文件，属于 `animation-demo` 原有旧工具文件，不进入当前主实验 store 计算链路。

禁止目录检查：

```bash
git diff --name-only origin/animation-demo...HEAD | rg '(^|/)\\.vite/|(^|/)outputs/|report_assets|node_modules|(^|/)\\.env$|package-lock|backend/api/aiAPI'
```

结果：无输出。未引入 `.vite`、`outputs`、`report_assets`、`node_modules`、真实 `.env`、lockfile 或后端 AI API。

# 自动验证命令与结果

全量单元测试：

```bash
npm run test:unit -- --run
```

结果：通过。21 个测试文件通过，220 个测试通过。

生产构建：

```bash
npm run build
```

结果：通过。Vite 仍提示大 chunk 警告，但构建成功。

视觉检查：

```bash
npm run vis:check -- --port 4182
```

结果：通过。使用独立端口 `4182`，避免默认端口和阶段 05 遗留预览互相影响。`baselineStatus.changed` 包含：

- `home-desktop.png`
- `home-mobile.png`
- `3d-overview.png`
- `3d-mobile.png`
- `3d-zoom-close.png`

这些 hash 差异来自阶段 01 到阶段 05 的页面升级和截图重捕；检查过程中未出现 missing 或 added 基线项。3D 诊断显示：

- `bridgeAvailable: true`
- `ready: true`
- `error: null`
- `objectCount: 70`
- `visibleObjectCount: 70`

3D 场景报告：

```bash
npm run scene:report -- --port 4183
```

结果：通过。Spline scene 哈希仍为 `e32f07f2c0d641d76bf7d9df1003b160d03a8a959bb85ed89be7986aec90120b`，运行时 bridge ready、无错误、70 个对象可见，左右屏幕 canvas 纹理仍已绑定。

后端静态检查沿用阶段 06 结果：

```bash
node --check backend/server.js
node --check backend/api/knowledgeGraphAPI.js
node --check backend/neo4j/config/neo4j.config.js
node --check backend/neo4j/queries/cypherQueries.js
node --check backend/neo4j/scripts/initKnowledgeGraph.js
node --check backend/test-server.js
```

结果：通过。

# 页面冒烟验证

使用 Playwright 对 `http://127.0.0.1:4181/` 做轻量冒烟检查。该预览来自阶段 05 页面验证后遗留监听，前端代码在阶段 06 后未变更，因此可用于页面状态复查。

检查结果：

- 顶部导航包含知识图谱、虚拟实验台、原理动画、数据处理和 3D 实验台入口。
- AI 助手在知识图谱页显示。
- 切到原理动画页后，`.principle-animation-section` 可见，模式按钮包含“截止电压”和“伏安特性”。
- AI 助手在原理动画页隐藏，未遮挡动画页。
- 切到虚拟实验台后，`.lab-device-panel`、`.console-shell`、`.knowledge-bank-panel` 均可见。
- 切到数据处理并打开实验报告后，`.report-panel` 可见。
- 点击“AI 生成初稿”后，报告结论文本长度为 69，fallback 初稿正常。
- 知识图谱 `.network-container` 可见。
- 后端未启动时，知识图谱直接加载本地图谱，首次加载显示 `25 / 25` 本地节点。
- 切换离开知识图谱后再返回，等待图谱初始化完成后仍恢复到 `25 / 25` 本地节点。

控制台中看到 3 到 4 条 `net::ERR_CONNECTION_REFUSED`，均为未启动知识图谱后端时请求 `/api/knowledge-graph/*` 的预期失败，前端 fallback 正常，没有错误覆盖层。

# 发现的问题与处理方式

- 冒烟脚本第一次在切回知识图谱后立即读取筛选统计，短时间读到 `0 / 0`。复测时等待图谱初始化完成，确认可恢复到 `25 / 25`。结论：异步等待不足，不是功能缺陷。
- `vis:check` 和 `scene:report` 均使用独立端口运行；阶段 05 的 `4181` 预览仍在监听。此前尝试通过外部权限关闭该进程时被自动审批系统拒绝，未继续绕路处理。
- 阶段 06 未安装后端依赖，因此没有进行真实 Neo4j 动态接口实测；本阶段只验证后端语法与前端无后端 fallback。

# 未完成 / 风险

- 未执行真实 Neo4j 启动、初始化和动态图谱接口验证。需要用户确认后在 `backend/` 安装依赖并提供本地 Neo4j 配置。
- 视觉基线 hash 已变化但未更新 baseline；当前只记录变化和运行结果，是否更新基线留到后续明确任务。
- 4181 端口可能仍由阶段 05 预览占用，不影响代码和提交，但可能影响用户后续本地端口选择。

# 结论

阶段 07 没有发现需要修复的业务代码问题。当前整合分支保留了 `animation-demo` 的主物理模型、原理动画、3D/Spline 链路、主页导航和无后端知识图谱 fallback。
