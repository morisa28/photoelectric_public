# new（3）开发接续说明

## 1. 文档用途

本文件面向后续继续开发本项目的代理或开发者。下一次对话如果需要继续在 `new（3）` 上开发，先阅读本文件，再按本文件中的入口文件和状态流继续处理，可以减少重复排查时间。

项目目录：

- `/home/just_monika/win_share/new（3）`

本项目基于 `new（2）` 继续迭代得到，并在原有 2D 与 3D 实验台双向同步基础上，升级为“单屏全景 3D 实验空间 + 模型来源候选清单”版本。

## 2. 当前目标达成情况

目前已经完成以下内容：

1. 以 `new（2）` 为基线创建了 `new（3）`
2. 保留 Pinia 作为 2D / 3D 的唯一状态源
3. 将 3D 页面重构为“完整实验空间单屏总览”，无需上下滚动即可看到全貌
4. 将汞灯、光阑、光电管、试验仪、线材拆为独立 3D 器材位并强化状态映射
5. 在 3D 页面内新增器材热点交互，使场景内操作也能回写到 2D
6. 新增 3D 模型来源候选清单，便于后续替换成真实外部模型
7. 将实验流程步骤、器材诊断、报告分析摘要统一纳入 store 派生数据

## 3. 核心架构

当前项目的核心是“Vue 主应用 + iframe 中的 3D 页面 + postMessage 桥接 + Pinia 单一状态源”。

### 3.1 单一状态源

统一状态文件：

- `/home/just_monika/win_share/new（3）/src/stores/experiment.js`

该文件现在是整个实验系统的核心。2D 页面和 3D 页面都必须以这里的数据结构为准，不要再各自维护一套独立状态。

主要状态分为：

1. `params`
   - 电压
   - 频率
   - 光强
   - 材料
   - 模式
2. `apparatus`
   - 光电管盖子
   - 汞灯盖子
   - 汞灯电源
   - 高频线连接
   - 光电管距离
   - 滤色片
   - 光阑
   - 电流量程
   - 电压范围
   - 调零
   - 手动/自动模式
   - 动画开关
   - 光子/电子/电场显示开关
3. `records`
   - 历史实验记录
4. `derived`
   - 由 getter 提供，例如波长、截止电压、光电流、逸出功、截止频率等

如果后续新增器材或实验条件，优先先扩展这里，再去改 2D 和 3D。

### 3.2 2D 页面

2D 实验台文件：

- `/home/just_monika/win_share/new（3）/src/components/VirtualLab.vue`

当前已改为通过 store 读写实验状态，不再以组件内部局部 `ref` 为主状态源。

如果后续发现 2D 和 3D 不同步，优先检查这里是否又新增了局部状态但没有写回 store。

### 3.3 3D 页面承载层

Vue 中负责承载 3D iframe 的文件：

- `/home/just_monika/win_share/new（3）/src/views/PlaceholderView.vue`

此文件现在不是单纯的 iframe 容器，而是 2D 和 3D 之间的同步中枢。

它负责：

1. 将 store 快照发给 iframe
2. 监听 iframe 发回的消息
3. 把 3D 页面的改动写回 store

### 3.4 3D 静态页面

3D 页面入口：

- `/home/just_monika/win_share/new（3）/public/3d/index.html`

相关脚本：

- `/home/just_monika/win_share/new（3）/public/3d/js/simulation.js`
- `/home/just_monika/win_share/new（3）/public/3d/js/controls.js`
- `/home/just_monika/win_share/new（3）/public/3d/js/three-scene.js`
- `/home/just_monika/win_share/new（3）/public/3d/js/charts.js`

样式文件：

- `/home/just_monika/win_share/new（3）/public/3d/css/style.css`

说明：

原始 `photoelectric-simulation` 使用了 `three.js` CDN 方案，但当前 `new（3）` 中没有继续依赖外网 CDN，也没有将 `three` 作为运行时强依赖接入。现版本 3D 采用的是“本地全景 3D 风格场景 + 透视渲染 + 粒子动画 + 图表 + 模型来源候选面板”的方案，可在现有环境稳定运行，并支持同步。

新增参考文档：

- `/home/just_monika/win_share/new（3）/说明文档4-3D模型来源清单.md`
- `/home/just_monika/win_share/new（3）/说明文档5-本地3D模型接入说明.md`

本地模型接入层相关文件：

- `/home/just_monika/win_share/new（3）/public/3d/models/manifest.json`
- `/home/just_monika/win_share/new（3）/public/3d/js/model-loader.js`

## 4. 当前消息协议

2D 父页面发往 3D iframe：

1. `sync-state`
   - 发送完整状态快照

3D iframe 发往 2D 父页面：

1. `ready`
   - iframe 已准备完成
2. `request-sync`
   - 请求父页面下发最新状态
3. `update-state`
   - 发送参数或器材状态变更
4. `record-data`
   - 请求记录当前数据
5. `reset-experiment`
   - 请求重置实验
6. `start-auto-scan`
   - 请求开始自动扫描
7. `stop-auto-scan`
   - 请求停止自动扫描

如果后续还要同步更多内容，不要绕开这个协议直接做 DOM 注入或 URL 参数同步，继续沿用这里。

## 5. 当前已实现的同步范围

2D 和 3D 目前已同步以下内容：

1. 汞灯电源开关
2. 汞灯盖子状态
3. 光电管盖子状态
4. 高频输入线连接状态
5. 光电管距离
6. 滤色片
7. 光阑
8. 阴极材料
9. 电压范围
10. 外加电压
11. 电流量程
12. 调零数值
13. 手动/自动模式
14. 动画运行状态
15. 光子/电子/电场显示开关
16. 历史记录数量和图表数据
17. 实时光电流、截止电压、波长等派生值
18. 实验流程步骤状态
19. 器材诊断结果
20. 普朗克常量、逸出功、截止频率与相对误差摘要
21. 同步版本号、最后更新时间与来源
22. 自动扫描进度与完成状态

## 6. 这次补齐了哪些 3D 缺失映射与空间能力

相对于旧版 3D 页面，当前已补齐：

1. 单屏完整实验空间布局，无需页面上下滚动即可看到全貌
2. 汞灯、光阑、光电管、试验仪、线材的独立 3D 器材位
3. 测试仪显示区与实时数据卡联动
4. 高频输入线状态映射
5. 盖子开合状态映射
6. 光电管距离变化映射
7. 滤色片波长引起的光束颜色变化
8. 光阑大小引起的光束宽度变化
9. 电场显示与正负电压方向表现
10. 自动扫描与记录动作接入
11. 场景热点点击交互，可直接回写 2D 状态
12. 模型来源候选面板，便于后续替换真实 3D 模型
13. 本地模型清单加载、模型目录规范和程序化回退机制

## 7. 后续继续开发时的优先入口

如果下一次对话继续开发，建议按下面顺序进入：

1. 先读本文件
2. 读 `/src/stores/experiment.js`
3. 读 `/src/views/PlaceholderView.vue`
4. 再读 `/src/components/VirtualLab.vue`
5. 如果是 3D 问题，再读 `/public/3d/js/*.js`
6. 如果是模型接入问题，再读 `/public/3d/models/manifest.json` 与 `/public/3d/js/model-loader.js`

这几个文件已经覆盖当前大部分关键逻辑。

## 8. 已验证事项

已完成验证：

1. `npm run build` 成功
2. `dist/3d` 中已包含静态页面、脚本、样式与图表依赖
3. `public/3d/js` 下的脚本已通过 `node --check`
4. 已启动过 `vite dev server`

## 9. 未做的深度验证

以下内容目前没有在浏览器自动化里做完整验证，只做了代码和构建层验证：

1. 用户在 2D 操作后立即切换到 3D 的全部边界行为
2. 用户在 3D 快速连续拖动、切换、扫描时的全部同步边界
3. 手机端完整触摸交互
4. 3D 视觉精度是否完全满足真实实验器材风格要求

如果后续用户要求“继续优化 3D 拟真度”或“继续补交互细节”，优先从这里着手。

## 10. 后续开发建议

若后续继续迭代，建议优先做以下几项：

1. 为 2D/3D 同步增加防抖或版本号，避免极端操作下状态回写抖动
2. 如需更进一步，可把当前 revision 机制扩展为冲突检测而不只是去抖
3. 为 3D 页面补更细的器材标签和实验步骤提示
4. 将历史记录图表统一抽成共享计算逻辑，避免 2D/3D 各写一份
5. 如需更强视觉拟真，再考虑重新引入本地化 `three.js` 资源而不是 CDN

## 11. 一句话接续说明

下次如果继续开发，请直接以 `new（3）` 为唯一工作目录，并默认：`experiment.js` 是唯一状态源，`PlaceholderView.vue` 是 2D/3D 桥接入口，`public/3d/js` 是 3D 侧同步实现，`说明文档4-3D模型来源清单.md` 是器材模型替换入口。
