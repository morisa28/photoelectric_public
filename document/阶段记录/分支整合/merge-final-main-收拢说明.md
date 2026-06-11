# merge-final-main-收拢说明

## 最终目标

将 `integration/full-branch-merge` 中已经阶段化整理的功能收拢到 `main`，完成 `animation-demo`、`2d-logic`、`ai-helper` 的工程化整合。整合过程不整支 merge `2d-logic` 或 `ai-helper`，保留 `animation-demo` 的底层物理、真实实验状态、原理动画、3D/Spline 链路和主页导航。

## 整合分支

- 主收拢分支：`main`
- 阶段整合分支：`integration/full-branch-merge`
- 前置准备分支：`integration/animation-demo-feature-merge`

## 提交号

- main 收拢 merge commit：`2b594f6`
- integration/full-branch-merge：`c784316`
- origin/main 基线：`e37310e`
- origin/animation-demo：`add15b6`
- origin/2d-logic：`d180126`
- origin/ai-helper：`f385728`

本文档提交后，`main` 最新提交号会包含本文档提交；最终交付回复以 `git rev-parse --short HEAD` 的结果为准。

## 阶段 commit 列表

- `d2b376a` merge: 阶段00 基线与红线保护
- `72a4382` merge: 阶段01 新版知识图谱前端
- `2700c5a` merge: 阶段02 新版AI助手
- `3ef3820` merge: 阶段03 新版数据处理与报告
- `97b2ac6` merge: 阶段04 评分系统与题库状态
- `6381210` merge: 阶段05 新版2D虚拟实验台
- `036cbd0` merge: 阶段06 图数据库后端
- `bdde241` merge: 阶段07 底层整合校验
- `c784316` merge: 阶段08 README重写与收口

## 阶段文档列表

- `document/阶段记录/分支整合/merge-00-基线与红线保护.md`
- `document/阶段记录/分支整合/merge-01-新版知识图谱前端.md`
- `document/阶段记录/分支整合/merge-02-新版AI助手.md`
- `document/阶段记录/分支整合/merge-03-新版数据处理与报告.md`
- `document/阶段记录/分支整合/merge-04-评分系统与题库状态.md`
- `document/阶段记录/分支整合/merge-05-新版2D虚拟实验台.md`
- `document/阶段记录/分支整合/merge-06-图数据库后端.md`
- `document/阶段记录/分支整合/merge-07-底层整合校验.md`
- `document/阶段记录/分支整合/merge-08-README重写与收口.md`
- `document/阶段记录/分支整合/merge-final-main-收拢说明.md`

## 合入功能清单

- 新版知识图谱前端：图谱组件、科学家资源、知识图谱服务、本地数据 fallback。
- 新版 AI 助手：聊天服务、实验指导、DeepSeek 环境变量配置、测试覆盖。
- 数据处理与报告：实验报告、历史记录、集成绘图面板、AI 教学洞察。
- 操作评分与题库：评分组件、题库组件、服务和辅助测试。
- 新版 2D 虚拟实验台：新版虚拟实验台、集成控制台、控制器组合式逻辑、复测辅助状态。
- 图数据库后端：独立 `backend/` 工程、Neo4j 配置、知识图谱 API、初始化脚本和示例环境文件。
- README 收口：重写项目说明、运行方式、功能清单、验证方式和整合边界。

## 明确未合入内容

- 未整支 merge `2d-logic`。
- 未整支 merge `ai-helper`。
- 未引入 `backend/api/aiAPI.js`。
- 未复制真实 `.env`、密钥、证书或 token。
- 未安装后端依赖，未生成 `backend/package-lock.json`。
- 未提交 `node_modules/`、`.vite/`、`outputs/`、`report_assets/`、`artifacts/` 等临时或构建产物。

## 红线保护结果

- `rg -n "tanh\\(\\(voltage \\+ cutoff" src` 无命中。
- `rg -n "backend/api/aiAPI|/api/ai" backend src` 无命中。
- `git ls-files | rg "(^|/)(node_modules|\\.vite|outputs|report_assets|artifacts)(/|$)|(^|/)\\.env$"` 无命中。
- `animation-demo` 的 3D/Spline 链路、真实实验状态和原理动画链路在最终验证中保持可用。

## 最终验证命令与结果

- `npm run test:unit -- --run`：通过，21 个测试文件、220 个测试用例通过。
- `npm run build`：通过，Vite 输出 chunk 体积警告。
- `npm run vis:check -- --port 4186`：通过，3D bridge ready，`objectCount: 70`，`visibleObjectCount: 70`，baseline 无 missing/added。
- `npm run scene:report -- --port 4187`：通过，`resource/scene.splinecode` 哈希 `e32f07f2c0d641d76bf7d9df1003b160d03a8a959bb85ed89be7986aec90120b`，runtime ready，70 个对象可见。

## 未执行或失败测试及原因

- 未执行后端 Neo4j 集成测试：本次没有安装后端依赖，也没有配置真实 Neo4j 连接环境。
- 未启动真实知识图谱后端做端到端联调：前端已验证无后端时可 fallback 到本地知识图谱数据。

## 远端推送状态

本文档创建时尚未推送；本文档提交后将执行 `git push origin main`。最终推送结果以交付回复为准，不做强推。

## 用户验收入口

- 前端本地开发：`npm run dev`
- 生产构建预览：`npm run build` 后使用 `npm run preview`
- 3D 页面：`/3d-view`
- 主页面功能入口：知识图谱、虚拟实验台、原理动画、数据处理、AI 助手

## 已知遗留问题

- 后端 Neo4j 需要用户提供 `backend/.env` 并安装 `backend/` 依赖后才能实际连接。
- 视觉基准截图哈希发生变化，属于本次 UI/功能整合后的预期变化；`vis:check` 未发现基准缺失或新增项。
- 早前阶段验证中曾启动过 `127.0.0.1:4181` 预览进程，普通沙箱下无法确认已终止；后续验证已改用 `4186`、`4187`。
