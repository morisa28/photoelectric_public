# 阶段目标

本阶段从 `origin/ai-helper` 合入新版 AI 助手体验，补齐拖拽、导出聊天、下一步指导、数据诊断、Markdown 渲染和本地 fallback 能力。

本阶段只迁移 AI 助手和独立辅助服务，不改变实验主计算、主记录和 3D/原理动画状态链路。

# 来源分支与参考内容

- 主要来源：`origin/ai-helper`
- 参考组件：`src/components/AIAssistant.vue`
- 参考服务：`src/services/assistantChat.js`、`src/services/experimentCoach.js`
- 参考测试：`src/services/__tests__/assistantChat.spec.js`、`deepseekApi.spec.js`、`experimentCoach.spec.js`

# 实际改动文件

- `src/components/AIAssistant.vue`
- `src/services/deepseekApi.js`
- `src/services/assistantChat.js`
- `src/services/experimentCoach.js`
- `src/services/__tests__/assistantChat.spec.js`
- `src/services/__tests__/deepseekApi.spec.js`
- `src/services/__tests__/experimentCoach.spec.js`
- `document/阶段记录/分支整合/merge-02-新版AI助手.md`

# 涉及的关键组件、store、service、feature

- `AIAssistant.vue`：AI 助手浮窗和聊天面板，支持按钮拖拽、面板拖拽、导出聊天、清空对话、快捷问题、下一步指导和数据诊断。
- `deepseekApi.js`：真实 AI 调用入口，使用环境变量配置，并在未配置或调用失败时返回本地备用回答。
- `assistantChat.js`：聊天 Markdown 导出和安全渲染辅助。
- `experimentCoach.js`：根据主实验 store 的流程、诊断、记录和分析摘要生成下一步指导。
- `dataDiagnostic.js`：继续使用当前主线已有的数据诊断服务。

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
- `src/views/HomeView.vue`

`HomeView.vue` 仍保留：

```vue
<AIAssistant v-if="currentSection !== 'principleAnimation'" />
```

因此原理动画页不会被 AI 助手浮窗遮挡。

# 本阶段没有合入的内容及原因

- 未合入 `ai-helper` 中带硬编码 API Key 的旧 `deepseekApi.js` 写法；本阶段改为环境变量配置和本地 fallback，避免把密钥写入仓库。
- 未合入 `aiTeachingInsights.js` 和 `operationScoring.js`；它们分别属于阶段 03 和阶段 04。
- 未修改 `experimentStore`；AI 助手只读取 `records`、`analysisSummary`、`workflowSteps` 和 `diagnostics`。

# 验证命令与结果

```bash
npm run test:unit -- --run
```

结果：通过。15 个测试文件通过，168 个测试通过。

```bash
npm run build
```

结果：通过。Vite 仍提示大 chunk 警告，但构建成功。

辅助检查：

```bash
rg "sk-[A-Za-z0-9]|VITE_DEEPSEEK|AIAssistant v-if|currentSection !== 'principleAnimation'" src document/阶段记录/分支整合 document/实施指导
```

结果：

- 未发现硬编码 `sk-` Key。
- `deepseekApi.js` 只使用 `VITE_DEEPSEEK_API_KEY` 等环境变量。
- `HomeView.vue` 仍在原理动画页隐藏 AI 助手。

# 测试失败或未执行项及原因

- 本阶段自动化测试和构建未失败。
- 未调用真实 DeepSeek 服务；当前环境没有配置真实 Key，阶段目标要求支持 fallback，相关路径已由 `deepseekApi.js` 覆盖。

# 发现的问题与处理方式

- `ai-helper` 版本的 `deepseekApi.js` 含旧硬编码 Key，未直接照搬。
- 当前实现保留兼容导出 `sendMessageToDeepSeek()`，同时提供新版 `sendToAI()` 和 `getAIConnectionStatus()`。
- `assistantChat.js` 对 Markdown、表格、代码块和公式做转义渲染，避免把用户或模型文本直接作为未转义脚本执行。

# 下一阶段注意事项

- 阶段 03 的报告 AI 初稿可复用 `sendToAI()` 和本地 fallback，但不得把 Key 写入文档或 README。
- 后续数据处理和报告组件读取实验数据时，必须继续使用 `useExperimentStore()` 的 `records` 与 `analysisSummary`。
