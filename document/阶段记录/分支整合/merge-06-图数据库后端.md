# 阶段目标

本阶段从 `origin/2d-logic` 合入知识图谱后端，并把它整理为独立 `backend/` 工程。

目标是让知识图谱页面具备可选的 Express + Neo4j 动态图谱能力，同时不污染前端根项目、不提交真实配置、不阻塞前端本地 fallback。

# 来源分支与参考内容

- 主要来源：`origin/2d-logic`
- 参考后端入口：`backend/server.js`
- 参考路由：`backend/api/knowledgeGraphAPI.js`
- 参考 Neo4j 配置：`backend/neo4j/config/neo4j.config.js`
- 参考 Cypher 查询：`backend/neo4j/queries/cypherQueries.js`
- 参考初始化脚本：`backend/neo4j/scripts/initKnowledgeGraph.js`
- 参考测试服务器：`backend/test-server.js`

已确认 `origin/2d-logic` 没有独立 `backend/package.json`，因此本阶段新增独立后端包文件。

# 实际改动文件

- `backend/server.js`
- `backend/api/knowledgeGraphAPI.js`
- `backend/neo4j/config/neo4j.config.js`
- `backend/neo4j/queries/cypherQueries.js`
- `backend/neo4j/scripts/initKnowledgeGraph.js`
- `backend/test-server.js`
- `backend/package.json`
- `backend/.env.example`
- `backend/.gitignore`
- `document/阶段记录/分支整合/merge-06-图数据库后端.md`

# 涉及的关键组件、store、service、feature

- `backend/server.js`：Express 服务入口，只挂载 `/api/knowledge-graph` 和 `/health`。
- `backend/api/knowledgeGraphAPI.js`：知识图谱 API 路由，提供 `/graph`、`/node/:id`、`/nodes/category/:category`、`/search`、`/stats`、`/learning-path/:id`、`/path/:from/:to`。
- `backend/neo4j/config/neo4j.config.js`：Neo4j driver 配置和连接生命周期。
- `backend/neo4j/queries/cypherQueries.js`：图谱节点、关系、学习路径、搜索和统计查询。
- `backend/neo4j/scripts/initKnowledgeGraph.js`：初始化知识图谱节点与关系。
- `backend/package.json`：后端独立依赖和脚本，不影响前端根 `package.json`。
- `src/services/knowledgeGraphService.js`：未修改，但已核对后端接口与其 `/api/knowledge-graph` 路径对齐。

# 明确保留的 animation-demo 底层内容

本阶段没有修改以下前端和实验核心文件：

- `package.json`
- `package-lock.json`
- `src/services/knowledgeGraphService.js`
- `src/components/PhotoelectricEffectGraph.vue`
- `src/stores/experiment.js`
- `src/features/experiment/photoelectricModel.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`
- `src/stores/principleAnimation.js`
- `src/features/photoelectric-animation/*`
- `src/components/PrincipleAnimationSection.vue`
- `src/components/SplineScene.vue`
- `resource/scene.splinecode`
- `vendor/spline-runtime/`

后端依赖只放在 `backend/package.json`，没有写入前端根项目依赖。

# 本阶段没有合入的内容及原因

- 未合入 `origin/2d-logic` 的 `backend/api/aiAPI.js`：实施文档要求后端 AI 接口不合入，AI 助手继续走阶段 02 的前端 AI 路线。
- 未复制 `origin/2d-logic` 根目录 `.env`：该文件可能含本地连接密码或敏感配置，本阶段只新增 `backend/.env.example`。
- 未新增根目录 `.env`：后端配置应放在 `backend/.env`，并由 `backend/.gitignore` 忽略。
- 未修改 `src/services/knowledgeGraphService.js`：阶段 01 已实现后端失败时 fallback，本阶段保持前端服务稳定。
- 未运行 `npm install` 生成 `backend/package-lock.json`：新增依赖和 lockfile 会影响依赖状态，按项目规则需要用户确认；本阶段只提交独立 `backend/package.json`。

# 关键适配

- `backend/server.js` 改为加载 `backend/.env`，不读取前端根 `.env`。
- `backend/server.js` 删除 `aiAPI` import 和 `/api/ai` 挂载。
- `backend/neo4j/config/neo4j.config.js` 改为读取 `backend/.env`，默认密码为空字符串，并且不打印密码。
- `backend/api/knowledgeGraphAPI.js` 的搜索接口兼容前端使用的 `keyword` 查询参数，同时继续兼容 `q`。
- `backend/api/knowledgeGraphAPI.js` 的最短路径接口返回节点名称、分类、描述、图标和重要性，便于前端路径展示。

# 验证命令与结果

后端语法检查：

```bash
node --check backend/server.js
node --check backend/api/knowledgeGraphAPI.js
node --check backend/neo4j/config/neo4j.config.js
node --check backend/neo4j/queries/cypherQueries.js
node --check backend/neo4j/scripts/initKnowledgeGraph.js
node --check backend/test-server.js
```

结果：通过。

JSON 与空白检查：

```bash
node -e "JSON.parse(require('fs').readFileSync('backend/package.json','utf8')); JSON.parse(require('fs').readFileSync('package.json','utf8')); console.log('json ok')"
git diff --check -- backend
```

结果：通过。

敏感项检查：

```bash
rg -n "aiAPI|/api/ai|sk-[A-Za-z0-9]|DEEPSEEK|OPENAI|API_KEY|PASSWORD=." backend
```

结果：无输出。未发现后端 AI 路由、硬编码 Key 或非空密码样例。

前端无后端 fallback 验证：

```bash
node --input-type=module -e "<Playwright 知识图谱 fallback 检查脚本>"
```

结果：通过。复用阶段 05 遗留的 `http://127.0.0.1:4181/` 前端预览，未启动知识图谱后端。检查到：

- 页面标题为“光电效应实验模拟仿真平台”。
- `.photoelectric-effect-container` 可见。
- `.network-container` 可见。
- `.filter-panel` 可见。
- 筛选面板显示本地类别和 `显示: 25 / 25 节点`。
- `.error-overlay` 不可见。
- 控制台有 3 条 `net::ERR_CONNECTION_REFUSED`，均为未启动后端时预期的知识图谱请求。
- 截图保存到 `/tmp/stage06-knowledge-graph-fallback.png`，未提交。

# 测试失败或未执行项及原因

- 未执行 `cd backend && npm install`：会新增依赖锁文件和安装目录，按项目规则需用户确认后再做。
- 未执行 `npm run start` 真实后端启动：当前未安装 `backend/` 依赖，且本地没有确认可用 Neo4j 服务。
- 未执行 `npm run init:graph`：本地 Neo4j 可用性未确认，避免对用户数据库写入或清空数据。
- 未完成“启动后端且 Neo4j 可用”的动态图谱实测；后端代码已合入，前端 fallback 已验证。

# 发现的问题与处理方式

- 来源后端读取根 `.env`。处理方式：改为只读取 `backend/.env`，并提交 `backend/.env.example` 和 `backend/.gitignore`。
- 来源后端挂载 `/api/ai`。处理方式：删除该 import 和路由，避免引入后端 AI 路线。
- 来源后端搜索接口只接收 `q`，前端当前调用 `keyword`。处理方式：后端同时兼容 `q` 与 `keyword`。
- 来源最短路径接口只返回节点 id。处理方式：后端路径查询返回节点基础信息，便于前端展示名称和分类。

# 下一阶段注意事项

- 阶段 07 做底层整合校验时，应继续验证无后端知识图谱 fallback。
- 如需要真实 Neo4j 动态接口，请先在 `backend/.env` 填写本地配置，再由用户确认后执行 `cd backend && npm install` 和 `npm run init:graph`。
- 不要把 `backend/.env`、Neo4j 密码、token 或任何真实连接配置提交到仓库。
