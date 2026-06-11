# 阶段目标

本阶段从 `origin/2d-logic` 迁移新版知识图谱前端体验，并按当前主线要求补齐无后端本地 fallback。

目标能力包括：加载状态、错误重试、分类筛选、节点搜索、学习路径模式、路径高亮、学习建议、主题切换、刷新、清除高亮、重置视图和科学家图片展示。

# 来源分支与参考内容

- 主要来源：`origin/2d-logic`
- 参考组件：`src/components/PhotoelectricEffectGraph.vue`
- 参考资源：`src/assets/scientists/*`
- 服务层参考：`src/services/neo4jService.js`

本阶段没有整支 merge `2d-logic`，没有合入 `2d-logic` 的 `HomeView.vue`、根 `package.json` 后端依赖或后端目录。

# 实际改动文件

- `src/components/PhotoelectricEffectGraph.vue`
- `src/services/knowledgeGraphService.js`
- `src/assets/scientists/einstein.webp`
- `src/assets/scientists/planck.jpeg`
- `src/assets/scientists/hertz.webp`
- `src/assets/scientists/lenard.webp`
- `src/assets/scientists/millikan.webp`
- `src/assets/scientists/de_broglie.webp`
- `src/assets/scientists/compton.webp`
- `document/阶段记录/分支整合/merge-01-新版知识图谱前端.md`

# 涉及的关键组件、store、service、feature

- `PhotoelectricEffectGraph.vue`：知识图谱页面组件，升级为新版筛选、搜索、主题和学习路径交互。
- `knowledgeGraphService.js`：新增知识图谱服务层，统一处理后端请求和本地数据 fallback。
- `knowledgeGraphData.js`：保留当前主线本地知识图谱数据，服务层读取它生成本地图谱、节点详情、搜索结果和学习路径。

# 明确保留的 animation-demo 底层内容

本阶段未修改以下底层红线内容：

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
- `resource/scene.splinecode`
- `vendor/spline-runtime/`

主页总导航、原理动画入口和 3D 入口保持 `animation-demo` 当前结构。

# 本阶段没有合入的内容及原因

- 未合入 `2d-logic` 的 `backend/*`：后端属于阶段 06。
- 未合入 `2d-logic` 的后端 AI API：扩展文档明确要求不要合入该路线。
- 未合入 `2d-logic` 的根 `package.json` 后端依赖：后端依赖后续应放入独立 `backend/package.json`。
- 未删除 `src/data/knowledgeGraphData.js`：虽然 `2d-logic` 删除了该文件，但当前目标要求后端不可用时必须本地 fallback。

# 验证命令与结果

```bash
npm run test:unit -- --run
```

结果：通过。12 个测试文件通过，157 个测试通过。

```bash
npm run build
```

结果：通过。Vite 仍提示大 chunk 警告，但构建成功。

```bash
npm run vis:check
```

结果：命令执行成功，生成首页与 3D 视觉矩阵。因为知识图谱 UI 已升级，`home-desktop.png` 和 `home-mobile.png` 与旧 baseline 哈希不同；3D 截图也因运行时状态和截图时机产生哈希差异，脚本未失败。

已查看 `artifacts/visual/current/home-desktop.png`，确认：

- 首页顶部导航仍包含知识图谱、虚拟实验台、原理动画、数据处理和 3D 入口。
- 未启动知识图谱后端时，页面显示本地图谱。
- 筛选面板、搜索入口、主题切换按钮和详情面板可见。
- 右侧详情面板可显示本地节点详情与学习路径建议。

# 测试失败或未执行项及原因

- 本阶段没有失败的自动化命令。
- 未启动 Neo4j 后端进行动态图谱实测；后端属于阶段 06，本阶段验证的是前端 fallback。

# 发现的问题与处理方式

- `2d-logic` 原组件使用硬编码后端地址和后端 AI 学习路径接口；已改为 `knowledgeGraphService.js`。
- `knowledgeGraphService.js` 使用 `VITE_KNOWLEDGE_GRAPH_API_BASE`，默认值为 `http://localhost:3000/api/knowledge-graph`，请求失败时回退到本地 `knowledgeGraphData.js`。
- `2d-logic` 原组件包含全局 `html/body` 样式和全局亮色主题类；已收束为组件根元素的局部 `light-theme` 类。
- 初次视觉检查时右侧默认详情加载偏晚；已改为图谱初始化后立即渲染首个节点详情。

# 下一阶段注意事项

- 阶段 02 合入 AI 助手时，不要覆盖本阶段知识图谱服务层。
- 后端知识图谱在阶段 06 合入时，应对齐 `knowledgeGraphService.js` 的路径：`/graph`、`/node/:id`、`/learning-path/:id`、`/path/:from/:to`。
- 后续 README 收口时需要说明本地数据模式与后端动态图谱模式。
