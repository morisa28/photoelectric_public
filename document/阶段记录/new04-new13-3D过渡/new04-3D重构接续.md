# new（4）3D重构接续文档

## 1. 本阶段结论

本阶段已经不再沿用 `new（3）` 中的 `iframe + public/3d` 作为 3D 主实现。

`new（4）` 当前的 3D 页面已经改为：

1. Vue 页面内直接承载 3D 场景
2. `Pinia` 仍然是 2D 与 3D 的唯一实验状态源
3. 3D 教室、实验台、器材节点、相机控制、微观浮窗全部走模块化实现

后续继续开发时，请直接以 `new（4）` 为唯一工作目录。

## 2. 当前最重要的事实

接手时先记住下面几点：

1. `src/stores/experiment.js` 仍然是唯一状态源，2D/3D 都直接读写这里
2. 3D 页面入口已经改为 `src/views/ThreeDView.vue`
3. 3D 场景 UI 和交互桥接在 `src/components/three-lab/PhotoelectricLabScene.vue`
4. Three 场景构建与对象节点逻辑在 `src/three-lab/createPhotoelectricLabScene.js`
5. 相机轨道控制在 `src/three-lab/createOrbitRig.js`
6. 微观过程浮窗在 `src/components/three-lab/MicroProcessWindow.vue`
7. 3D 参数选项、交互提示、相机限制在 `src/three-lab/config.js`
8. 路由入口已从 `PlaceholderView.vue` 切换到 `ThreeDView.vue`

## 3. 当前已经完成的内容

### 3.1 3D 场景

已经具备一个完整的教室式实验空间，包含：

1. 教室地面、墙面、黑板、柜体、窗户、照明和环境布置
2. 中央课桌与实验台
3. 汞灯、滤色片托盘、光阑、光电管、控制面板、测试仪、线缆等器材占位模型
4. 光束、电场线、光子和电子粒子效果

### 3.2 视角控制

已经实现：

1. 鼠标拖动旋转视角
2. 滚轮缩放
3. 方向键和小键盘 `2/4/6/8` 辅助旋转
4. 最小/最大缩放距离限制
5. 俯仰角限制

### 3.3 实验交互

当前场景内可以直接交互并回写实验参数的对象包括：

1. 汞灯电源
2. 汞灯盖子
3. 光电管盖子
4. 光阑切换
5. 滤色片切换
6. 高频输入线
7. 外加电压旋钮
8. 电流调零旋钮

这些交互都不是单纯视觉反馈，而是直接调用 store action 更新统一状态。

### 3.4 微观过程浮窗

已经实现：

1. 打开
2. 拖动
3. 关闭
4. 与实验条件联动

当前逻辑是：

1. 条件不足时，只显示静态入射光，不进入电子逸出状态
2. 条件满足后，电子开始逸出
3. 动画速度、粒子频率和强度会随频率、光强和光电流变化

## 4. 当前结构与替换正式模型的方式

这次重构的核心原则是“显示模型”和“交互逻辑”解耦。

当前器材虽然仍是简化几何体，但都已经分成独立对象节点和槽位：

1. `lamp-model-slot`
2. `aperture-model-slot`
3. `photocell-model-slot`
4. `control-panel-model-slot`
5. `meter-model-slot`

后续如果要替换为正式 `GLTF / GLB` 模型，推荐方式是：

1. 保留当前交互 key，不改 `PhotoelectricLabScene.vue` 中的动作映射
2. 在 `createPhotoelectricLabScene.js` 中把简化几何体替换为真实模型挂载
3. 让真实模型的子节点去承接现在这些交互热点和状态变化

不要回退到“模型写死 + 逻辑绑死在材质或临时 mesh 上”的方式。

## 5. 与 new（3）相比的关键变化

`new（4）` 和 `new（3）` 的最大差异如下：

1. 不再以 `src/views/PlaceholderView.vue` 作为 3D 主入口
2. 不再以 `iframe` 和消息协议作为 2D/3D 主同步方式
3. 不再把 3D 主逻辑放在 `public/3d/js/*.js`
4. 3D 现在直接运行在 Vue 组件树内
5. 3D 与 2D 同步现在直接依赖同一个 Pinia store

因此后续开发时，优先看 `src/components/three-lab` 和 `src/three-lab`，不是旧的 `public/3d`。

## 6. 当前依赖与运行注意事项

当前项目在运行层面有两个注意点：

1. `package.json` 的 `dev/build/preview` 脚本已经改为直接调用 `node node_modules/...`，这是为了规避当前共享盘环境下 `.bin` 包装脚本不可执行的问题
2. Three.js 当前通过 `src/three-lab/loadThreeModule.js` 在运行时从 `https://esm.sh/three@0.180.0` 动态加载

这意味着：

1. 当前版本可以构建
2. 当前版本可以继续开发
3. 但如果要彻底离线运行，下一阶段需要把 Three.js 改为本地 vendor 或稳定本地依赖

## 7. 最适合继续做的方向

如果下一阶段继续推进，建议优先级如下：

1. 将 Three.js 改为完全本地依赖，去掉运行时远程加载
2. 为器材接入正式 `GLTF / GLB` 模型
3. 给正式模型补充节点命名规范和挂载规则
4. 把微观动画从 2D canvas 提升为更完整的粒子/轨迹系统
5. 增加更多实验器材和可交互组件
6. 做真实浏览器联调和手工验收

## 8. 推荐阅读顺序

后续开发建议按下面顺序阅读：

1. `src/views/ThreeDView.vue`
2. `src/components/three-lab/PhotoelectricLabScene.vue`
3. `src/three-lab/createPhotoelectricLabScene.js`
4. `src/three-lab/createOrbitRig.js`
5. `src/components/three-lab/MicroProcessWindow.vue`
6. `src/stores/experiment.js`

## 9. 本阶段验证情况

本阶段结束前已确认：

1. `npm run build` 成功
2. 3D 页面入口已切换到 Vue 内部实现
3. 2D 与 3D 使用统一 Pinia 状态源
4. 旋钮、盖子、光阑、线缆等交互已能真实影响实验参数
5. 微观过程浮窗已可打开、拖动、关闭并跟实验条件联动

## 10. 一句话交接

`new（4）` 已经从“旧的 3D 外挂页面”升级为“Vue 内部直接驱动的教室实验场景版 3D 页面”，下一位开发者最适合直接继续做“正式模型接入 + Three 本地化 + 进一步细化实验交互”。
