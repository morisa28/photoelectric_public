# new(8) Spline 接入与 3D 清理收口任务总结

## 1. 本轮任务目标

本轮任务的核心目标有三件事：

1. 将 `new(8)` 中原有的网页端 3D 场景替换为 Spline 导出的 `scene.splinecode`
2. 清理原先遗留的 Three.js 3D 页面、模型资源与桥接代码
3. 将项目从对 `new（6）/node_modules` 的共享依赖，收口为 `new(8)` 自己的本地依赖结构

本轮任务完成后，`new(8)` 的 3D 页面已经不再走旧的 Three.js 场景链路，而是改为直接加载 Spline 网页场景。

---

## 2. 本轮完成的主要改动

### 2.1 Spline 页面正式接入

当前 `/3d-view` 页面已改为加载 Spline 场景，核心文件如下：

- `src/views/ThreeDView.vue`
- `src/components/SplineScene.vue`
- `resource/scene.splinecode`

接入方式为：

1. 页面保留独立 3D 路由 `/3d-view`
2. 3D 页面内部不再挂载 `PhotoelectricLabScene.vue`
3. 改为通过 `spline-viewer` Web Component 直接加载打包后的 `.splinecode` 资源
4. `scene.splinecode` 由 Vite 当作构建资源处理，并输出到 `dist/assets/*.splinecode`

这样做的直接结果是：

1. 3D 页面结构更简单
2. 不再依赖原本的 Three.js 场景初始化流程
3. 后续如果继续由 Spline 维护网页端 3D，前端侧只需维护页面壳和资源入口

---

## 3. 已移除的旧 3D 链路

本轮已将原先的 Three.js 3D 链路整体下线。

### 3.1 删除的代码模块

已删除以下旧 3D 代码：

- `src/three-lab/`
- `src/components/three-lab/`
- `src/composables/usePhotoelectricSceneController.js`
- `src/views/PlaceholderView.vue`

这些文件原本负责：

1. Three.js 场景初始化
2. 相机控制
3. GLB / HDR 加载
4. 微观过程窗口
5. 旧版 iframe 桥接页
6. 模型交互与视觉状态同步

现在这些职责已不再由当前网页端承担。

### 3.2 删除的旧模型资源

已删除以下旧 Three.js 模型与环境资源：

- `resource/all4.glb`
- `resource/base.glb`
- `resource/env.hdr`
- `resource/lab_bench.glb`
- `resource/light.glb`
- `resource/table.glb`
- `resource/tube.glb`

当前 `resource/` 目录只保留：

- `resource/scene.splinecode`

这意味着当前 3D 页面只依赖 Spline 场景文件，不再依赖旧的本地模型组合。

---

## 4. 已收掉的状态桥接遗留

原先 store 中为了兼容旧版 iframe / Three.js 交互保留过一批桥接逻辑。

本轮已从 `src/stores/experiment.js` 中移除：

1. `bridgeSnapshot`
2. `applyStatePatch`
3. `MODEL_MANIFEST` 的引用链路

这些内容原本服务于：

1. 旧 iframe 页面同步
2. 旧 Three.js 页面回写状态
3. 调试或外部桥接快照

由于当前网页端 3D 已经改为 Spline 场景，这批桥接残留继续保留只会增加维护成本，因此已一并收口。

---

## 5. 依赖结构收口结果

本轮前期为了快速完成接入，曾短暂复用过：

- `../new（6）/node_modules`

后续已继续完成收口，当前状态如下：

1. `new(8)/node_modules` 已恢复为本地真实目录
2. `package-lock.json` 已重新生成
3. `package.json` 的 `dev / build / preview` 脚本已恢复为本地 `node_modules/vite`
4. `vite.config.js` 已恢复为标准包导入：
   - `import { defineConfig } from 'vite'`
   - `import vue from '@vitejs/plugin-vue'`

因此，`new(8)` 现在已经不再依赖 `new（6）` 的运行时依赖目录。

---

## 6. 当前项目状态

截至本轮结束，`new(8)` 的网页端结构可以理解为：

### 6.1 保留的主链路

1. 2D 首页与实验流程仍然保留
2. 路由结构仍然保留 `/` 和 `/3d-view`
3. `/3d-view` 仍然是独立网页端入口
4. 3D 页面已改为 Spline 版本

### 6.2 当前 3D 页面的职责

当前 3D 页面只负责：

1. 提供单独的网页端 3D 入口
2. 加载 `scene.splinecode`
3. 提供返回 2D 页面的入口按钮

它不再负责：

1. Three.js 场景装配
2. 本地模型拼装
3. 旧版实验器材交互映射
4. store -> Three 实时桥接
5. iframe 消息通信

---

## 7. 验证结果

本轮已完成以下验证：

### 7.1 构建验证

已执行：

```sh
npm run build
```

结果：

1. 构建通过
2. `dist/assets/scene-*.splinecode` 已正常产出
3. 页面 JS / CSS 资源正常输出

### 7.2 预览验证

已启动本地预览服务：

```sh
npm run preview -- --host 0.0.0.0 --port 4300
```

当前访问地址：

- `http://localhost:4300/`

---

## 8. 本轮任务结论

如果把本轮任务定义为：

“将 `new(8)` 原有网页端 3D 替换为 Spline 场景，并清理旧 Three.js 3D 链路，同时把项目依赖结构重新收口为本地独立项目”

那么本轮任务已经完成。

当前结果可以概括为：

1. Spline 已接入
2. 旧 3D 代码已移除
3. 旧 3D 模型资源已清理
4. 旧桥接逻辑已收掉
5. `new(8)` 已恢复为独立可构建、可预览的本地项目

本轮到此结束。
