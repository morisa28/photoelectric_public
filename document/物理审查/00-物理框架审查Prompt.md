# Codex 分轮任务 Prompt：项目物理框架审查

## 使用方式

每一轮单独复制到 Codex 中执行。  
不要一次性把所有轮次都发给 Codex。  
每轮结束后，先检查 Codex 生成的 `document/physics_review/*.md`，再继续下一轮。  
本任务默认不修改业务代码，只允许创建和更新审查文档。

## new(26) 项目适配说明

本 prompt 已按当前项目实际情况收紧。当前项目是 Vue 3 + Vite + Pinia 的光电效应实验模拟仿真平台。本审查只聚焦微观动画、实验 store、数值计算、曲线采样、数据记录和论文知识库依据。

执行时必须遵守以下项目适配规则：

1. 默认不创建 Git worktree。项目级 `AGENTS.md` 已说明普通任务优先在当前工作区处理；本审查只写 `document/physics_review/`，不需要隔离 worktree。
2. 不修改业务代码，不修改 `README.md`、`package.json`、`src/`、`scripts/`、`knowledge/`、`papers/`、`resource/`、`vendor/`。只允许创建或更新 `document/physics_review/*.md`。
3. 不读取或扫描大型/生成目录：`node_modules/`、`dist/`、`artifacts/`、`.git/`、`.worktrees/`。
4. 不审查与动画或数值计算无关的资源文件、截图基线、美术资产和自动化脚本。
5. 不批量读取 `papers/*.pdf` 或 `papers/*.caj`。优先使用 `knowledge/README.md`、`knowledge/project-mapping.md`、`knowledge/claims.md`、`knowledge/topics/*.md`、`knowledge/papers/P*.md` 和 `knowledge/paper-index.*`。只有某条结论必须核对原文时，标注“需 PDF/CAJ 原文复核”，不要在本轮强行读取大文件。
6. 大文件必须分片读取。`src/features/photoelectric-animation/engine.js`、`src/components/VirtualLab.vue`、`src/components/animation/PhotoelectricAnimationPanel.vue`、`src/stores/experiment.js` 这类文件不要一次性全文读完。
7. 结论必须标注证据等级：
   - A：代码直接证据 + 项目知识库理论依据均支持。
   - B：代码直接证据支持，但理论依据只来自通用物理常识或需后续论文复核。
   - C：知识库理论依据支持，但代码证据不完整或只看到局部片段。
   - D：推断或不确定，必须标注“需要进一步确认”。
8. 优先区分“真实物理错误”“教学仿真的合理简化”“UI/文案表达不严谨”“动画表现不完全真实”。不要把合理工程简化直接标成 P0。

---

## 第 0 轮：任务边界确认与工作目录准备

```text
你现在位于当前项目根目录。请先进行一次只读型任务准备，不要修改业务代码。

任务目标：
为后续“物理框架审查”建立最小工作区和审查边界，避免后续长任务触发上下文压缩失败。

硬性约束：
1. 不要修改任何业务代码。
2. 不要重构项目。
3. 不要安装依赖。
4. 不要运行长时间命令。
5. 不要读取 node_modules、dist、build、.git、coverage、缓存目录。
6. 不要使用视觉分析组件。
7. 本轮只允许创建或更新 document/physics_review/ 下的 md 文件。
8. 每一步都要控制上下文，不要一次性读取大文件全文。
9. 必须把本项目的适配规则写入任务边界，尤其是“不创建 worktree”“只写 document/physics_review/”“不读取大目录和论文原文全文”。

请执行：
1. 确认当前目录是否为项目根目录。
2. 简要扫描项目顶层结构。
3. 创建目录：
   - document/physics_review/
4. 创建文件：
   - document/physics_review/00_task_scope.md
5. 在 00_task_scope.md 中写入：
   - 本次审查目标
   - 审查范围
   - 禁止修改的内容
   - 允许输出的文档位置
   - 本项目中可能涉及物理的关键词列表
   - 当前项目的主要审查入口文件
   - 证据等级 A/B/C/D 的定义
   - 后续建议分轮策略

当前项目主要审查入口建议写入：
- `src/stores/experiment.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`
- `src/features/photoelectric-animation/physics.js`
- `src/features/photoelectric-animation/mapping.js`
- `src/features/photoelectric-animation/voltAmpTrajectory.js`
- `src/features/photoelectric-animation/engine.js`
- `src/components/animation/PhotoelectricAnimationPanel.vue`
- `src/components/animation/PhotoelectricAnimationCanvas.vue`
- `src/components/animation/ThreeDMicroAnimationPanel.vue`
- `src/components/VirtualLab.vue`
- `src/components/ExperimentReport.vue`
- `src/components/IntegratedPlotPanel.vue`
- `knowledge/README.md`
- `knowledge/project-mapping.md`
- `knowledge/claims.md`
- `knowledge/topics/*.md`

关键词至少包括：
- 光电效应
- 光源
- 频率
- 波长
- 光强
- 光电子
- 遏止电压
- 饱和电流
- 光电流
- 电压
- 电流
- 调零
- 误差
- 测量误差
- 仪器精度
- 电子运动
- 电场
- 阴极
- 阳极
- 光谱
- 普朗克常量
- 逸出功
- 最大初动能
- I-V 曲线
- 实验流程

输出要求：
1. 在终端输出本轮完成了什么。
2. 输出下一轮可直接复制使用的 RESUME_PROMPT。
3. 不要继续执行下一轮。
```

---

## 第 1 轮：全项目物理相关文件索引

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md

本轮目标：
只做“物理相关文件索引”，不要深入分析物理正确性，不要生成最终报告。索引必须结合当前项目实际结构，避免把整个仓库无差别扫进审查范围。

硬性约束：
1. 只读分析，不修改业务代码。
2. 只允许创建或更新 document/physics_review/01_physics_file_index.md。
3. 不要读取 node_modules、dist、build、.git、coverage、缓存目录。
4. 不要一次性读取大文件全文。
5. 对每个候选文件，只读取必要片段判断是否相关。
6. 不要使用视觉分析组件。
7. 不要对物理结论做最终判断，本轮只做索引。
8. 如果上下文接近过长，立刻停止并写出 RESUME_CHECKPOINT，不要硬撑到自动 compact。
9. 候选文件总数建议控制在 30 个以内，高优先级文件建议控制在 12 个以内。
10. 不索引与动画或数值计算无关的资源文件、页面承载壳、截图脚本或视觉基线。
11. 对知识库只索引 `knowledge/` 下的摘要、主题、claims 和项目映射；`papers/` 原文只登记路径和格式，不作为本轮深读对象。

请扫描项目中可能涉及物理逻辑、实验流程、实验数据、UI 显示、数值计算、状态管理、动画控制、交互逻辑的文件。

优先扫描这些路径：
- `src/stores/experiment.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`
- `src/features/photoelectric-animation/physics.js`
- `src/features/photoelectric-animation/mapping.js`
- `src/features/photoelectric-animation/voltAmpTrajectory.js`
- `src/features/photoelectric-animation/engine.js`
- `src/components/animation/*.vue`
- `src/components/VirtualLab.vue`
- `src/components/ExperimentReport.vue`
- `src/components/IntegratedPlotPanel.vue`
- `knowledge/README.md`
- `knowledge/project-mapping.md`
- `knowledge/claims.md`
- `knowledge/topics/*.md`

一般不纳入深度审查：
- 与动画或数值计算无关的资源文件、页面承载壳、截图检查和自动化相关文件
- `papers/*.pdf`
- `papers/*.caj`
- `picture/`
- `animation/ani_document/` 中的历史迭代文档，除非当前代码证据不足且需要追溯设计意图。

重点查找：
1. 光电效应理论相关逻辑
2. 光源、频率、波长、光强相关逻辑
3. 电压、电流、光电流相关逻辑
4. 遏止电压、饱和电流、I-V 曲线相关逻辑
5. 调零、误差、随机扰动、测量精度相关逻辑
6. 实验步骤、按钮、旋钮、滑块、接线流程中会影响数值状态的逻辑
7. 微观原理动画、电子运动、轨迹、收集率和电场映射逻辑
8. 图表、数据记录、实验结果导出相关逻辑
9. 动画状态适配逻辑，包括 `ThreeDMicroAnimationPanel.vue` 中复用微观动画状态的部分
10. 文档或知识库中与物理理论相关的论文、资料、说明

请在 document/physics_review/01_physics_file_index.md 中输出表格，字段包括：
- 序号
- 文件路径
- 文件类型
- 相关程度：高 / 中 / 低
- 涉及的物理关键词
- 可能涉及的物理模块
- 初步判断依据
- 建议后续审查优先级
- 下一轮是否需要深入读取
- 建议读取方式：全文 / 分片 / 只读关键函数 / 只登记路径
- 预期证据等级：A / B / C / D

同时请在文件末尾给出：
1. 高优先级文件清单
2. 中优先级文件清单
3. 低优先级文件清单
4. 建议下一轮优先分析的前 3-5 个文件
5. 下一轮可直接复制使用的 RESUME_PROMPT

本轮停止条件：
完成 01_physics_file_index.md 后立即停止，不要继续深入分析。
```

---

## 第 2 轮：知识库论文与理论依据索引

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md

本轮目标：
只整理“知识库论文 / 理论依据索引”，不要审查业务代码，不要生成最终报告。

硬性约束：
1. 不修改业务代码。
2. 只允许创建或更新 document/physics_review/02_physics_knowledge_index.md。
3. 不要使用互联网。
4. 只基于项目内已有知识库、论文、文档、README、资料文件。
5. 不要一次性读取大 PDF 或大文档全文。
6. 优先读取标题、摘要、目录、关键公式、结论、与光电效应相关章节。
7. 不要使用视觉分析组件，除非项目内资料无法以文本读取且确实必须查看图片公式；如必须使用，先说明原因。
8. 如果上下文接近过长，立刻停止并写出 RESUME_CHECKPOINT。
9. 现有知识库已经把 32 篇论文整理为摘要和主题索引；本轮优先读取 `knowledge/`，不要把 `papers/` 原文全文展开进上下文。
10. 如果某条理论依据只来自 `knowledge/claims.md` 或 `knowledge/topics/*.md`，在输出中标注“知识库摘要依据”。如果需要原文页码或图表支持，标注“需 PDF/CAJ 原文复核”。
11. 不允许使用互联网补材料；缺失就标注缺失。

请在项目内查找可能的知识库资料目录，例如但不限于：
- docs/
- document/
- papers/
- paper/
- knowledge/
- knowledge_base/
- reference/
- references/
- assets/docs/
- public/docs/
- README 或说明文档
- 中文命名的“知识库”“论文”“参考资料”“物理资料”等目录

当前项目优先读取：
- `knowledge/README.md`
- `knowledge/project-mapping.md`
- `knowledge/claims.md`
- `knowledge/claims.json`
- `knowledge/review-results.csv`
- `knowledge/paper-index.csv`
- `knowledge/source-coverage.json`
- `knowledge/topics/*.md`
- `knowledge/papers/P*.md`（按需要读取，不要一次性全部展开）
- `README.md` 中的物理模型、状态流和微观动画说明
- `document/` 中最新阶段总结，只在需要追溯设计意图时读取

请重点查找以下理论依据：
1. 爱因斯坦光电效应方程
2. 光子能量与频率关系
3. 最大初动能与遏止电压关系
4. 截止频率与逸出功关系
5. 光强与光电流关系
6. 频率与遏止电压关系
7. 光电流-电压曲线规律
8. 饱和电流概念
9. 反向电压与截止电流
10. 实验误差、仪器精度、调零误差
11. 汞灯光谱、滤光片、不同频率光源
12. 教学仿真实验中允许的工程简化

请在 document/physics_review/02_physics_knowledge_index.md 中输出：
1. 资料文件清单
2. 每个资料文件的主题
3. 与本项目相关的物理结论
4. 可作为审查依据的公式或规律
5. 需要特别对照项目实现的理论点
6. 资料中没有覆盖、后续需要谨慎判断的点

每条理论依据请尽量包含：
- 理论名称
- 简要说明
- 对应公式，若有
- 适用条件
- 与项目仿真的关联
- 后续审查时要检查的实现点
- 来源文件路径
- 证据等级：A / B / C / D
- 是否需要论文原文复核：是 / 否

请在文件末尾输出：
1. 后续代码审查应优先使用的理论依据清单
2. 下一轮建议分析的项目文件
3. 下一轮可直接复制使用的 RESUME_PROMPT

本轮停止条件：
完成 02_physics_knowledge_index.md 后立即停止，不要继续审查业务代码。
```

---

## 第 3 轮：第一批高优先级文件逐文件审查

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md
- document/physics_review/02_physics_knowledge_index.md

本轮目标：
只分析 01_physics_file_index.md 中“高优先级”的首批文件。
普通小文件最多分析 3 个；大文件每轮只分析 1 个。
不要生成最终报告。

硬性约束：
1. 只读分析，不修改业务代码。
2. 只允许创建或更新 document/physics_review/03_file_level_review_part1.md。
3. 每分析完 1 个文件，必须立刻把结果写入 md，不能等全部分析完再写。
4. 不要一次性读取无关文件全文。
5. 必须把项目实现与 02_physics_knowledge_index.md 中的理论依据对照。
6. 对不确定的地方要标注“不确定”，不要编造。
7. 不要使用视觉分析组件。
8. 如果上下文接近过长，立刻停止并写出 RESUME_CHECKPOINT。
9. 如果首批文件包含 `engine.js`、`VirtualLab.vue`、`PhotoelectricAnimationPanel.vue` 或 `experiment.js`，本轮只分析其中 1 个大文件。
10. 每个判断都要标注证据等级 A/B/C/D。没有代码证据的问题不能标 P0。
11. 优先从以下顺序开始：`src/features/experiment/constants.js`、`src/features/experiment/helpers.js`、`src/stores/experiment.js`、`src/features/photoelectric-animation/mapping.js`、`src/features/photoelectric-animation/physics.js`、`src/features/photoelectric-animation/voltAmpTrajectory.js`、`src/features/photoelectric-animation/engine.js`。

请对每个文件按以下结构分析：

## 文件：路径

### 1. 文件职责

说明这个文件在项目中负责什么。

### 2. 涉及的物理概念

列出它涉及的物理概念，例如：
- 光电效应
- 光强
- 频率
- 波长
- 光电流
- 遏止电压
- 饱和电流
- 调零误差
- 测量误差
- 实验流程
- I-V 曲线
- 电子运动
- 光源切换
- 接线状态

### 3. 当前实现逻辑

用工程语言说明当前代码如何实现这些物理现象或实验流程。  
需要指出关键变量、函数、状态、组件、数据流。

### 4. 与现实物理理论的一致点

列出当前实现中符合现实物理理论或教学实验规律的地方。

### 5. 与现实物理理论的偏差

逐条列出偏差：
- 当前实现是什么
- 现实理论应该是什么
- 差异在哪里
- 可能造成什么教学误导或实验误差
- 证据等级：A / B / C / D
- 是否可能只是教学仿真简化：是 / 否 / 需要进一步确认

### 6. 当前这样实现的可能工程理由

不要只批评，也要解释为什么当前实现可能这样设计，例如：
- 简化教学流程
- 降低交互复杂度
- 避免学生难以理解
- 保证动画表现
- 减少计算复杂度
- 适应 Vue / Canvas / 前端状态限制
- 为了演示效果牺牲严格物理

### 7. 建议改动方向

分为：
- 必须改
- 建议改
- 可保留
- 需要进一步确认

### 8. 改动理由

说明为什么要这样改，改了能提升什么：
- 物理真实性
- 教学一致性
- 实验逻辑可信度
- 数据解释能力
- 用户交互合理性
- 后续扩展性

### 9. 风险与代价

说明改动可能带来的工程代价：
- 是否影响现有交互
- 是否影响现有动画交互或数值显示
- 是否增加状态复杂度
- 是否影响 UI 展示
- 是否需要补充测试

### 10. 建议验证方法

提出如何验证改动正确：
- 单元测试
- 手动操作路径
- 数据点检查
- 曲线形状检查
- UI 显示检查
- 边界条件检查

每个文件最后给出一个小结：
- 物理可靠性评分：1-5
- 工程可接受度评分：1-5
- 修改优先级：P0 / P1 / P2 / P3
- 是否建议进入最终报告
- 证据等级汇总
- 需要后续跨文件验证的问题

本轮停止条件：
分析完最多 3 个小文件，或 1 个大文件后立即停止。
在终端输出下一轮可直接复制使用的 RESUME_PROMPT，用于继续分析下一批文件。
```

---

## 第 4 轮：后续批次文件审查通用模板

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md
- document/physics_review/02_physics_knowledge_index.md
- 已存在的 document/physics_review/03_file_level_review_part*.md

本轮目标：
继续分析 01_physics_file_index.md 中尚未深入审查的高优先级或中优先级文件。
普通小文件本轮最多分析 3 个；大文件本轮只分析 1 个。
不要生成最终报告。

硬性约束：
1. 只读分析，不修改业务代码。
2. 只允许创建或更新新的分批审查文件，例如：
   - document/physics_review/03_file_level_review_part2.md
   - document/physics_review/03_file_level_review_part3.md
   具体编号请根据已有文件自动递增。
3. 不要重复分析已经在 part 文件中完成审查的文件。
4. 每分析完 1 个文件，必须立刻写入 md。
5. 必须对照 02_physics_knowledge_index.md。
6. 不要使用视觉分析组件。
7. 如果上下文接近过长，立刻停止并写出 RESUME_CHECKPOINT。
8. 大文件标准：超过 800 行，或包含复杂 Canvas 动画、粒子运动、状态映射或数值计算的文件。大文件必须只选 1 个，并按函数/片段读取。
9. 不审查与动画或数值计算无关的资源细节、页面承载壳或自动化脚本。
10. 对每条偏差都标注证据等级 A/B/C/D 和是否属于可接受工程简化。

请先判断：
1. 哪些文件已经分析过。
2. 哪些高优先级文件尚未分析。
3. 如果高优先级已完成，则分析中优先级文件。
4. 本轮最多选择 3 个最值得继续分析的小文件；如果候选中包含大文件，则只选择 1 个大文件。

每个文件按以下结构输出：

## 文件：路径

### 1. 文件职责
### 2. 涉及的物理概念
### 3. 当前实现逻辑
### 4. 与现实物理理论的一致点
### 5. 与现实物理理论的偏差
### 6. 当前这样实现的可能工程理由
### 7. 建议改动方向
### 8. 改动理由
### 9. 风险与代价
### 10. 建议验证方法
### 11. 文件级小结

文件级小结必须包括：
- 物理可靠性评分：1-5
- 工程可接受度评分：1-5
- 修改优先级：P0 / P1 / P2 / P3
- 是否建议进入最终报告
- 是否需要后续跨文件综合判断
- 证据等级汇总
- 后续需要复核的具体文件或论文编号

本轮结束时，请输出：
1. 本轮完成的文件清单
2. 剩余待审查文件清单
3. 是否还需要继续运行本模板
4. 下一轮可直接复制使用的 RESUME_PROMPT

本轮停止条件：
最多分析 3 个小文件，或 1 个大文件后立即停止。
```

---

## 第 5 轮：跨文件物理框架综合分析

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md
- document/physics_review/02_physics_knowledge_index.md
- document/physics_review/03_file_level_review_part*.md

本轮目标：
不再逐文件展开，而是从全局角度总结当前项目的物理框架体系。
不要修改业务代码。
不要生成最终报告，只生成综合分析中间稿。

硬性约束：
1. 只读分析，不修改业务代码。
2. 只允许创建或更新 document/physics_review/04_cross_file_physics_framework.md。
3. 不要重新全仓库扫描，除非前面文档中存在明显缺口。
4. 不要重复粘贴逐文件分析全文，要做归纳。
5. 不要使用视觉分析组件。
6. 如果上下文接近过长，立刻停止并写出 RESUME_CHECKPOINT。
7. 综合分析只能基于前面已经落盘的索引和逐文件审查；如果证据不足，应明确标注“证据不足”，不要补脑。
8. P0/P1 的判定必须同时满足：有明确代码证据，有明确理论依据，且会造成核心教学误导或实验数据错误。
9. 对“教学仿真合理简化”单独归类，不要混入“物理错误”清单。

请从以下维度综合分析：

## 1. 项目当前物理框架总览

说明项目目前如何表达光电效应实验：
- 宏观实验流程
- 微观原理表达
- 光源参数
- 电压 / 电流关系
- 数据记录
- 曲线展示
- 调零与误差
- 动画状态与数值模型之间的映射

## 2. 物理概念覆盖情况

列出项目已经覆盖的物理概念：
- 已覆盖且较合理
- 已覆盖但存在偏差
- 暂未覆盖但建议补充
- 不适合在当前工程阶段加入

## 3. 关键物理关系审查

逐项分析：

### 3.1 光子能量与频率

检查是否体现：
- E = hν
- 频率越高，单个光子能量越高
- 波长与频率反比

### 3.2 逸出功与截止频率

检查是否体现：
- 只有频率超过截止频率才产生光电子
- 不同金属材料可能有不同逸出功
- 低于截止频率时增加光强也不能产生光电子

### 3.3 最大初动能与遏止电压

检查是否体现：
- eUc = hν - W
- 遏止电压主要与频率有关
- 遏止电压不应主要由光强决定

### 3.4 光强与光电流

检查是否体现：
- 在频率超过截止频率时，光强越大，光电子数量越多
- 光强主要影响光电流大小
- 光强不应直接决定最大初动能

### 3.5 光电流-电压曲线

检查是否体现：
- 正向电压增大时电流趋向饱和
- 反向电压增大时电流逐渐减小
- 达到遏止电压附近时电流接近 0
- 曲线应有合理形状，而不是纯线性或随意变化

### 3.6 调零与测量误差

检查是否体现：
- 调零是仪器基线校准，不是改变物理规律
- 调零误差应影响读数，不应改变真实光电流
- 随机误差应受控在合理范围
- 显示精度应与仪器设定一致

### 3.7 接线与实验流程

检查是否体现：
- 未接线、接错线、接线后状态变化
- 测量前后状态逻辑
- 调零确认与读数显示之间的因果关系
- 交互流程是否符合真实实验或教学实验习惯

## 4. 主要不符合现实物理理论的地方

每条问题请按以下结构：
- 问题标题
- 涉及文件
- 当前实现
- 现实理论
- 差异说明
- 教学影响
- 修改优先级
- 建议改动方向
- 证据等级：A / B / C / D
- 是否属于可接受工程简化

## 5. 当前实现中可以保留的工程简化

不是所有偏差都必须修改。请列出：
- 哪些简化可以保留
- 为什么可以保留
- 需要在 UI 或说明中如何解释
- 是否会误导学生

## 6. 推荐的物理框架改造方向

请分层提出建议：

### P0：必须修复

严重违背核心物理规律，可能导致错误教学理解。

### P1：强烈建议修复

影响实验可信度或数据解释，但不一定阻断使用。

### P2：建议优化

提升真实感、教学性或扩展性。

### P3：可选增强

更高级的物理建模、动画表达或数值验证。

## 7. 推荐后续验证方案

包括：
- 参数测试
- 曲线测试
- 交互流程测试
- 边界条件测试
- 教学解释一致性测试

本轮结束后，请输出：
1. document/physics_review/04_cross_file_physics_framework.md 已完成的说明
2. 最终报告建议结构
3. 下一轮可直接复制使用的 RESUME_PROMPT

本轮停止条件：
完成综合分析中间稿后立即停止，不要生成最终报告。
```

---

## 第 6 轮：生成最终物理框架审查报告

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md
- document/physics_review/02_physics_knowledge_index.md
- document/physics_review/03_file_level_review_part*.md
- document/physics_review/04_cross_file_physics_framework.md

本轮目标：
生成最终 Markdown 报告：
- document/physics_review/final_physics_framework_review.md

硬性约束：
1. 不修改业务代码。
2. 不重新进行大规模代码扫描。
3. 不把所有中间文档原文复制进最终报告，要做归纳、重组、压缩。
4. 所有关键结论必须能追溯到前面的审查文档或项目文件。
5. 对不确定的内容必须标注“不确定”或“需要进一步确认”。
6. 不要使用视觉分析组件。
7. 如果上下文接近过长，优先生成已确认部分，不要硬撑到自动 compact。
8. 最终报告必须包含“证据等级说明”和“未复核范围”。没有 A 或 B 级证据的问题，不得作为 P0。
9. 如果某个结论依赖论文但没有原文页码或图表核验，必须标注“基于项目知识库摘要，需论文原文复核”。

最终报告结构如下：

# 项目物理框架审查报告

## 1. 审查背景与目标

说明本次审查的目标：
- 识别项目中所有与物理理论或物理实际相关的部分
- 对照项目知识库中的论文和理论资料
- 判断当前物理框架是否符合现实光电效应理论
- 找出偏差、解释原因、提出改动方向
- 为后续工程修改提供依据

## 2. 审查范围

列出：
- 审查过的代码文件
- 审查过的文档 / 知识库资料
- 未审查或无法确认的范围
- 本报告不涉及的内容

同时列出证据等级定义：
- A：代码直接证据 + 项目知识库理论依据均支持。
- B：代码直接证据支持，但理论依据只来自通用物理常识或需后续论文复核。
- C：知识库理论依据支持，但代码证据不完整或只看到局部片段。
- D：推断或不确定。

## 3. 当前项目物理框架总览

从整体上说明项目当前如何模拟光电效应实验：
- 实验装置
- 光源
- 光电管
- 电压 / 电流
- 调零
- 接线
- 数据记录
- 曲线绘制
- 微观原理动画
- 动画状态适配与数值显示

## 4. 已实现的物理概念清单

用表格列出：
- 物理概念
- 项目中的实现位置
- 当前实现方式
- 是否基本合理
- 主要问题
- 修改优先级
- 证据等级

## 5. 与现实物理理论一致的部分

列出当前项目做得合理的地方，并说明为什么合理。

## 6. 与现实物理理论不一致或存在偏差的部分

这是报告重点。每个问题按以下结构写：

### 问题 N：标题

- 涉及文件：
- 当前实现：
- 现实物理理论：
- 差异说明：
- 可能造成的教学误导：
- 当前这样实现的可能原因：
- 建议改动方向：
- 改动理由：
- 修改优先级：
- 验证方法：
- 证据等级：
- 是否需论文原文复核：
- 是否属于教学仿真可接受简化：

重点问题至少覆盖：
1. 光强、频率、光电流、遏止电压之间的关系是否正确
2. 低于截止频率时是否错误地产生光电流
3. 光强是否错误影响最大初动能或遏止电压
4. I-V 曲线是否过度线性或不符合光电管特征
5. 调零逻辑是否把仪器校准和真实物理量混淆
6. 随机误差是否合理
7. 电压 / 电流显示精度是否合理
8. 接线流程是否符合实验因果关系
9. 微观原理动画是否准确表达光电子逸出过程
10. 动画状态与数值计算结果是否一致

## 7. 当前实现的合理工程简化

说明哪些地方虽然不完全真实，但可以作为教学仿真保留。
每条包括：
- 简化点
- 为什么可以保留
- 可能风险
- 是否需要 UI 文案解释

## 8. 建议改动方案

按优先级输出：

### P0：必须修复

列出严重物理错误。

### P1：强烈建议修复

列出影响实验可信度的问题。

### P2：建议优化

列出提升教学质量的问题。

### P3：可选增强

列出更高级但非必须的增强。

每条建议包括：
- 改动内容
- 涉及文件
- 改动理由
- 预期收益
- 工程风险
- 验证方式
- 证据等级
- 最小可行修改范围

## 9. 推荐的目标动画与数值物理模型

提出一个更合理但仍适合前端教学仿真的动画与数值计算模型。

至少包括：
1. 输入参数
   - 频率 ν
   - 波长 λ
   - 光强 I
   - 电压 U
   - 逸出功 W
   - 仪器误差
   - 接线状态
   - 调零状态

2. 核心关系
   - E = hν
   - Ek,max = hν - W
   - eUc = hν - W
   - 光强影响饱和电流
   - 频率影响遏止电压
   - 低于截止频率时无真实光电流

3. 工程简化建议
   - 使用归一化单位
   - 使用教学型曲线拟合
   - 使用受控随机误差
   - 使用显示层误差而不改变真实物理量

## 10. 验证方案

给出具体测试路径：
- 参数边界测试
- 曲线形状测试
- 光强变化测试
- 频率变化测试
- 截止频率测试
- 调零流程测试
- 接线流程测试
- UI 显示测试
- 数据记录测试

## 11. 结论

给出总体评价：
- 当前物理框架是否可用于教学演示
- 最大风险是什么
- 最优先要改什么
- 哪些内容可以保留
- 后续迭代建议

输出要求：
1. 最终报告必须写入 document/physics_review/final_physics_framework_review.md。
2. 报告语言使用中文。
3. 语气专业、工程化、可执行。
4. 不要只写理论，要把理论和项目实现对应起来。
5. 结束后在终端输出报告路径和下一步建议。
6. 完成后停止。
```

---

## 第 7 轮：最终报告质量复核与补强

```text
继续进行项目物理框架审查。

请先阅读：
- document/physics_review/final_physics_framework_review.md
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md
- document/physics_review/02_physics_knowledge_index.md
- document/physics_review/03_file_level_review_part*.md
- document/physics_review/04_cross_file_physics_framework.md

本轮目标：
只复核最终报告质量，并补强证据链。
不要修改业务代码。

硬性约束：
1. 不修改业务代码。
2. 只允许更新 document/physics_review/final_physics_framework_review.md。
3. 不做大规模重新扫描。
4. 只在发现报告证据不足时，回看必要文件片段。
5. 不要使用视觉分析组件。
6. 不要扩写成空泛长文，要提升准确性和可执行性。
7. 检查报告是否把合理工程简化误判为 P0/P1；如果是，降级或移入“合理简化”章节。
8. 检查每个 P0/P1 是否有 A 或 B 级证据；没有则降级或标注需复核。

请检查最终报告是否满足：

## 1. 覆盖完整性

- 是否覆盖所有高优先级物理相关文件
- 是否覆盖主要物理概念
- 是否覆盖实验流程
- 是否覆盖数值模型
- 是否覆盖 UI 显示和交互
- 是否覆盖误差与调零

## 2. 物理准确性

检查是否正确区分：
- 光强影响光电子数量 / 光电流
- 频率影响单个光子能量 / 最大初动能 / 遏止电压
- 低于截止频率时不会产生真实光电效应
- 遏止电压与最大初动能关系
- 调零误差与真实物理量的区别
- 仪器显示误差与底层物理模型的区别

## 3. 工程可执行性

检查每条建议是否包含：
- 涉及文件
- 当前问题
- 修改方向
- 修改理由
- 风险
- 验证方法

## 4. 证据链

检查报告中的关键结论是否能追溯到：
- 项目文件
- 中间审查文档
- 知识库理论依据
- 证据等级 A/B/C/D
- 需要原文复核的论文编号或文件路径

## 5. 优先级合理性

检查 P0 / P1 / P2 / P3 是否分得合理。  
严重违背核心光电效应理论的问题应为 P0 或 P1。  
纯展示增强不应被标成 P0。

请直接更新 final_physics_framework_review.md，使其更加：
- 准确
- 具体
- 工程可执行
- 避免空泛批评
- 避免没有依据的判断

本轮结束后输出：
1. 修改了报告哪些部分
2. 仍然不确定的点
3. 后续真正修改代码时建议从哪个问题开始
4. 完成后停止。
```

---

## 通用断点续跑 Prompt

当 Codex 因上下文、网络或 compact 报错中断后，可以新开 Codex 会话，复制下面这个 prompt：

```text
继续上一个“物理框架审查”任务。

请先不要修改业务代码。
请先阅读 document/physics_review/ 下已有的所有 md 文件，恢复当前任务状态。

你需要判断：
1. 已经完成了哪些轮次
2. 哪些文件已经审查
3. 哪些文件还没有审查
4. 上一次任务可能中断在哪一步
5. 下一步最小可执行任务是什么

硬性约束：
1. 不要重新全仓库扫描，除非现有索引明显缺失。
2. 不要重复生成已经完成的内容。
3. 不要使用视觉分析组件。
4. 每完成一个小步骤都要落盘到 document/physics_review/。
5. 如果上下文接近过长，立即停止并输出 RESUME_CHECKPOINT。
6. 恢复时先检查是否已经存在 `document/physics_review/00_task_scope.md`，并沿用其中的项目适配规则，不要重新发散扫描。

请先输出当前恢复状态，然后只执行一个最小任务：
- 如果 00_task_scope.md 不存在，执行第 0 轮。
- 如果 01_physics_file_index.md 不存在，执行第 1 轮。
- 如果 02_physics_knowledge_index.md 不存在，执行第 2 轮。
- 如果还有高优先级文件未逐文件审查，执行下一批审查：小文件最多 3 个，大文件 1 个。
- 如果逐文件审查已完成但 04_cross_file_physics_framework.md 不存在，执行跨文件综合分析。
- 如果 final_physics_framework_review.md 不存在，生成最终报告。
- 如果最终报告已存在，执行质量复核。

完成一个最小任务后立即停止，并输出下一轮 RESUME_PROMPT。
```

---

## 更保守的超短批次审查 Prompt

如果 remote compact 仍然频繁失败，用这个更保守版本。每轮只分析 1 个文件：

```text
继续物理框架审查。

请先阅读：
- document/physics_review/00_task_scope.md
- document/physics_review/01_physics_file_index.md
- document/physics_review/02_physics_knowledge_index.md
- 已存在的 document/physics_review/03_file_level_review_part*.md

本轮只允许分析 1 个尚未审查的最高优先级文件。
不要分析第二个文件。
不要生成最终报告。
不要修改业务代码。
不要使用视觉分析组件。
必须标注证据等级 A/B/C/D。

步骤：
1. 判断下一个最值得分析的文件。
2. 只读取该文件必要内容。
3. 对照知识库理论依据。
4. 将结果追加写入新的或已有的 document/physics_review/03_file_level_review_part_single.md。
5. 写完立即停止。

分析结构：
- 文件路径
- 文件职责
- 涉及物理概念
- 当前实现逻辑
- 符合现实物理理论的地方
- 不符合现实物理理论的地方
- 当前这样实现的可能原因
- 建议改动方向
- 改动理由
- 风险与代价
- 验证方法
- 物理可靠性评分
- 工程可接受度评分
- 修改优先级
- 是否进入最终报告
- 证据等级
- 是否可能只是教学仿真简化
- 是否需论文原文复核

完成后输出下一轮 RESUME_PROMPT。
```

---

## 推荐执行顺序

建议实际使用时按这个顺序运行：

```text
第 0 轮
→ 第 1 轮
→ 第 2 轮
→ 第 3 轮
→ 第 4 轮循环多次（小文件最多 3 个，大文件 1 个）
→ 第 5 轮
→ 第 6 轮
→ 第 7 轮
```

如果仍然在第 3 / 第 4 轮触发 remote compact，就改用“更保守的超短批次审查 Prompt”，每次只审查 1 个文件。
