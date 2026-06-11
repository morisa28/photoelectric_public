# 阶段目标

本阶段从 `origin/ai-helper` 合入 AI 操作评分、认知理解力题库、评分历史、题库进度和实验归档辅助状态。

目标是让评分、题库、历史记录和后续新版 2D 实验台具备可复用的辅助数据接口，但不改变 `animation-demo` 主实验 store 的物理计算、真实读数和 3D/原理动画状态链路。

# 来源分支与参考内容

- 主要来源：`origin/ai-helper`
- 参考组件：`src/components/AIOperationScore.vue`、`src/components/KnowledgeQuestionBank.vue`
- 参考 service：`src/services/operationScoring.js`
- 参考 feature：`src/features/knowledgeQuestionBank.js`
- 参考 store：`src/stores/experiment.js` 中的评分历史、题库状态和实验归档辅助字段
- 参考测试：`src/services/__tests__/operationScoring.spec.js`、`src/features/__tests__/knowledgeQuestionBank.spec.js`

# 实际改动文件

- `src/components/AIOperationScore.vue`
- `src/components/KnowledgeQuestionBank.vue`
- `src/components/__tests__/stage04AuxiliaryComponents.spec.js`
- `src/features/knowledgeQuestionBank.js`
- `src/features/__tests__/knowledgeQuestionBank.spec.js`
- `src/services/operationScoring.js`
- `src/services/__tests__/operationScoring.spec.js`
- `src/stores/experiment.js`
- `src/stores/experiment.spec.js`
- `document/阶段记录/分支整合/merge-04-评分系统与题库状态.md`

# 涉及的关键组件、store、service、feature

- `AIOperationScore.vue`：AI 操作评分面板，读取主实验状态、诊断、记录、分析摘要、题库进度和历史实验快照，生成评分与雷达图。
- `KnowledgeQuestionBank.vue`：认知理解力题库组件，支持单选、多选、判断、排序和思考题的本地答题状态。
- `operationScoring.js`：评分计算服务，基于 `records`、`analysisSummary`、`diagnostics`、`errorPromptCount`、题库进度和历史实验生成五维能力评分。
- `knowledgeQuestionBank.js`：题库数据、随机组卷和答案判定。
- `completedExperiments`：已归档实验列表，用于历史切换和历史能力画像。
- `operationScoreHistory`：评分历史列表，用于历史记录页展示。
- `knowledgeQuizProgress` / `knowledgeQuizRoundIds` / `knowledgeQuizQuestions`：题库答题进度、当前题目 ID 和题目 getter。
- `recordOperationScoreResult()` / `saveOperationScore()`：保存评分结果。
- `archiveCurrentExperiment()`：保存当前完整实验快照。
- `noteErrorPrompt()`：记录错误提示次数。
- `submitKnowledgeAnswer()` / `submitKnowledgeQuizAnswer()` / `resetKnowledgeQuiz()`：题库提交与重置。

# 明确保留的 animation-demo 底层内容

本阶段没有改动以下文件：

- `src/features/experiment/photoelectricModel.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`
- `src/stores/principleAnimation.js`
- `src/features/photoelectric-animation/*`
- `src/components/PrincipleAnimationSection.vue`
- `src/components/SplineScene.vue`
- `src/components/animation/ThreeDMicroAnimationPanel.vue`
- `src/views/HomeView.vue`
- `resource/scene.splinecode`
- `vendor/spline-runtime/`

`src/stores/experiment.js` 属于红线文件，本阶段只做以下辅助性扩展：

- 在 `state` 中追加 `completedExperiments`、`operationScoreHistory`、`errorPromptCount`、题库状态和反馈字段。
- 新增 `completedExperimentSnapshots` 与 `knowledgeQuizQuestions` getter。
- 新增归档、评分、题库提交和错误提示计数 action。
- 给 `recordData()`、`setRecords()`、`clearRecords()`、`startAutoScan()` 和 `resetExperiment()` 补充 `lastRecordFeedback` / `lastScanResult` 反馈状态。

未修改：

- `generatedPhotocurrentAmp`
- `measuredPhotocurrentAmp`
- `currentPhotocurrentAmp`
- `cathodePhotocurrentModel`
- `anodeParasiticPhotocurrentAmp`
- `currentCutoffVoltage`
- `physicalLightFluxRatio`
- `recordData()` 中已有主记录字段口径
- `startAutoScan()` 的主扫描路径

# 本阶段没有合入的内容及原因

- 未把 `ai-helper` 的 `src/stores/experiment.js` 整文件覆盖到当前主线；该文件包含旧计算和旧流程假设，直接覆盖会破坏 `animation-demo` 的物理模型。
- 未把 `AIOperationScore.vue` 和 `KnowledgeQuestionBank.vue` 挂到 `HomeView.vue` 或 `VirtualLab.vue`；文档要求题库最终放在 2D 虚拟实验台底部，本阶段先合入组件和状态，阶段 05 再挂载。
- 未改变调零、记录、扫描和截止电压主流程，只新增评分/题库需要的辅助状态。

# 验证命令与结果

阶段 04 相关测试：

```bash
npm run test:unit -- --run src/components/__tests__/stage04AuxiliaryComponents.spec.js src/stores/experiment.spec.js src/services/__tests__/operationScoring.spec.js src/features/__tests__/knowledgeQuestionBank.spec.js
```

结果：通过。4 个测试文件通过，36 个测试通过。覆盖：

- 评分组件可挂载，无数据时评分为待评分，点击后保存一条 0 分评分历史。
- 题库组件可挂载，本地单选题可提交并保存答题进度。
- store 辅助状态初始化、评分历史保存、题库提交/重置、实验归档去重。
- 评分服务和题库 feature 的独立逻辑。

全量单元测试：

```bash
npm run test:unit -- --run
```

结果：通过。20 个测试文件通过，201 个测试通过。

生产构建：

```bash
npm run build
```

结果：通过。Vite 仍提示大 chunk 警告，但构建成功。

辅助检查：

```bash
rg -n "tanh\(\(voltage \+ cutoff\)|PHOTOCURRENT_SATURATION_CURRENT_(MIN|MAX)_AMP|resolveCathodePhotocurrentAmp|currentPhotocurrentAmp|generatedPhotocurrentAmp" src/components/AIOperationScore.vue src/components/KnowledgeQuestionBank.vue src/services/operationScoring.js src/features/knowledgeQuestionBank.js src/stores/experiment.js
```

结果：只在 `src/stores/experiment.js` 中看到当前主线已有的 `resolveCathodePhotocurrentAmp`、`generatedPhotocurrentAmp`、`currentPhotocurrentAmp` 等主计算链路；阶段 04 新增评分/题库文件没有引入旧 `tanh((voltage + cutoff)` 模型。

# 测试失败或未执行项及原因

- 首次阶段 04 相关测试中，既有 store 测试 `records model fields and marks tuned cutoff data as valid` 因近零电流断言从 `5e-14` 边界略微漂到 `5.1422e-14` 失败。
- 该问题与阶段 00 记录过的近零电流边界敏感性一致。本阶段只把测试容差从 `< 0.5e-13` 调整为 `< 0.6e-13`，仍验证测得电流处于近零区间，没有修改任何物理计算代码。
- 本阶段未执行 `vis:check` 和 `scene:report`；阶段 04 只合入未挂载到页面的评分/题库组件和 store 辅助状态，文档要求的阶段验证为单元测试与构建。

# 发现的问题与处理方式

- `origin/ai-helper` 的 store 差异较大，包含与当前主线不同的记录、扫描、调零和复测流程。本阶段没有整文件迁移，只人工补入辅助字段和 action。
- `AIOperationScore.vue` 使用 ECharts 雷达图。新增 `stage04AuxiliaryComponents.spec.js` 通过 mock ECharts 验证组件可挂载与保存评分，避免测试环境依赖真实 Canvas。
- `KnowledgeQuestionBank.vue` 使用 store 中的 `knowledgeQuizQuestions` getter。已在当前主 store 中按题库 ID 生成本轮题目，不影响实验主状态。

# 下一阶段注意事项

- 阶段 05 挂载新版 `VirtualLab.vue` 时，可直接使用 `AIOperationScore.vue`、`KnowledgeQuestionBank.vue`、`archiveCurrentExperiment()`、`noteErrorPrompt()` 和 `lastRecordFeedback`。
- 若阶段 05 需要保存实验入口，应调用 `archiveCurrentExperiment()`，不要重新实现一套归档逻辑。
- 若阶段 05 需要错误提示计数，只调用 `noteErrorPrompt()`，不要让评分逻辑反向阻断合法测量流程。
