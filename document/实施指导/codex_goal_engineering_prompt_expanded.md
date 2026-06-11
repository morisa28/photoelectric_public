/goal
你是 `my_photoelectric` 项目的工程化整合执行者。请完整读取本文件、`AGENTS.md`、`README.md`、`package.json` 以及当前 Git 分支状态，然后在不中断、不反复询问用户确认的前提下，按本文档完成 `2d-logic`、`ai-helper` 到 `animation-demo` 的工程化功能整合。整合完成后必须收拢到 `main` 分支；如果当前环境具备远端写权限，还要推送到远端，最终结果应当是远端main分支为最完整分支。整套任务完成后再交付用户验收。

> 本文件适配 Codex `/goal` 模式：请把它当作完整目标任务执行，而不是把它拆成需要用户逐步确认的问答流程。  
> 允许并鼓励使用项目中的真实文件名、组件名、函数名、store 字段名、测试脚本名和 Git 命令。不要为了“可读性”隐藏关键技术细节；但所有阶段文档仍应解释这些名字对应的中文含义。

---

# 0. 执行总目标

把三个开发线整合成一个稳定、完整、可继续开发的主线版本：

```text
main
└── 最终应包含：
    ├── animation-demo 的底层物理框架、原理动画、3D 实验、真实实验状态和计算体系
    ├── 2d-logic 的新版知识图谱页面体验和可选图数据库后端能力
    ├── ai-helper 的新版 2D 虚拟实验台页面
    ├── ai-helper 的新版数据处理、图表、实验报告与 AI 初稿能力
    ├── ai-helper 的新版 AI 助手、下一步指导、聊天导出和 fallback 能力
    ├── ai-helper 的评分系统、题库、评分历史和实验归档能力
    └── 完整阶段文档、最终 README、验证记录和 main 收拢说明
```

本任务不是“把两个分支 merge 进来”。本任务是 **以 `animation-demo` 为底层主线，按模块吸收 `2d-logic` 与 `ai-helper` 的新版页面和辅助能力**。

---

# 1. 总原则与优先级

## 1.1 分支职责

### `animation-demo` 的职责

`animation-demo` 是底层主线，必须保留它的：

```text
- 底层物理框架
- 主实验状态 store
- 电流计算体系
- 截止电压计算体系
- 光强与光路计算体系
- 阳极污染、阳极寄生电流、截止尾流逻辑
- 原理动画状态链路
- 伏安轨迹、微观动画引擎、动画映射逻辑
- 3D 实验台状态同步
- 3D 微观小窗状态读取链路
- Spline / Three / resource 场景链路
- 主页总导航中的知识图谱、2D 实验台、原理动画、数据处理、3D入口
```

尤其不要让 `ai-helper` 的旧简化电流模型覆盖 `animation-demo` 中的：

```text
src/stores/experiment.js
src/features/experiment/photoelectricModel.js
src/features/experiment/constants.js
src/features/experiment/helpers.js
src/features/photoelectric-animation/*
src/stores/principleAnimation.js
src/components/PrincipleAnimationSection.vue
src/components/SplineScene.vue
src/components/animation/ThreeDMicroAnimationPanel.vue
```

### `2d-logic` 的职责

`2d-logic` 主要提供新版知识图谱：

```text
- PhotoelectricEffectGraph.vue 新版 UI
- 分类筛选
- 节点搜索
- 学习路径
- 路径高亮
- 学习建议卡片
- 主题切换
- 刷新与清除高亮
- 科学家图片
- 前端知识图谱服务
- 可选的 Express + Neo4j 后端
```

知识图谱页面以 `2d-logic` 为主体，但必须：

```text
- 不覆盖 HomeView.vue 的主导航结构
- 不影响原理动画入口
- 不影响 3D 入口
- 不影响 2D 实验台和数据处理页
- 后端不可用时仍然使用本地图谱数据 fallback
```

### `ai-helper` 的职责

`ai-helper` 主要提供新版教学与辅助模块：

```text
- 新版 2D 虚拟实验台页面
- IntegratedVirtualConsole
- KnowledgeQuestionBank
- AIAssistant 新版交互
- assistantChat
- experimentCoach
- deepseekApi 配置化结构和 fallback
- AIOperationScore
- AITeachingInsights
- operationScoring
- experimentReport
- HistoryRecords 新版历史实验切换
- ExperimentReport 新版报告与 AI 初稿
- 评分历史
- 题库状态
- 实验归档
- 错误操作计数
```

这些页面和辅助模块可以参考 `ai-helper`，但凡涉及实验主数据、主读数、主计算、主记录口径，必须接入 `animation-demo` 的主实验状态和计算体系。

---

# 2. 绝对红线

以下内容不得被 `2d-logic` 或 `ai-helper` 覆盖、删除、替换或降级：

```text
1. src/stores/experiment.js 中 animation-demo 的主物理计算体系
2. src/features/experiment/photoelectricModel.js
3. 非包围式光电管模型
4. resolveCathodePhotocurrentAmp / resolveAnodeParasiticPhotocurrentAmp / resolveNormalizedLightFlux 等主计算链路
5. anodeParasiticPhotocurrentAmp / measuredPhotocurrentAmp / cathodePhotocurrentModel 等主读数合成逻辑
6. cutoffModeCathodeTail* 相关截止尾流逻辑
7. currentCutoffVoltage 及其相关截止电压口径
8. physicalLightFluxRatio / aperture / distance 等光强换算口径
9. src/stores/principleAnimation.js 原理动画独立 store
10. src/components/PrincipleAnimationSection.vue 原理动画入口与状态隔离
11. src/features/photoelectric-animation/* 动画引擎、映射、伏安轨迹逻辑
12. src/components/SplineScene.vue 3D 场景桥接
13. src/components/animation/ThreeDMicroAnimationPanel.vue 3D 微观小窗
14. resource/scene.splinecode
15. vendor/spline-runtime/
16. HomeView.vue 中的原理动画入口、3D 实验台入口和主页总导航结构
```

若 `ai-helper` 中出现以下旧逻辑，只能参考，不得作为主逻辑合入：

```text
- 使用 tanh((voltage + cutoff) / 0.45) 的旧简化电流曲线
- 旧版 generatedPhotocurrentAmp / currentPhotocurrentAmp 主读数合成
- 旧版 PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP / PHOTOCURRENT_SATURATION_CURRENT_MIN_AMP 作为主模型的写法
- 任何会绕开 photoelectricModel.js 的主电流计算
```

允许保留的独立辅助计算：

```text
- operationScoring.js 的评分计算
- knowledgeQuestionBank.js 的题库判断
- assistantChat.js 的聊天导出与渲染辅助
- experimentCoach.js 的下一步指导建议
- aiTeachingInsights.js 的教学洞察
- experimentReport.js 的报告 HTML/SVG 组织与展示
```

这些辅助逻辑不能反向影响主实验数据、3D 状态、原理动画状态和真实读数。

---

# 3. 执行方式要求

## 3.1 不要中途询问

不要中途向用户询问是否继续。
不要阶段完成后等待用户确认。
不要只完成一个阶段就停止。
不要把本文档改写成待办列表后停下。

遇到不明确处，按以下规则自行决定：

```text
底层物理、原理动画、3D、实验状态、实验计算：animation-demo 优先。
知识图谱页面：2d-logic 优先。
2D 页面、数据处理、AI 助手、评分、题库、报告：ai-helper 优先。
无法判断是否影响底层计算时：保留 animation-demo 计算接口，只移植 UI 或辅助状态。
```

只有遇到不可恢复阻塞才停止，例如：

```text
- 缺失远端关键分支
- Git 工作区存在无法安全保存的用户未提交改动
- 当前环境无法写文件或无法提交 Git
- 合并到 main 或 push 需要权限但当前环境没有认证
```

即便如此，也必须写清楚阻塞位置、已完成内容、未完成内容、可恢复步骤。

## 3.2 禁止整支合并

禁止执行：

```bash
git merge origin/2d-logic
git merge origin/ai-helper
git pull origin 2d-logic
git pull origin ai-helper
```

允许使用：

```bash
git show origin/2d-logic:path/to/file
git show origin/ai-helper:path/to/file
git diff origin/animation-demo..origin/2d-logic -- path/to/file
git diff origin/animation-demo..origin/ai-helper -- path/to/file
git checkout origin/ai-helper -- path/to/file   # 仅限非红线文件，且必须之后人工适配
```

对于红线文件，例如 `src/stores/experiment.js`、`src/features/experiment/helpers.js`、`src/features/experiment/constants.js`，不要整文件 checkout。只能逐段移植辅助字段、辅助 action、辅助测试。

## 3.3 每阶段必须提交 Git

每个阶段完成后必须：

```text
1. 编写阶段文档
2. 执行本阶段要求的验证
3. git status 检查范围
4. git diff 检查是否误改红线文件
5. git add 只添加本阶段相关文件
6. git commit 提交本阶段成果
```

不要多个阶段混在一个 commit。
不要把 `.vite/`、`outputs/`、`report_assets/`、临时截图、构建产物、旧阶段无关文档混入 commit。
不要把无法验证的测试写成已通过。

## 3.4 阶段文档要求

阶段文档统一放在：

```text
document/阶段记录/分支整合/
```

文件命名：

```text
merge-00-基线与红线保护.md
merge-01-新版知识图谱前端.md
merge-02-新版AI助手.md
merge-03-新版数据处理与报告.md
merge-04-评分系统与题库状态.md
merge-05-新版2D虚拟实验台.md
merge-06-图数据库后端.md
merge-07-底层整合校验.md
merge-08-文档与收口.md
merge-final-main-收拢说明.md
```

每份阶段文档必须包含：

```text
# 阶段目标
# 来源分支与参考内容
# 实际改动文件
# 涉及的关键组件、store、service、feature
# 明确保留的 animation-demo 底层内容
# 本阶段没有合入的内容及原因
# 验证命令与结果
# 测试失败或未执行项及原因
# 发现的问题与处理方式
# 下一阶段注意事项
```

允许使用真实函数名、变量名、组件名和文件路径。例如：

```text
useExperimentStore
currentPhotocurrentAmp
measuredPhotocurrentAmp
PhotoelectricEffectGraph.vue
src/features/experiment/photoelectricModel.js
```

但必须解释其中文含义，例如“主实验 store”“实际读数合成”“知识图谱页面组件”。

---

# 4. 准备阶段：远端同步与整合分支

## 4.1 检查工作区

先执行：

```bash
git status --short
git branch
git branch -r
git remote -v
```

如果工作区有未提交改动：

```bash
mkdir -p artifacts/integration
git diff > artifacts/integration/preexisting-worktree.patch
git diff --staged > artifacts/integration/preexisting-index.patch
```

然后判断：

```text
- 如果是本次任务相关改动，可以纳入当前阶段，但必须在阶段 00 文档说明。
- 如果不是本次任务相关改动，先安全保存，不要覆盖，不要删除。
- 如果无法判断，不要继续破坏工作区；写明阻塞。
```

## 4.2 确保能抓所有远端分支

检查远端抓取规则：

```bash
git config --get-all remote.origin.fetch
```

如果只抓取单个分支，改为抓取所有远端分支：

```bash
git config remote.origin.fetch "+refs/heads/*:refs/remotes/origin/*"
```

同步远端引用：

```bash
git fetch origin --prune --tags
```

确认分支存在：

```bash
git rev-parse --short origin/main
git rev-parse --short origin/animation-demo
git rev-parse --short origin/2d-logic
git rev-parse --short origin/ai-helper
```

如果缺失任何关键分支，停止并写明：

```text
缺失的分支名
已看到的远端分支
建议用户确认远端仓库是否同步
```

## 4.3 创建整合分支

从远端 `animation-demo` 创建整合分支：

```bash
git switch -c integration/full-branch-merge origin/animation-demo
```

如果分支已存在：

```bash
git switch integration/full-branch-merge
git status --short
```

不要强制覆盖已有整合分支。如果已有分支包含改动，先记录当前提交号和状态，再继续。

记录提交号：

```bash
git rev-parse --short HEAD
git rev-parse --short origin/main
git rev-parse --short origin/animation-demo
git rev-parse --short origin/2d-logic
git rev-parse --short origin/ai-helper
```

---

# 5. 阶段 00：基线与红线保护

## 目标

锁定 `animation-demo` 当前状态，建立整合红线，确认基线可构建、可测试到什么程度。

## 操作

检查项目脚本：

```bash
cat package.json
```

运行基线验证：

```bash
npm run test:unit -- --run
npm run build
npm run scene:report
```

如果 `node_modules` 缺失，先执行：

```bash
npm install
```

如果存在平台原生依赖缺失，按错误提示修复，但必须记录到阶段文档。

## 需要检查的红线文件

记录这些文件的基线状态：

```bash
git ls-tree -r --name-only HEAD src/stores src/features src/components src/views | sed -n '1,200p'
```

重点查看：

```text
src/stores/experiment.js
src/stores/principleAnimation.js
src/features/experiment/photoelectricModel.js
src/features/experiment/constants.js
src/features/experiment/helpers.js
src/features/photoelectric-animation/engine.js
src/features/photoelectric-animation/mapping.js
src/features/photoelectric-animation/voltAmpTrajectory.js
src/components/PrincipleAnimationSection.vue
src/components/SplineScene.vue
src/components/animation/ThreeDMicroAnimationPanel.vue
src/views/HomeView.vue
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-00-基线与红线保护.md
```

必须记录：

```text
origin/main 短哈希
origin/animation-demo 短哈希
origin/2d-logic 短哈希
origin/ai-helper 短哈希
当前整合分支名
基线测试结果
已知基线问题
明确红线
```

## 提交

```bash
git add document/阶段记录/分支整合/merge-00-基线与红线保护.md
git commit -m "merge: 阶段00 基线与红线保护"
```

---

# 6. 阶段 01：新版知识图谱前端

## 来源

主要参考：

```text
origin/2d-logic
```

## 目标

把知识图谱页面升级为 `2d-logic` 新版体验，并保留无后端本地 fallback。

## 参考文件

优先检查：

```text
src/components/PhotoelectricEffectGraph.vue
src/services/neo4jService.js
src/data/knowledgeGraphData.js
src/assets/scientists/*
```

参考命令：

```bash
git diff origin/animation-demo..origin/2d-logic -- src/components/PhotoelectricEffectGraph.vue
git show origin/2d-logic:src/components/PhotoelectricEffectGraph.vue > /tmp/PhotoelectricEffectGraph.2d-logic.vue
git show origin/2d-logic:src/services/neo4jService.js > /tmp/neo4jService.2d-logic.js || true
```

## 合入能力

必须实现或迁移：

```text
- loading 状态
- error 状态
- retryLoad
- filter panel / category filter
- activeCategories
- visibleNodeCount / totalNodeCount
- searchKeyword
- searchMode: node / path
- searchResults
- showResults
- performSearch
- clearSearch
- selectSearchResult
- learningPath / currentPath
- path highlight
- learningAdvice
- currentTheme
- toggleTheme
- refreshData
- clearHighlight
- resetView
- scientistImages
- detail panel photo section
```

可以使用真实变量名和函数名，但要保证实现适配当前主线。

## 服务层要求

不要直接保留硬编码：

```text
http://localhost:3000/api/knowledge-graph
```

实现或调整为：

```text
VITE_KNOWLEDGE_GRAPH_API_BASE
默认 http://localhost:3000/api/knowledge-graph
请求失败 fallback 到本地 knowledgeGraphData
```

建议新增或重构为：

```text
src/services/knowledgeGraphService.js
```

该服务应提供：

```text
fetchGraphData()
fetchNodeDetails(nodeId)
fetchLearningPath(nodeId)
searchNodes(keyword)
```

其中：

```text
- 后端可用时用后端
- 后端不可用时用本地数据
- 后端路径不可用时给出温和提示
```

## 数据 fallback 要求

保留：

```text
src/data/knowledgeGraphData.js
```

如果 `2d-logic` 有新增节点、详情、科学家图片字段、路径字段，人工合并进本地数据结构。
不要删除本地数据。
不要让后端缺失时知识图谱空白。

## 样式要求

`PhotoelectricEffectGraph.vue` 可以采用 `2d-logic` 新版样式，但必须限制作用范围：

```text
- 优先使用 scoped style
- 避免修改全局 body/html
- 不影响 HomeView 顶部导航
- 不影响 VirtualLab、PrincipleAnimationSection、ThreeDView
```

## 禁止

不要合入：

```text
2d-logic 的 HomeView.vue
2d-logic 的旧 3D 资源删除
2d-logic 的旧 Three 结构
backend/*（本阶段先不合）
root package.json 中的后端依赖
.env（本阶段先不处理）
```

## 验证

```bash
npm run test:unit -- --run
npm run build
npm run vis:check
```

若 `vis:check` 因本地浏览器环境失败，记录原因并至少完成构建和人工页面检查。

人工验证：

```text
不启动后端，打开知识图谱页面，图谱显示。
节点点击后详情面板显示。
分类筛选生效。
本地搜索生效。
学习路径无后端时不报错，并提示动态路径暂不可用。
主题切换不影响其他页面。
主页仍保留原理动画和 3D 入口。
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-01-新版知识图谱前端.md
```

## 提交

```bash
git status --short
git diff -- src/components/PhotoelectricEffectGraph.vue src/services src/data src/assets document/阶段记录/分支整合/merge-01-新版知识图谱前端.md
git add src/components/PhotoelectricEffectGraph.vue src/services src/data src/assets document/阶段记录/分支整合/merge-01-新版知识图谱前端.md
git commit -m "merge: 阶段01 新版知识图谱前端"
```

如果 `src/assets` 中只新增部分科学家图片，只添加实际新增文件，不要粗暴添加无关资源。

---

# 7. 阶段 02：新版 AI 助手

## 来源

主要参考：

```text
origin/ai-helper
```

## 目标

合入新版 AI 助手体验，不改变实验主计算。

## 参考文件

```text
src/components/AIAssistant.vue
src/services/deepseekApi.js
src/services/assistantChat.js
src/services/experimentCoach.js
src/services/dataDiagnostic.js
src/services/__tests__/assistantChat.spec.js
src/services/__tests__/deepseekApi.spec.js
src/services/__tests__/experimentCoach.spec.js
```

参考命令：

```bash
git diff origin/animation-demo..origin/ai-helper -- src/components/AIAssistant.vue src/services
git show origin/ai-helper:src/components/AIAssistant.vue > /tmp/AIAssistant.ai-helper.vue
git show origin/ai-helper:src/services/deepseekApi.js > /tmp/deepseekApi.ai-helper.js
```

## 合入能力

必须迁移：

```text
- AI助手新版界面
- assistantRoot
- header drag / panel drag
- exportConversation
- clearConversation
- requestCoachGuidance
- requestDataDiagnostic
- sendToAPI
- getAIConnectionStatus
- sendToAI
- fallback response
- buildChatMarkdown
- renderAssistantMarkdown
- buildCoachGuidance
- buildCoachPrompt
- formatCoachGuidance
```

## AI Key 策略

实现：

```text
优先使用 import.meta.env.VITE_DEEPSEEK_API_KEY
如果没有配置，保留当前硬编码 Key 作为默认值
如果真实 AI 调用失败，使用本地 fallback
```

不要把硬编码 Key 写入阶段文档。
不要把 Key 发给 AI。
不要在 README 中公开具体 Key 内容。

可以保留兼容导出：

```js
export async function sendMessageToDeepSeek(messages, systemPrompt = null) {
  return sendToAI(messages, systemPrompt)
}
```

或等效实现，保证旧调用不坏。

## 原理动画页规则

`HomeView.vue` 中仍应保留：

```vue
<AIAssistant v-if="currentSection !== 'principleAnimation'" />
```

如果改为其他写法，也必须保证原理动画页不被 AI 助手遮挡。

## 数据读取边界

AI助手允许读取：

```text
records
analysisSummary
workflowSteps
diagnostics
apparatus 状态摘要
params 状态摘要
```

不得读取或发送：

```text
API Key
.env
Neo4j 密码
后端连接密码
本地绝对路径
```

## 验证

```bash
npm run test:unit -- --run
npm run build
```

人工验证：

```text
AI助手在普通页面显示。
原理动画页不显示或不遮挡。
拖拽正常。
输入问题正常。
快捷按钮正常。
下一步指导能读当前实验流程。
导出聊天正常。
清空对话正常。
无配置 Key 时走硬编码默认 Key 或 fallback。
真实 AI 失败时 fallback 正常。
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-02-新版AI助手.md
```

## 提交

```bash
git status --short
git add src/components/AIAssistant.vue src/services document/阶段记录/分支整合/merge-02-新版AI助手.md
git commit -m "merge: 阶段02 新版AI助手"
```

---

# 8. 阶段 03：新版数据处理与实验报告

## 来源

主要参考：

```text
origin/ai-helper
```

## 目标

合入新版数据处理、历史记录、图表展示、报告生成和 AI 报告初稿。所有数据、拟合结果和实验结论必须来自 `animation-demo` 主实验状态和分析摘要。

## 参考文件

```text
src/components/ExperimentReport.vue
src/components/HistoryRecords.vue
src/components/IntegratedPlotPanel.vue
src/components/AITeachingInsights.vue
src/features/experimentReport.js
src/services/aiTeachingInsights.js
src/features/__tests__/experimentReport.spec.js
src/services/__tests__/aiTeachingInsights.spec.js
```

参考命令：

```bash
git diff origin/animation-demo..origin/ai-helper -- src/components/ExperimentReport.vue src/components/HistoryRecords.vue src/components/IntegratedPlotPanel.vue src/features/experimentReport.js src/services/aiTeachingInsights.js
```

## 合入能力

```text
- 实验报告新版 UI
- reportReadyText
- reportFitChart
- reportDeviationChart
- generateAIDraft
- buildExperimentReportHtml
- buildReportChartSvg
- escapeReportHtml
- formatReportNumber
- 历史实验切换
- completedExperiments 展示
- operationScoreHistory 展示
- score history 列表
- AI教学洞察
- 数据处理页插入 AITeachingInsights
```

## 图表规则

允许：

```text
- 使用 ai-helper 的图表样式
- 使用 ai-helper 的 SVG 报告图
- 使用 ai-helper 的复测离散度图
- 使用 ai-helper 的历史实验切换界面
```

禁止：

```text
- 图表组件自己重新计算主物理结论
- 报告组件自己重新拟合出另一套普朗克常量
- 使用 ai-helper 的旧电流模型生成报告数据
```

正确方式：

```text
records 来自 useExperimentStore()
analysisSummary 来自 useExperimentStore()
fitRecords 来自 analysisSummary.fitRecords
cutoffVoltage、theoreticalCutoffVoltage、cutoffVoltageDeviation 使用主线记录字段
```

## Store 辅助字段

如果本阶段需要 `completedExperiments`、`operationScoreHistory` 等字段，但还未在阶段 04 添加，可先做兼容空数组处理，不强行改主 store。也可以在本阶段添加最小辅助字段，但不得触碰主计算。

## 验证

```bash
npm run test:unit -- --run
npm run build
npm run vis:check
```

人工验证：

```text
数据处理页能打开。
无数据时不崩溃。
当前实验记录可显示。
历史实验区域无数据时有友好提示。
报告页面能打开。
AI初稿按钮可生成文本。
报告导出不报错。
图表显示的数据与 records / analysisSummary 一致。
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-03-新版数据处理与报告.md
```

## 提交

```bash
git status --short
git add src/components/ExperimentReport.vue src/components/HistoryRecords.vue src/components/IntegratedPlotPanel.vue src/components/AITeachingInsights.vue src/features/experimentReport.js src/services/aiTeachingInsights.js src/features/__tests__ src/services/__tests__ document/阶段记录/分支整合/merge-03-新版数据处理与报告.md
git commit -m "merge: 阶段03 新版数据处理与报告"
```

只添加本阶段实际存在且修改过的文件。

---

# 9. 阶段 04：评分系统与题库状态

## 来源

主要参考：

```text
origin/ai-helper
```

## 目标

合入评分系统、题库系统、评分历史、题库状态和相关测试，不改变实验主计算。

## 参考文件

```text
src/components/AIOperationScore.vue
src/components/KnowledgeQuestionBank.vue
src/services/operationScoring.js
src/features/knowledgeQuestionBank.js
src/services/__tests__/operationScoring.spec.js
src/features/__tests__/knowledgeQuestionBank.spec.js
src/stores/experiment.js
src/stores/__tests__/experiment.spec.js
```

参考命令：

```bash
git diff origin/animation-demo..origin/ai-helper -- src/components/AIOperationScore.vue src/components/KnowledgeQuestionBank.vue src/services/operationScoring.js src/features/knowledgeQuestionBank.js src/stores/experiment.js
```

## 合入能力

```text
- AIOperationScore
- KnowledgeQuestionBank
- operationScoring
- buildOperationScore
- buildOperationScorePrompt
- formatOperationScoreText
- knowledge question bank
- quiz round
- answer grading
- operationScoreHistory
- completedExperiments
- completedExperimentSnapshots
- errorPromptCount
- knowledgeQuestionBank
- knowledgeQuizProgress
- knowledgeQuizRoundIds
- knowledgeQuizQuestions
- saveOperationScore
- resetKnowledgeQuiz
- submitQuiz / submitKnowledgeQuizAnswer
- archiveCurrentExperiment
- noteErrorPrompt
```

## Store 修改要求

可以在 `src/stores/experiment.js` 增加辅助状态：

```text
completedExperiments
operationScoreHistory
errorPromptCount
knowledgeQuestionBank
knowledgeQuizProgress
knowledgeQuizRoundIds
lastRecordFeedback
lastScanResult
```

可以增加辅助 getter/action：

```text
completedExperimentSnapshots
knowledgeQuizQuestions
archiveCurrentExperiment()
noteErrorPrompt()
saveOperationScore()
resetKnowledgeQuiz()
submitKnowledgeQuizAnswer()
```

不得覆盖或降级：

```text
generatedPhotocurrentAmp
measuredPhotocurrentAmp
currentPhotocurrentAmp
cathodePhotocurrentModel
anodeParasiticPhotocurrentAmp
currentCutoffVoltage
physicalLightFluxRatio
recordData 主记录字段口径
startAutoScan 主扫描口径
```

如果需要修改 `recordData` 以支持复测字段，只添加字段，不替换主计算：

```text
repeatMeasurementIndex
measurementSignature
cutoffVoltageDeviation
theoreticalCutoffVoltage
appliedVoltage
measurementMode
```

若字段已存在，以 `animation-demo` 为准，只做兼容。

## 题库位置

组件 `KnowledgeQuestionBank.vue` 最终放在 2D 虚拟实验台底部，本阶段可先合入组件和状态，阶段 05 再挂载到 2D 页面。

## 验证

```bash
npm run test:unit -- --run
npm run build
```

人工验证：

```text
评分组件能挂载。
无数据时评分显示待评分。
有记录时评分能生成。
雷达图能显示。
题库组件能读取题目。
答题能提交。
题库进度能保存。
评分历史能保存。
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-04-评分系统与题库状态.md
```

## 提交

```bash
git status --short
git add src/components/AIOperationScore.vue src/components/KnowledgeQuestionBank.vue src/services/operationScoring.js src/features/knowledgeQuestionBank.js src/stores/experiment.js src/stores/__tests__ src/services/__tests__ src/features/__tests__ document/阶段记录/分支整合/merge-04-评分系统与题库状态.md
git commit -m "merge: 阶段04 评分系统与题库状态"
```

---

# 10. 阶段 05：新版 2D 虚拟实验台页面

## 来源

主要参考：

```text
origin/ai-helper
```

## 目标

升级 2D 实验台页面为 `ai-helper` 新版体验，同时所有交互接入 `animation-demo` 主实验状态和计算体系。

## 参考文件

```text
src/components/VirtualLab.vue
src/components/IntegratedVirtualConsole.vue
src/composables/useVirtualLabController.js
src/components/__tests__/virtualLabLayout.spec.js
src/components/KnowledgeQuestionBank.vue
```

参考命令：

```bash
git diff origin/animation-demo..origin/ai-helper -- src/components/VirtualLab.vue src/components/IntegratedVirtualConsole.vue src/composables/useVirtualLabController.js src/components/__tests__/virtualLabLayout.spec.js
```

## 合入能力

```text
- ai-helper 新版设备面板
- device status strip
- instrument power toggle
- photocell carriage
- lamp module
- distance handle
- material select
- filter buttons
- aperture buttons
- workflow section
- IntegratedVirtualConsole
- tester panel
- KnowledgeQuestionBank 放在实验台底部
- powerOnInstrument
- handleRecordExperiment
- goToHistory -> dataProcessing
- noteErrorPrompt
- 复测提示和保存实验入口
```

## 接入主线状态

必须继续使用：

```text
useVirtualLabController(emit)
useExperimentStore()
storeToRefs(experimentStore)
```

按钮和控件必须接入主线 store：

```text
currentMaterial -> experimentStore.updateParams({ material })
currentFilter -> experimentStore.setFilterByWavelength()
currentAperture -> experimentStore.setAperture()
currentDistance -> experimentStore.apparatus.photocellDistance
sliderPosition -> distanceToSliderPosition()
startDrag -> experimentStore.setPhotocellDistance()
isPhotocellCoverOpen -> experimentStore.setPhotocellCoverOpen()
isLampCoverOpen -> experimentStore.setLampCoverOpen()
isLampPowerOn -> experimentStore.updateApparatus({ lampPowerOn })
isInstrumentPowered -> experimentStore.apparatus.instrumentPowerOn
powerOnInstrument -> experimentStore.setInstrumentPower(true)
handleRecordData -> experimentStore.recordData()
handleAutoScan -> experimentStore.startAutoScan()
handleStopScan -> experimentStore.stopAutoScan()
handleRecordExperiment -> experimentStore.archiveCurrentExperiment()
```

若 `setInstrumentPower` 或 `archiveCurrentExperiment` 在主线不存在，应按阶段 04 的辅助 action 方式补齐，不得改主计算。

## 调零规则

不强制改变主线测量通过条件。
如果迁移 `zeroAdjustTouched` 相关逻辑，只用于评分和提示，不用于阻断已有合法测量流程。

## 复测规则

实现：

```text
同条件再次记录时，不一刀切拒绝。
如果用户意图复测，允许保存，并计入 repeatMeasurementIndex / repeatCount / cutoffVoltageStdDev。
```

如果当前没有复测弹窗，可以先实现“允许重复并标记复测”，阶段文档说明后续可增加确认弹窗。

## 禁止

不得把 `ai-helper` 旧电流计算带入：

```text
generatedPhotocurrentAmp 旧 tanh 模型
currentPhotocurrentAmp 旧简化合成
PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP 作为主读数模型
```

不得覆盖：

```text
src/features/experiment/photoelectricModel.js
src/stores/principleAnimation.js
src/components/PrincipleAnimationSection.vue
src/components/SplineScene.vue
```

## 验证

```bash
npm run test:unit -- --run
npm run build
npm run vis:check
npm run scene:report
```

人工验证完整流程：

```text
首页进入虚拟实验台。
开启试验仪。
打开汞灯电源。
打开汞灯盖和光电管盖。
调零。
接线。
切换滤光片。
调整光阑。
拖动光电管距离。
调节电压。
记录数据。
同条件复测。
保存本次实验。
完成题库。
进入数据处理页查看历史实验、图表、评分、报告。
进入 3D 页面检查读数、模式、小窗是否同步。
进入原理动画页检查 AI 不遮挡。
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-05-新版2D虚拟实验台.md
```

## 提交

```bash
git status --short
git add src/components/VirtualLab.vue src/components/IntegratedVirtualConsole.vue src/components/KnowledgeQuestionBank.vue src/composables/useVirtualLabController.js src/components/__tests__/virtualLabLayout.spec.js src/stores/experiment.js document/阶段记录/分支整合/merge-05-新版2D虚拟实验台.md
git commit -m "merge: 阶段05 新版2D虚拟实验台"
```

只添加实际改动文件。

---

# 11. 阶段 06：图数据库后端

## 来源

主要参考：

```text
origin/2d-logic
```

## 目标

合入知识图谱后端，作为独立后端工程，不污染前端根项目，不阻塞前端运行。

## 参考文件

```text
backend/server.js
backend/api/knowledgeGraphAPI.js
backend/neo4j/config/neo4j.config.js
backend/neo4j/queries/cypherQueries.js
backend/neo4j/scripts/initKnowledgeGraph.js
backend/test-server.js
```

参考命令：

```bash
git ls-tree -r --name-only origin/2d-logic backend
git show origin/2d-logic:backend/server.js > /tmp/backend-server.2d-logic.js
```

## 合入内容

```text
- backend/server.js
- backend/api/knowledgeGraphAPI.js
- backend/neo4j/config/neo4j.config.js
- backend/neo4j/queries/cypherQueries.js
- backend/neo4j/scripts/initKnowledgeGraph.js
- backend/test-server.js
- backend/package.json
- backend/.env
- backend/.env.example
```

如果 `2d-logic` 没有独立 `backend/package.json`，创建一个，只放后端依赖：

```text
express
cors
dotenv
neo4j-driver
```

不要把这些依赖加到前端根 `package.json`，除非前端已经实际需要。

## 配置规则

配置放在：

```text
backend/.env
backend/.env.example
```

不要新增根目录 `.env`。
如果用户已有根目录 `.env`，不要删除，但后端配置应迁到 `backend/.env`。

`.env.example` 要包含：

```text
PORT=3000
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=
```

如果后端还需要其他字段，补充到 example。

## 后端 AI 接口

不要合入 `2d-logic` 的后端 AI API。
AI 助手以阶段 02 的前端 AI 路线为主。

## 前端联动

确认阶段 01 的 `knowledgeGraphService` 可以请求：

```text
http://localhost:3000/api/knowledge-graph
```

但请求失败时仍 fallback。

## 验证

如果本地有 Node 环境：

```bash
cd backend
npm install
npm run start
```

如果没有脚本，补充：

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "node server.js"
  }
}
```

验证：

```text
不启动后端：前端知识图谱显示本地数据。
启动后端但无 Neo4j：后端给出明确错误，前端 fallback。
启动后端且 Neo4j 可用：动态图谱、搜索、学习路径可用。
```

如果本地没有 Neo4j，写入阶段文档：

```text
后端代码已合入。
前端 fallback 已验证。
Neo4j 动态接口因本地环境缺失未完成实测。
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-06-图数据库后端.md
```

## 提交

```bash
git status --short
git add backend document/阶段记录/分支整合/merge-06-图数据库后端.md
git commit -m "merge: 阶段06 图数据库后端"
```

---

# 12. 阶段 07：底层整合校验与回归修补

## 目标

确认所有页面升级后，底层物理、原理动画、3D、2D/3D 同步和实验记录口径仍然正确。

## 必查文件

```text
src/stores/experiment.js
src/stores/principleAnimation.js
src/features/experiment/photoelectricModel.js
src/features/experiment/constants.js
src/features/experiment/helpers.js
src/features/photoelectric-animation/engine.js
src/features/photoelectric-animation/mapping.js
src/features/photoelectric-animation/voltAmpTrajectory.js
src/components/PrincipleAnimationSection.vue
src/components/SplineScene.vue
src/components/animation/ThreeDMicroAnimationPanel.vue
src/components/VirtualLab.vue
src/components/ExperimentReport.vue
src/components/HistoryRecords.vue
src/components/IntegratedPlotPanel.vue
src/components/AIAssistant.vue
src/components/PhotoelectricEffectGraph.vue
src/views/HomeView.vue
```

## 差异检查

执行：

```bash
git diff origin/animation-demo...HEAD -- src/stores src/features src/components src/views
```

重点排查：

```text
- 是否出现 ai-helper 旧 generatedPhotocurrentAmp tanh 模型
- 是否出现旧 currentPhotocurrentAmp 简化合成覆盖
- 是否删除 NON_ENCLOSING_PHOTOTUBE_MODEL
- 是否删除 CUTOFF_MODE_CATHODE_TAIL
- 是否删除 resolveAnodeParasiticPhotocurrentAmp
- 是否删除 resolveCathodePhotocurrentAmp
- 是否删除 resolveNormalizedLightFlux
- 是否破坏 PrincipleAnimationSection
- 是否破坏 SplineScene
- 是否破坏 ThreeDMicroAnimationPanel
- 是否 HomeView 删除 principleAnimation 或 3d-view 入口
- 是否引入 .vite / outputs / report_assets
```

可用 grep 辅助：

```bash
grep -R "tanh((voltage + cutoff)" -n src || true
grep -R "PHOTOCURRENT_SATURATION_CURRENT_MIN_AMP" -n src || true
grep -R "resolveAnodeParasiticPhotocurrentAmp" -n src
grep -R "NON_ENCLOSING_PHOTOTUBE_MODEL" -n src
```

若 grep 出现旧常量，不一定就是错误，但必须判断它是否进入主计算链路；如果只是测试或注释，也要记录。

## 验证命令

```bash
npm run test:unit -- --run
npm run build
npm run vis:check
npm run scene:report
```

## 手动验证矩阵

```text
1. 首页：
   - 顶部导航仍有知识图谱、虚拟实验台、原理动画、数据处理、3D入口。

2. 知识图谱：
   - 无后端时显示本地图谱。
   - 搜索、筛选、主题切换正常。
   - 后端不可用时学习路径不报错。

3. 2D实验台：
   - 开电源、开盖、调零、接线、切滤光片、调电压、记录、复测、保存实验正常。

4. 数据处理：
   - 记录表显示正确。
   - 图表和记录一致。
   - 报告生成正常。
   - AI初稿正常。
   - 评分和题库结果显示正常。

5. AI助手：
   - 普通页面显示。
   - 原理动画页不遮挡。
   - 下一步指导读取当前状态。
   - fallback 正常。

6. 原理动画：
   - 截止模式正常。
   - 伏安模式正常。
   - 动画控制不被 2D 页面影响。
   - principleAnimation store 与 experiment store 隔离。

7. 3D实验：
   - 场景加载正常。
   - 2D 改参数后 3D 读数同步。
   - 3D 微观小窗模式同步。
   - 3D 按钮切换测试模式后 2D/记录状态一致。
```

## 修复要求

如果发现任何问题，直接修复，不要询问用户。
修复后重新运行相关测试。
如果某测试因环境问题无法运行，明确记录“未执行原因”，不要写成通过。

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-07-底层整合校验.md
```

## 提交

```bash
git status --short
git add <修复文件> document/阶段记录/分支整合/merge-07-底层整合校验.md
git commit -m "merge: 阶段07 底层整合校验"
```

若本阶段没有业务修复，也至少提交阶段文档。

---

# 13. 阶段 08：文档与收口

## 目标

重写 README，清理无关产物，确认最终整合分支可收拢到 main。

## README 重写要求

README 必须反映最终实际项目，不要沿用旧阶段描述。

必须包含：

```text
1. 项目总览
2. 页面结构
   - 知识图谱
   - 2D 虚拟实验台
   - 原理动画
   - 数据处理
   - 3D 实验台
3. 知识图谱说明
   - 本地数据模式
   - 后端动态图谱模式
   - 搜索、筛选、学习路径
4. 2D 虚拟实验台说明
   - 仪器电源
   - 汞灯、盖子、光电管距离
   - 滤光片、光阑、电压、调零、接线
   - 记录、复测、保存实验
   - 题库
5. 数据处理说明
   - 历史实验
   - 图表
   - 评分
   - 报告
   - AI初稿
6. AI助手说明
   - 下一步指导
   - 数据分析
   - 导出聊天
   - Key 策略和 fallback，不写具体 Key
7. 原理动画说明
   - 状态隔离
   - 截止模式
   - 伏安模式
8. 3D实验台说明
   - Spline 场景
   - 读数同步
   - 3D 微观小窗
9. 底层物理框架说明
   - 非包围式光电管模型
   - 阳极寄生电流
   - 截止尾流
   - 统一实验状态
10. 后端知识图谱启动方式
11. 常用命令
12. 测试与验收
13. 已知限制
```

## 清理检查

确认未合入：

```bash
git status --ignored --short
```

重点确认没有纳入：

```text
.vite/
outputs/
report_assets/
artifacts/visual/
artifacts/scene/
dist/
node_modules/
临时截图
临时 patch
无关旧阶段文档
```

检查 staged 文件：

```bash
git diff --cached --name-only
```

如果依赖变化，重新生成：

```bash
npm install
```

然后：

```bash
npm run test:unit -- --run
npm run build
npm run vis:check
npm run scene:report
```

## 阶段文档

写：

```text
document/阶段记录/分支整合/merge-08-文档与收口.md
```

## 提交

```bash
git status --short
git add README.md document/阶段记录/分支整合/merge-08-文档与收口.md
```

如果依赖文件实际变化，再添加：

```bash
git add package.json package-lock.json
```

提交：

```bash
git commit -m "merge: 阶段08 文档与收口"
```

---

# 14. 最终收拢到 main

## 14.1 确认整合分支状态

执行：

```bash
git status --short
git log --oneline --decorate -20
npm run test:unit -- --run
npm run build
npm run vis:check
npm run scene:report
```

若有环境导致未能执行的测试，写入最终文档。

## 14.2 合并到 main

执行：

```bash
git switch main
git pull --ff-only origin main
git merge --no-ff integration/full-branch-merge -m "merge: 整合 animation-demo、2d-logic 与 ai-helper"
```

如果 `main` 本地不存在：

```bash
git switch -c main origin/main
git merge --no-ff integration/full-branch-merge -m "merge: 整合 animation-demo、2d-logic 与 ai-helper"
```

## 14.3 main 上最终验证

```bash
npm run test:unit -- --run
npm run build
npm run vis:check
npm run scene:report
```

## 14.4 最终收拢文档

写：

```text
document/阶段记录/分支整合/merge-final-main-收拢说明.md
```

必须包含：

```text
# 最终目标
# 整合分支
# main 最终提交号
# 各来源分支提交号
# 阶段 commit 列表
# 阶段文档列表
# 合入功能清单
# 明确未合入内容
# 红线保护结果
# 最终验证命令与结果
# 未执行或失败测试及原因
# 远端推送状态
# 用户验收入口
# 已知遗留问题
```

提交：

```bash
git add document/阶段记录/分支整合/merge-final-main-收拢说明.md
git commit -m "docs: 分支整合最终收拢说明"
```

## 14.5 推送远端

如果当前环境有远端写权限：

```bash
git push origin main
```

如果无权限，不要伪造成功。写明：

```text
本地 main 已完成。
远端 main 未推送。
原因：当前环境无写权限或未配置认证。
```

如果推送被拒绝，因为远端 main 有新提交：

```bash
git fetch origin --prune
git log --oneline main..origin/main
```

如果新提交与本任务无冲突，执行：

```bash
git pull --rebase origin main
npm run test:unit -- --run
npm run build
git push origin main
```

若 rebase/冲突无法安全处理，写明阻塞，不要强推。

---

# 15. 最终回复用户格式

任务全部完成后，最终只输出：

```text
1. 当前所在分支
2. main 最新提交号
3. 是否已推送远端 main
4. 阶段提交清单
5. 阶段文档清单
6. 最终合入功能清单
7. 明确未合入内容清单
8. 验证命令与结果
9. 未通过或未执行测试及原因
10. 用户验收入口
11. 已知遗留问题
```

不要输出大量中间过程。
不要说“需要用户确认后继续”。
不要把未执行测试写成已通过。
不要编造远端推送成功。

---

# 16. 用户验收入口

最终提醒用户至少验收：

```text
首页：
- 顶部导航仍包含知识图谱、虚拟实验台、原理动画、数据处理、3D入口。

知识图谱：
- 无后端本地显示。
- 搜索、筛选、主题切换、节点详情。
- 后端可用时动态图谱和学习路径。

2D实验台：
- 开启试验仪。
- 汞灯、盖子、光电管距离、滤光片、光阑。
- 调零、接线、电压调节。
- 记录、复测、保存实验。
- 题库显示与答题。

数据处理：
- 当前记录。
- 历史实验。
- 图表。
- 评分。
- 题库成绩。
- 实验报告。
- AI初稿。

AI助手：
- 拖拽。
- 导出聊天。
- 下一步指导。
- 数据分析。
- 真实AI与备用回答。

原理动画：
- AI不遮挡。
- 截止模式正常。
- 伏安模式正常。
- 原理动画状态与真实实验状态隔离。

3D实验台：
- 场景加载。
- 2D参数同步到3D读数。
- 3D测试模式切换。
- 3D微观小窗同步。
```

---

# 17. 禁止事项汇总

禁止：

```text
1. 中途停下来问用户是否继续。
2. 整支 merge origin/2d-logic。
3. 整支 merge origin/ai-helper。
4. 覆盖 animation-demo 的物理计算。
5. 覆盖 animation-demo 的原理动画链路。
6. 覆盖 animation-demo 的 3D 资源和 Spline 链路。
7. 覆盖 animation-demo 的主页总导航。
8. 把 ai-helper 的旧电流模型带进主实验状态。
9. 让图表或报告重新算一套与 animation-demo 不一致的物理结论。
10. 把 .vite、outputs、report_assets、dist、node_modules 等产物合入。
11. 把后端依赖塞进前端根 package.json。
12. 合入 2d-logic 的后端 AI 接口。
13. 删除用户未提交文件。
14. 伪造测试通过。
15. 伪造远端推送成功。
16. 强推 main。
```

严格按照本文档从准备阶段执行到最终 main 收拢完成。
