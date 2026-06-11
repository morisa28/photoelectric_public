# 阶段目标

本阶段从 `origin/ai-helper` 合入新版数据处理、历史记录、统一图表、实验报告和 AI 教学洞察能力。

阶段目标是升级数据处理页面体验，但所有实验数据、拟合结果、截止电压结论和报告数据来源继续以 `animation-demo` 主实验 store 为准。

# 来源分支与参考内容

- 主要来源：`origin/ai-helper`
- 参考组件：`src/components/ExperimentReport.vue`、`src/components/HistoryRecords.vue`、`src/components/IntegratedPlotPanel.vue`、`src/components/AITeachingInsights.vue`
- 参考 feature：`src/features/experimentReport.js`
- 参考 service：`src/services/aiTeachingInsights.js`
- 参考测试：`src/features/__tests__/experimentReport.spec.js`、`src/services/__tests__/aiTeachingInsights.spec.js`

# 实际改动文件

- `src/components/AITeachingInsights.vue`
- `src/components/ExperimentReport.vue`
- `src/components/HistoryRecords.vue`
- `src/components/IntegratedPlotPanel.vue`
- `src/features/experimentReport.js`
- `src/features/__tests__/experimentReport.spec.js`
- `src/services/aiTeachingInsights.js`
- `src/services/__tests__/aiTeachingInsights.spec.js`
- `src/views/HomeView.vue`
- `document/阶段记录/分支整合/merge-03-新版数据处理与报告.md`

# 涉及的关键组件、store、service、feature

- `useExperimentStore`：主实验 store，是本阶段数据处理、历史记录、报告和教学洞察唯一读取的真实实验状态源。
- `records`：主实验记录数组，历史记录、I-U 图和报告原始记录都从这里读取。
- `analysisSummary`：主线拟合摘要，报告中的普朗克常量、相对误差、逸出功、截止频率和 `fitRecords` 都从这里读取。
- `ExperimentReport.vue`：新版实验报告页，提供 `reportReadyText`、`reportFitChart`、`reportDeviationChart`、`generateAIDraft` 和 HTML 报告导出。
- `HistoryRecords.vue`：新版历史记录页，支持当前实验与归档实验切换，并显示评分历史入口。
- `IntegratedPlotPanel.vue`：统一图表面板，继续从 `records` 与 `analysisSummary.fitRecords` 绘制 Uc-ν 和 I-U 图。
- `AITeachingInsights.vue`：数据处理页 AI 提问推荐区，使用 `aiTeachingInsights.js` 和 `sendToAI()`，真实 AI 不可用时保留本地 fallback。
- `experimentReport.js`：报告 HTML 与 SVG 图表生成辅助，不重新计算主物理结论。
- `aiTeachingInsights.js`：教学洞察、AI 问题、报告初稿和异常点定位辅助逻辑。

# 明确保留的 animation-demo 底层内容

本阶段未修改以下红线文件：

- `src/stores/experiment.js`
- `src/features/experiment/photoelectricModel.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`
- `src/stores/principleAnimation.js`
- `src/features/photoelectric-animation/*`
- `src/components/PrincipleAnimationSection.vue`
- `src/components/SplineScene.vue`
- `src/components/animation/ThreeDMicroAnimationPanel.vue`
- `resource/scene.splinecode`
- `vendor/spline-runtime/`

`src/views/HomeView.vue` 只做局部挂载：

```vue
<AITeachingInsights />
```

该组件仅插入数据处理页，不改变主页总导航结构，不改变原理动画入口，不改变 3D 实验台入口，也保留：

```vue
<AIAssistant v-if="currentSection !== 'principleAnimation'" />
```

# 本阶段没有合入的内容及原因

- 未修改 `src/stores/experiment.js` 添加 `completedExperiments` 或 `operationScoreHistory`；这些辅助状态属于阶段 04。本阶段在 `HistoryRecords.vue` 中对它们做空数组兼容，避免当前阶段页面崩溃。
- 未合入 `ai-helper` 中任何旧电流模型或会绕开 `photoelectricModel.js` 的计算逻辑。
- 未让报告组件重新拟合另一套普朗克常量；报告图表只消费 `analysisSummary.fitRecords`。
- 未启动或合入后端服务；知识图谱可选后端仍留到阶段 06。

# 验证命令与结果

```bash
npm run test:unit -- --run
```

结果：通过。17 个测试文件通过，179 个测试通过。

```bash
npm run build
```

结果：通过。Vite 仍提示大 chunk 警告，但构建成功。

```bash
npm run vis:check
```

结果：通过执行并生成视觉矩阵。`baselineStatus.changed` 出现 `home-desktop.png`、`home-mobile.png`、`3d-overview.png`、`3d-mobile.png`、`3d-zoom-close.png` 的 hash 差异；本阶段新增数据处理页内容且视觉脚本会重新捕获 3D 页面，因此记录为需后续人工确认的视觉基线变化，不作为构建失败。

辅助检查：

```bash
rg -n "tanh\(\(voltage \+ cutoff\)|PHOTOCURRENT_SATURATION_CURRENT_(MIN|MAX)_AMP|generatedPhotocurrentAmp" src/components/ExperimentReport.vue src/components/HistoryRecords.vue src/components/IntegratedPlotPanel.vue src/components/AITeachingInsights.vue src/features/experimentReport.js src/services/aiTeachingInsights.js
```

结果：无输出。本阶段迁移文件未引入 `ai-helper` 旧简化电流模型。

渲染验证：

```bash
npm run preview -- --host 127.0.0.1 --port 4173
node --input-type=module -e "<Playwright 数据处理页检查脚本>"
```

结果：通过。检查到：

- 页面标题为“光电效应实验模拟仿真平台”。
- 点击“数据处理”后 `AITeachingInsights` 可见。
- “实验报告”标签页可切换并处于 active。
- 点击“AI 生成初稿”后结论文本长度为 69，说明本地初稿生成正常。
- Playwright 截图保存到 `/tmp/stage03-data-processing.png`，未提交。

# 测试失败或未执行项及原因

- 第一次 Playwright 检查把 3 条 `net::ERR_CONNECTION_REFUSED` 计为失败；复查后确认它们全部来自可选知识图谱后端：
  - `http://localhost:3000/api/knowledge-graph/graph`
  - `http://localhost:3000/api/knowledge-graph/node/光电效应`
  - `http://localhost:3000/api/knowledge-graph/learning-path/光电效应`
- 这些请求失败符合阶段 01 的本地 fallback 设计，不是阶段 03 数据处理页运行时错误。
- 本阶段未进行真实 AI 服务调用；环境未配置真实 Key，且阶段目标要求保留 fallback。

# 发现的问题与处理方式

- `origin/ai-helper` 的 `HistoryRecords.vue` 直接从 store 解构 `completedExperiments` 和 `operationScoreHistory`。当前阶段尚未向主 store 添加这些字段，已改为通过 `storeToRefs(experimentStore)` 做可选读取并兼容空数组。
- 当前主线默认材料为 `agOK`，已在 `HistoryRecords.vue` 和 `aiTeachingInsights.js` 中补充中文材料名，避免页面显示裸 key。
- `HomeView.vue` 是红线文件，本阶段只做数据处理页局部插入，不替换导航结构。

# 下一阶段注意事项

- 阶段 04 可以在 `src/stores/experiment.js` 中按最小方式添加 `completedExperiments`、`operationScoreHistory`、题库状态和评分历史 action，但不得改变主物理计算 getter。
- `HistoryRecords.vue` 已具备兼容逻辑；阶段 04 补齐 store 后可自然显示归档实验和评分记录。
- 继续保留 `records`、`analysisSummary` 和 `analysisSummary.fitRecords` 作为报告与图表唯一可信数据源。
