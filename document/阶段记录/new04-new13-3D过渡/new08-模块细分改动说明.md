# new(8) 模块细分改动说明

## 1. 文档目的

本文档用于说明上一轮对 `new(8)` 所做的 3D 结构整理工作。

这一轮工作的目标不是继续改主功能，而是先把 3D 场景相关模块进一步明确，避免后续继续把节点绑定、视觉状态、交互注册、粒子逻辑、屏幕逻辑全部堆在同一个场景文件中。

---

## 2. 本轮改动范围

本轮主要处理的是 `src/three-lab` 下的 3D 场景组织方式，核心对象是：

- `src/three-lab/createPhotoelectricLabScene.js`
- `src/three-lab/photoelectric-lab/*.js`

本轮没有进入以下内容的功能重做：

1. 相机控制规则重做
2. 按钮和旋钮手感重做
3. 滤光片真实交互重做
4. 汞灯发光联动表现重做
5. 双屏显示逻辑优化
6. store 侧实验规则修改

换句话说，这一轮是“结构细分”，不是“功能升级”。

---

## 3. 改动前的问题

在拆分前，`src/three-lab/createPhotoelectricLabScene.js` 同时承担了过多职责，包括：

1. Three.js 基础对象初始化
2. HDR 与 GLB 资源加载
3. 模型节点绑定
4. 按钮与旋钮交互注册
5. 光束、灯光、线缆等动态对象创建
6. 屏幕 overlay 创建
7. 视觉状态更新
8. 粒子系统更新
9. 指针事件处理
10. 资源释放

这样会导致两个直接问题：

1. 后续开发者很难判断某个问题应该改哪一层
2. 任何一个局部功能调整，都容易再次把主场景文件改得更重

---

## 4. 拆分后的结构

### 4.1 主入口保留

以下文件仍然是 3D 场景总装配入口：

- `src/three-lab/createPhotoelectricLabScene.js`

它现在主要负责：

1. 初始化 renderer、scene、camera、light
2. 加载 GLB 与 HDR
3. 调用各个子模块完成装配
4. 维护场景 runtime
5. 对外暴露 `applyState`、交互事件和 `dispose`

它不再继续承载全部细节逻辑。

### 4.2 新增细分模块

#### `src/three-lab/photoelectric-lab/constants.js`

职责：

1. 统一存放实验台 3D 相关常量
2. 提供模型资源与环境资源入口
3. 提供按钮交互映射表
4. 提供滤光片、量程、调零旋钮等角度映射

这样后续如果要改交互 key 或旋钮角度，不需要再去主场景文件搜索散落常量。

#### `src/three-lab/photoelectric-lab/objectUtils.js`

职责：

1. 交互元数据挂载
2. 电压 / 电流显示格式化
3. 按轴旋转节点
4. 读取模型局部中心与局部尺寸
5. 统一材质释放

这是把原来散在场景文件中的通用工具函数收口成公共工具层。

#### `src/three-lab/photoelectric-lab/bindModelNodes.js`

职责：

1. 集中绑定 GLB 中的关键模型节点
2. 统一维护房间、实验台、汞灯、光电管、滤光片、线缆、屏幕等命名入口
3. 在入口阶段检查关键节点是否缺失

后续如果 GLB 节点命名变化，可以优先在这里集中调整。

#### `src/three-lab/photoelectric-lab/registerLabInteractions.js`

职责：

1. 批量注册按钮、旋钮、滤光片转盘、接线柱等交互对象
2. 维护 `interactiveObjects`
3. 维护 `interactiveMap`
4. 处理材质克隆、按钮按压深度等交互基础信息

这样 Raycaster 命中系统和模型节点绑定逻辑就不再混在一起。

#### `src/three-lab/photoelectric-lab/createLabDynamicObjects.js`

职责：

1. 创建程序补充的光束对象
2. 创建汞灯发光球
3. 创建汞灯聚光灯
4. 保存线缆默认姿态

这个模块对应的是“GLB 之外、由程序补建的动态表现对象”。

#### `src/three-lab/photoelectric-lab/createLabScreenOverlays.js`

职责：

1. 为模型中的左右显示屏创建 overlay
2. 将左右屏与主场景装配流程解耦

后续如果要重做双屏显示样式，可以优先在这层继续拆。

#### `src/three-lab/photoelectric-lab/createLabVisualState.js`

职责：

1. 统一收口各种视觉刷新逻辑
2. 更新汞灯、光束、光斑和按钮发光
3. 更新滤光片可见性与材质状态
4. 更新线缆与接线柱视觉状态
5. 更新旋钮角度
6. 更新双屏文本

它的定位是“把归一化状态映射成 3D 视觉表现”。

#### `src/three-lab/photoelectric-lab/createLabParticleSystem.js`

职责：

1. 创建光子 / 光电子粒子
2. 管理粒子生命周期
3. 在每帧更新粒子位置与回收逻辑

它的定位是“纯表现层粒子系统”，不参与业务数值计算。

---

## 5. 调用链变化

拆分后，调用链可以理解为：

1. `ThreeDView.vue`
2. `PhotoelectricLabScene.vue`
3. `usePhotoelectricSceneController.js`
4. `createPhotoelectricLabScene.js`
5. `photoelectric-lab/*.js`
6. `scene/snapshot.js`
7. `stores/experiment.js`

其中：

- `usePhotoelectricSceneController.js` 仍然负责把 store 状态推给 Three 场景
- `scene/snapshot.js` 仍然负责把外部状态归一化
- `photoelectric-lab/*.js` 负责 3D 实验台内部细分职责

这意味着后续开发时可以先判断问题落在哪一层，再决定修改位置。

---

## 6. 本轮实际收益

完成这次拆分后，项目获得的直接收益包括：

1. 3D 主场景文件职责下降，后续可读性更高
2. 模型节点绑定位置固定，方便后续查节点名
3. 视觉状态更新逻辑集中，方便后续替换表现
4. 粒子系统不再和主状态更新逻辑混写
5. 屏幕 overlay 成为单独模块，后续重做显示更容易
6. 常量、工具、动态对象、交互注册边界已经明确

这些收益主要服务于后续迭代，不是直接的用户可见功能变化。

---

## 7. 本轮未改变的行为

为了保证“先细分再暂停”，本轮刻意保持以下行为不变：

1. 页面入口不变
2. 2D -> 3D 跳转方式不变
3. store 到 Three 的状态同步方式不变
4. 现有按钮点击 key 不变
5. 现有旋钮拖动 key 不变
6. 现有粒子触发条件不变
7. 现有屏幕显示数据来源不变

因此，这一轮更接近“重排结构”，而不是“改规则”。

---

## 8. 验证情况

本轮完成后，已执行一次构建校验：

```sh
node node_modules/vite/bin/vite.js build --outDir /tmp/new8-build
```

结果：构建通过。

说明当前拆分后的模块引用关系和打包结果是正常的。

---

## 9. 建议的后续承接方式

下一轮如果继续只做结构细分，建议优先处理：

1. 将 `usePhotoelectricSceneController.js` 中的动作映射再独立成 action 模块
2. 将 `createPhotoelectricLabScene.js` 中的事件处理再独立成 interaction 模块
3. 将 `createOrbitRig.js` 中的输入规则和边界规则进一步拆开

下一轮如果开始进入功能修改，建议优先顺序为：

1. 相机控制
2. 按钮 / 旋钮交互
3. 汞灯发光联动
4. 双屏数值显示

---

## 10. 当前结论

上一轮改动已经完成了“对 3D 实验台进行进一步细分、明确模块边界”的目标。

当前项目已经从“一个主场景文件承载过多职责”的状态，进入“主入口装配 + 子模块分工”的状态。

这为后续继续修补 `new(8)` 的 3D 界面提供了更清晰的落点。
