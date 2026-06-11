# 阶段目标

本阶段重写 README，并执行收口清理检查，确认最终整合分支具备收拢到 `main` 的文档基础。

# 实际改动文件

- `README.md`
- `document/阶段记录/分支整合/merge-08-README重写与收口.md`

# README 覆盖内容

新版 README 已按最终项目状态重写，包含以下栏目：

1. 项目总览
2. 页面结构
3. 知识图谱说明
4. 2D 虚拟实验台说明
5. 数据处理说明
6. AI 助手说明
7. 原理动画说明
8. 3D 实验台说明
9. 底层物理框架说明
10. 后端知识图谱启动方式
11. 常用命令
12. 测试与验收
13. 已知限制

README 明确说明：

- 知识图谱的本地数据模式和后端动态图谱模式。
- 后端不可用时的本地 fallback。
- 2D 实验台的电源、汞灯、盖子、光电管距离、滤光片、光阑、电压、调零、接线、记录、复测、保存实验和题库。
- 数据处理中的历史实验、图表、评分、报告和 AI 初稿。
- AI 助手的下一步指导、数据分析、导出聊天、Key 策略和 fallback。
- 原理动画的状态隔离、截止模式和伏安模式。
- 3D 实验台的 Spline 场景、读数同步和 3D 微观小窗。
- 底层物理框架中的非包围式光电管模型、阳极寄生电流、截止尾流和统一实验状态。
- 后端知识图谱启动方式和 `.env` 策略。
- 常用命令、测试验收和已知限制。

# 清理检查

执行：

```bash
git status --ignored --short | rg '(^|/)\\.vite/|(^|/)outputs/|report_assets|artifacts/visual|artifacts/scene|dist/|node_modules/|tmp|patch|\\.png$'
```

结果：

```text
!! dist/
!! node_modules/
```

说明：

- `dist/` 和 `node_modules/` 为本地忽略目录，未进入暂存。
- 未发现 `.vite/`、`outputs/`、`report_assets/`、`artifacts/visual/`、`artifacts/scene/`、临时截图、临时 patch 或其他无关产物进入待提交范围。

# 验证结果

README 为文档改动，不改变运行时代码。阶段 08 沿用阶段 07 的完整回归结果：

- `npm run test:unit -- --run`：通过，21 个测试文件，220 个测试。
- `npm run build`：通过，仅保留 Vite 大 chunk 警告。
- `npm run vis:check -- --port 4182`：通过，3D bridge ready，70 个对象可见。
- `npm run scene:report -- --port 4183`：通过，Spline scene 哈希未变。
- Playwright 页面冒烟：导航、原理动画隐藏 AI 助手、2D 实验台、数据处理报告、知识图谱 fallback 均通过。
- 后端 `node --check` 静态检查：通过。

本阶段额外执行 README 与清理检查，没有新增依赖、没有修改 lockfile、没有运行会改变依赖状态的 `npm install`。

# 未完成 / 风险

- 后端真实 Neo4j 动态接口仍需用户配置 `backend/.env` 并确认安装依赖后才能实测。
- 视觉基线 hash 已变化，但本阶段未更新 baseline。
- 本地 `4181` 预览服务可能仍在监听；README 已在已知限制中说明可使用其他端口。

# 结论

README 已反映当前整合后的实际项目结构和运行方式。本阶段未发现需要清理的已暂存产物，整合分支可以进入最终收拢到 `main` 的步骤。
