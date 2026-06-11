# 阶段目标

本阶段从 `origin/animation-demo` 锁定整合基线，确认远端分支、当前整合分支、工作区既有改动、项目脚本、红线文件和基线验证结果。

本阶段不合入 `2d-logic` 或 `ai-helper` 的功能代码，不执行整支 merge，只建立后续模块迁移的保护边界。

# 来源分支与参考内容

- 当前整合分支：`integration/full-branch-merge`
- 当前基线提交：`add15b6`
- `origin/main`：`e37310e`
- `origin/animation-demo`：`add15b6`
- `origin/2d-logic`：`d180126`
- `origin/ai-helper`：`f385728`

已执行远端同步：

```bash
git fetch origin --prune --tags
```

远端抓取规则为：

```text
+refs/heads/*:refs/remotes/origin/*
```

# 实际改动文件

本阶段实际纳入提交的文件：

- `document/实施指导/codex_goal_engineering_prompt.md`
- `document/实施指导/codex_goal_engineering_prompt_expanded.md`
- `document/阶段记录/分支整合/merge-00-基线与红线保护.md`

说明：

- 开始 PR-0 前，工作区已有工程化文档替换：旧版 `codex_goal_engineering_prompt.md` 被删除，新增 `codex_goal_engineering_prompt_expanded.md`。
- 该改动属于本次整合任务输入，不是业务代码改动，因此纳入阶段 00 记录与提交。
- 已按文档要求保存既有差异补丁到 `artifacts/integration/preexisting-worktree.patch` 和 `artifacts/integration/preexisting-index.patch`。`artifacts/` 为忽略目录，不纳入提交。

# 涉及的关键组件、store、service、feature

本阶段只做基线检查，未修改组件、store、service 或 feature。

已记录的红线范围包括：

- `src/stores/experiment.js`：主实验 store，承载真实实验状态、记录和读数。
- `src/stores/principleAnimation.js`：原理动画独立 store。
- `src/features/experiment/photoelectricModel.js`：主物理模型。
- `src/features/experiment/constants.js`：主实验常量。
- `src/features/experiment/helpers.js`：主实验辅助计算。
- `src/features/photoelectric-animation/*`：原理动画与伏安轨迹链路。
- `src/components/PrincipleAnimationSection.vue`：原理动画入口。
- `src/components/SplineScene.vue`：3D Spline 场景桥接。
- `src/components/animation/ThreeDMicroAnimationPanel.vue`：3D 微观小窗。
- `src/views/HomeView.vue`：主页总导航和页面入口。

# 明确保留的 animation-demo 底层内容

后续阶段必须继续保留：

- 非包围式光电管模型。
- 主电流计算链路。
- 实际读数合成口径。
- 截止电压计算口径。
- 光强、光阑、距离换算口径。
- 阳极污染、阳极寄生电流和截止尾流逻辑。
- `PrincipleAnimationSection.vue` 与 `principleAnimation` store 的状态隔离。
- `photoelectric-animation` 动画引擎、映射和伏安轨迹逻辑。
- `SplineScene.vue`、`ThreeDMicroAnimationPanel.vue`、`resource/scene.splinecode` 和 `vendor/spline-runtime/`。
- `HomeView.vue` 中的知识图谱、虚拟实验台、原理动画、数据处理和 3D 入口。

# 本阶段没有合入的内容及原因

- 未合入 `2d-logic` 知识图谱前端：该内容属于阶段 01。
- 未合入 `ai-helper` AI 助手：该内容属于阶段 02。
- 未合入 `ai-helper` 数据处理、报告、评分、题库和 2D 实验台：这些内容属于阶段 03 到阶段 05。
- 未合入图数据库后端：该内容属于阶段 06。
- 未修改任何业务源码：本阶段目标是基线保护。

# 验证命令与结果

```bash
npm run test:unit -- --run
```

结果：未完全通过。共 12 个测试文件，11 个通过、1 个失败；共 157 个测试，156 个通过、1 个失败。

失败用例：

```text
src/stores/experiment.spec.js > experiment store photoelectric model integration > records model fields and marks tuned cutoff data as valid
AssertionError: expected 5.0518e-14 to be less than 5e-14
```

该失败是基线阶段发现的边界阈值差异，PR-0 不修改主计算逻辑，后续底层回归阶段统一判断和修补。

```bash
npm run build
```

结果：通过。Vite 输出大 chunk 警告，但构建成功。

```bash
npm run scene:report
```

结果：通过。脚本完成生产构建、预览启动、3D 场景加载和运行时诊断报告生成。

# 测试失败或未执行项及原因

- `npm run test:unit -- --run`：存在 1 个基线失败，用例阈值为 `5e-14`，实际记录值为 `5.0518e-14`。
- 未运行 `npm run vis:check`：阶段 00 文档要求的基线验证为单测、构建和 `scene:report`，视觉矩阵从阶段 01 起按要求执行。

# 发现的问题与处理方式

- 当前挂载路径 `/mnt/f/file/wsl_shared_files/event/new(26)` 对 Git 写入表现为只读；后续统一使用同一仓库的允许写入路径 `/home/just_monika/win_share/event/new(26)`。
- 创建 `integration/full-branch-merge` 时 Git 输出 `.git/config.lock` 的 `chmod` 警告，分支创建成功，但 upstream 未能写入配置。后续命令显式引用远端分支，不依赖 `@{u}`。
- 构建和 `scene:report` 生成的 `dist/` 与 `artifacts/` 均为忽略目录，不纳入提交。

# 下一阶段注意事项

- 阶段 01 只从 `origin/2d-logic` 迁移知识图谱前端与服务层，不能覆盖 `HomeView.vue` 主导航。
- 知识图谱必须保留本地 fallback，后端不可用时不能空白。
- 不得整支 merge `origin/2d-logic`。
- 本阶段发现的单测边界失败需要在阶段 07 的底层整合校验中复核；如果功能迁移过程中触及相关记录口径，也要重新运行单测确认是否扩大影响。
