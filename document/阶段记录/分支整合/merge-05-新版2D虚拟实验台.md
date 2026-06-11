# 阶段目标

本阶段从 `origin/ai-helper` 合入新版 2D 虚拟实验台、集成控制台、题库挂载和复测展示能力。

阶段目标是升级 `virtualLab` 页面的 2D 操作体验，同时继续复用 `animation-demo` 的主实验 store、真实物理模型、3D 同步状态和数据分析链路。

# 来源分支与参考内容

- 主要来源：`origin/ai-helper`
- 参考组件：`src/components/VirtualLab.vue`、`src/components/IntegratedVirtualConsole.vue`
- 参考 composable：`src/composables/useVirtualLabController.js`
- 参考测试：`src/components/__tests__/virtualLabLayout.spec.js`
- 依赖上一阶段已合入内容：`KnowledgeQuestionBank.vue`、`AIOperationScore.vue`、题库进度、评分历史和实验归档辅助状态

# 实际改动文件

- `src/components/VirtualLab.vue`
- `src/components/IntegratedVirtualConsole.vue`
- `src/components/__tests__/virtualLabLayout.spec.js`
- `src/composables/useVirtualLabController.js`
- `src/features/experiment/helpers.js`
- `src/stores/experiment.js`
- `src/stores/experiment.spec.js`
- `document/阶段记录/分支整合/merge-05-新版2D虚拟实验台.md`

# 涉及的关键组件、store、service、feature

- `VirtualLab.vue`：新版 2D 台面，包含光电管距离轨道、独立汞灯模块、试验仪电源、滤色片、光阑、实验流程、控制台和题库。
- `IntegratedVirtualConsole.vue`：新版控制台，包含接线/调零/波长/量程/模式/电压精度/复测起点/记录/自动扫描/保存/重置/拟合预览。
- `KnowledgeQuestionBank.vue`：挂载在 2D 虚拟实验台底部，当前轮 6 题批量提交。
- `useVirtualLabController.js`：继续作为 2D 页面事件与 `useExperimentStore` 的桥接层，本阶段只把试验仪按钮改为真正 toggle。
- `resetCutoffVoltageForRepeat()`：新增复测入口，把测量模式切回截止电压、精度设为 `0.001 V`、电压回到 `-1.998 V`，并递增 `measurementAttemptId`。
- `recordData()`：新增 `appliedVoltage`、`repeatMeasurementIndex`、`measurementAttemptId`、`measurementSignature` 等记录字段，用于复测展示和后续报告/评分。
- `buildFitRecords()`：同一波长多次有效截止电压先求平均和标准差，再进入 Uc-v 拟合；保留 `repeatCount` 和 `repeatedRecordIds`。

# 明确保留的 animation-demo 底层内容

本阶段没有修改以下文件：

- `src/features/experiment/photoelectricModel.js`
- `src/features/experiment/constants.js`
- `src/stores/principleAnimation.js`
- `src/features/photoelectric-animation/*`
- `src/components/PrincipleAnimationSection.vue`
- `src/components/SplineScene.vue`
- `src/components/animation/ThreeDMicroAnimationPanel.vue`
- `src/views/HomeView.vue`
- `resource/scene.splinecode`
- `vendor/spline-runtime/`

`src/stores/experiment.js` 和 `src/features/experiment/helpers.js` 属于红线相关文件，本阶段只做以下辅助扩展：

- 增加复测尝试编号、复测签名和记录字段。
- 增加复测起点 action。
- 让拟合输入在同一波长重复测量时使用平均截止电压。
- 补充对应单元测试。

未修改：

- `resolveCathodePhotocurrentAmp()`
- `generatedPhotocurrentAmp`
- `measuredPhotocurrentAmp`
- `currentPhotocurrentAmp`
- `cathodePhotocurrentModel`
- `anodeParasiticPhotocurrentAmp`
- `currentCutoffVoltage`
- `physicalLightFluxRatio`
- `photoelectricModel.js` 中的真实物理模型
- 3D/Spline 场景文件与运行时

# 本阶段没有合入的内容及原因

- 未整支 merge `2d-logic` 或 `ai-helper`；本阶段只选择性迁移新版 2D 页和控制台相关文件。
- 未引入 `origin/ai-helper` 或 `origin/2d-logic` 的旧简化光电流公式，避免覆盖当前主线物理模型。
- 未修改 `HomeView.vue` 导航结构；`VirtualLab.vue` 仍通过既有 `currentSection === 'virtualLab'` 路径渲染。
- 未修改原理动画和 3D 实验台入口；`AIAssistant` 在原理动画页隐藏的规则继续保留。
- 未启动或合入知识图谱后端；页面中知识图谱请求失败时继续走阶段 01 的本地 fallback。

# 验证命令与结果

阶段 05 相关测试：

```bash
npm run test:unit -- --run src/components/__tests__/virtualLabLayout.spec.js src/stores/experiment.spec.js
```

结果：通过。2 个测试文件通过，37 个测试通过。覆盖：

- 新版 2D 页面布局、题库挂载、按钮文案和汞灯独立布局。
- 控制台器材诊断、手动/自动模式切换、复测标签展示。
- AI 助手、AI 评分、历史记录的阶段 05 关联布局断言。
- 复测记录平均、标准差、重复次数和复测起点状态。

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
npm run vis:check -- --port 4179
```

结果：通过。首次使用默认端口时发现 `4173` 已被旧预览占用，为避免复用旧服务，改用 `4179` 独立端口执行。`baselineStatus.changed` 包含 `home-desktop.png`、`home-mobile.png`、`3d-overview.png`、`3d-mobile.png`、`3d-zoom-close.png`，记录为阶段 05 页面变更和截图重捕导致的基线差异。3D 诊断结果：

- `bridgeAvailable: true`
- `ready: true`
- `error: null`
- `objectCount: 70`
- `visibleObjectCount: 70`

3D 场景报告：

```bash
npm run scene:report -- --port 4180
```

结果：通过。报告显示 Spline bridge 可用、3D ready、无错误、70 个对象可见，左右屏幕 canvas 纹理仍已绑定。

渲染与交互验证：

```bash
npm run preview -- --host 127.0.0.1 --port 4181
node --input-type=module -e "<Playwright 虚拟实验台检查脚本>"
```

结果：通过。Browser 插件当前不可用，按前端测试技能回退到常规 Playwright。检查到：

- 页面标题为“光电效应实验模拟仿真平台”。
- `h2` 为“虚拟实验台”。
- `.lab-device-panel`、`.console-shell`、`.knowledge-bank-panel` 均可见。
- 题库卡片数量为 6。
- 试验仪电源按钮可从“试验仪已开启”切换为“开启试验仪”。
- 填写本轮 6 题后，批量提交按钮可用，提交后显示“已答 6/6”。
- 截图保存到 `/tmp/stage05-virtual-lab-submitted.png`，未提交。

辅助检查：

```bash
rg -n "tanh\(\(voltage \+ cutoff\)|PHOTOCURRENT_SATURATION_CURRENT_(MIN|MAX)_AMP|generatedPhotocurrentAmp|currentPhotocurrentAmp|resolveCathodePhotocurrentAmp" src/components/VirtualLab.vue src/components/IntegratedVirtualConsole.vue src/composables/useVirtualLabController.js src/features/experiment/helpers.js src/stores/experiment.js
```

结果：阶段 05 新组件没有旧 `tanh((voltage + cutoff)` 模型；命中项仅在 `src/stores/experiment.js` 原有主物理链路和控制器读取当前电流的位置。

```bash
git diff --check
```

结果：通过，无空白错误。

# 测试失败或未执行项及原因

- 未执行真实 AI 调用；当前环境未配置真实 AI Key，阶段 05 也不要求真实 AI 服务。
- Playwright 控制台记录到 3 条 `net::ERR_CONNECTION_REFUSED`，均来自可选知识图谱后端：
  - `http://localhost:3000/api/knowledge-graph/graph`
  - `http://localhost:3000/api/knowledge-graph/node/光电效应`
  - `http://localhost:3000/api/knowledge-graph/learning-path/光电效应`
- 这些请求失败符合阶段 01 的本地 fallback 设计，不是阶段 05 虚拟实验台运行时错误。

# 发现的问题与处理方式

- 默认 `4173` 端口已有旧预览监听。处理方式：`vis:check` 改用 `--port 4179`，`scene:report` 使用 `--port 4180`，Playwright 预览使用 `4181`。
- 第一次 Playwright 脚本尝试直接点击“提交本轮答题”，但按钮在未答题时按设计禁用。处理方式：改为填写完整 6 题后提交，并验证“已答 6/6”。
- `origin/ai-helper` 的试验仪按钮初始逻辑只会开启电源，不会关闭。处理方式：在 `useVirtualLabController.js` 中将 `powerOnInstrument()` 对接为 `toggleInstrumentPower()`，使新版按钮具备真实 toggle 行为。
- 复测相关字段需要落到主 store 才能被控制台、报告和评分复用。处理方式：只新增复测编号、签名和均值聚合，不修改真实光电流计算。

# 下一阶段注意事项

- 阶段 06 合入图数据库后端时，应保持后端为独立工程，不污染根 `package.json`，不要提交真实 `.env`。
- 继续保留当前 `animation-demo` 的物理模型和 3D/Spline 链路，不允许后端或 2D 页面反向替换主实验计算。
- 如果后续报告需要展示复测结果，应优先读取 `analysisSummary.fitRecords` 中的 `repeatCount`、`cutoffVoltageStdDev` 和 `repeatedRecordIds`，不要重新拟合另一套数据口径。
