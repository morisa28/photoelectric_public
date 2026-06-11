# new(7) 3D界面修补开发指南

## 1. 目的

本文档面向后续继续修补 `new(7)` 3D 界面的开发者，目标是把当前项目的：

1. 页面入口
2. 状态入口
3. Three.js 场景入口
4. 模型节点入口
5. 交互入口
6. 后续应修补的重点问题

统一整理清楚，避免后续继续“在场景文件里堆逻辑”。

本文档重点覆盖以下用户要求：

1. 场景光照过暗，需要补光。
2. 初始镜头要正对实验台全貌，滚轮缩放，中键旋转，`W/S/A/D` 平滑平移，不能超出房间。
3. 重做按钮、旋钮、滤光片、接线柱等交互。
4. 重做汞灯柱体发光状态与汞灯电源联动。
5. 电流、电压分别显示在两块模型显示屏上。
6. 给出明确的结构、接口位置和实现方案，方便继续迭代。

---

## 2. 当前项目结构与入口

### 2.1 页面入口

- 2D 主页面路由：`src/router/index.js`
  - `/` -> `src/views/HomeView.vue`
- 3D 页面路由：`src/router/index.js`
  - `/3d-view` -> `src/views/ThreeDView.vue`

### 2.2 2D -> 3D 切换入口

- 文件：`src/views/HomeView.vue`
- 入口按钮：顶部导航按钮“进入3D实验台”
- 跳转方式：`router.push('/3d-view')`

这部分当前已经符合“2D 与 3D 分开，不同屏”的要求。

### 2.3 3D 页面装配入口

- 页面容器：`src/views/ThreeDView.vue`
- 3D 组件：`src/components/three-lab/PhotoelectricLabScene.vue`

`ThreeDView.vue` 只负责包一层页面和返回 2D 按钮，不负责任何 Three 逻辑。

### 2.4 3D 控制器入口

- 文件：`src/composables/usePhotoelectricSceneController.js`

职责：

1. 从 Pinia 读实验状态。
2. 创建 Three 场景实例。
3. 把 Three 交互回调映射成 store 更新。
4. 监听状态变化并调用 `sceneApi.applyState(...)`。

这是当前 3D 与业务状态的总入口。

### 2.5 Three 场景主入口

- 文件：`src/three-lab/createPhotoelectricLabScene.js`

职责：

1. 加载 `resource/all4.glb`
2. 加载 `resource/env.hdr`
3. 初始化 renderer / camera / lights
4. 绑定模型节点
5. 做射线拾取
6. 做按钮、旋钮、光束、数码屏等表现

当前问题是：这个文件已经过大，后续应继续拆分。

### 2.6 相机控制入口

- 文件：`src/three-lab/createOrbitRig.js`

当前职责：

1. 控制球坐标相机半径、方位角、俯仰角
2. 接收滚轮
3. 接收拖拽旋转
4. 接收 `W/S/A/D`

当前缺口：

1. 旋转目前不是“中键拖动专用”
2. `W/S/A/D` 目前不是严格意义上的平面平移
3. 没有基于房间边界做相机约束

### 2.7 实验状态入口

- 文件：`src/stores/experiment.js`

当前 store 已经承担：

1. 实验参数 `params`
2. 器材状态 `apparatus`
3. 光电流、截止电压计算
4. 记录数据
5. 自动扫描
6. 实验报告分析

这是当前项目最稳定的一层，后续尽量保留。

### 2.8 实验规则常量入口

- 文件：`src/features/experiment/constants.js`

当前内容：

1. 材料
2. 滤光片
3. 光阑
4. 电流量程
5. 电压范围

注意：当前 `CURRENT_RANGES` 是 **8 档**，而用户要求是 **6 档**，这里后续必须改。

---

## 3. 当前模型节点梳理

根据当前 `all4.glb` 中已存在的对象名，后续应以这些命名为主：

### 3.1 房间与台体

- `Room`
- `experiment_table`
- `test_bench_base`

### 3.2 汞灯与光电管

- `Mercury_Lamp_Box`
- `Illumination_Port`
- `Phototube_Box`
- `Rear_Port`

### 3.3 滤光片

- `Filter_Rotator`
- `Filter_Window_1`
- `Filter_Window_2`
- `Filter_Window_3`
- `Filter_Window_4`
- `Filter_Window_5`

### 3.4 按钮与旋钮

- `Btn_Power`
- `Btn_AutoManual`
- `Btn_Left`
- `Btn_Right`
- `Btn_Up`
- `Btn_Down`
- `Btn_Query`
- `Btn_Query.001`
- `Btn_ZeroConfirm`
- `Btn_Storage_1` ~ `Btn_Storage_5`
- `Knob_CurrentRange`
- `Knob_ZeroAdjust`

### 3.5 屏幕与线缆

- `Screen_Left`
- `Screen_Right`
- `Line_current_input`
- `Conn_SignalOut`
- `Conn_SyncOut`

---

## 4. 当前代码地图

### 4.1 2D 主页面

- `src/views/HomeView.vue`

职责：

1. 顶部导航
2. 左侧实验导航
3. 中间 2D 虚拟实验台和静态内容切换
4. 右侧状态摘要

### 4.2 2D 虚拟实验台

- `src/components/VirtualLab.vue`
- `src/composables/useVirtualLabController.js`

说明：

这一套仍然是 DOM/CSS 版本的实验台，不是模型版。

后续如果继续维护“2D 实验台 + 3D 独立页”，则这部分仍然保留。

### 4.3 3D 页面

- `src/views/ThreeDView.vue`
- `src/components/three-lab/PhotoelectricLabScene.vue`
- `src/composables/usePhotoelectricSceneController.js`
- `src/three-lab/createPhotoelectricLabScene.js`
- `src/three-lab/createOrbitRig.js`

### 4.4 业务状态与公式

- `src/stores/experiment.js`
- `src/features/experiment/constants.js`
- `src/features/experiment/helpers.js`

说明：

Three 层只应该读写这些状态，不应该再自己保存第二套实验真相。

---

## 5. 六个重点问题的现状、缺口与修补方案

## 5.1 场景光照过暗

### 当前现状

当前场景已经有：

1. `AmbientLight`
2. 一个顶部 `SpotLight`
3. HDR 环境贴图
4. 汞灯本身的局部 `SpotLight`

但是房间整体仍偏暗，原因主要有：

1. 只有一个主顶灯，房间四周没有补光
2. 模型材质较暗，环境贴图强度不够
3. 实验台正面缺少“面向镜头的补光”

### 推荐改法

建议把光照拆为四层：

1. **环境基础光**
   - `AmbientLight('#dfe8f5', 1.1)`
   - 负责抬底

2. **房间顶部主灯**
   - `SpotLight`
   - 照亮实验台与房间整体

3. **镜头前补光**
   - 再加一个从前上方打向实验台的 `SpotLight`
   - 重点提升“实验台正面”可见度

4. **侧后方轮廓光**
   - 强度较低
   - 只负责把实验台从背景墙里拉出来

### 建议改动位置

- 文件：`src/three-lab/createPhotoelectricLabScene.js`
- 位置：场景初始化、灯光创建区域

### 具体建议

1. 保留 HDR，但提高 `toneMappingExposure`
2. 新增前补光和侧补光
3. 对重点节点单独提高材质 `emissiveIntensity` 或 `envMapIntensity`

---

## 5.2 初始视角、缩放、旋转、WASD 平移、边界限制

### 当前现状

当前相机逻辑在 `src/three-lab/createOrbitRig.js`。

已具备：

1. 滚轮缩放
2. 鼠标拖拽旋转
3. `W/S/A/D` 键盘移动
4. 平滑阻尼

但和目标要求仍不一致：

1. 初始镜头不一定正对实验台正面全貌
2. 旋转不是“中键拖动”
3. `W/S/A/D` 目前更像“半屏移 + 半推拉”，不是严格平移
4. 只有限幅，没有按房间边界裁剪

### 推荐改法

将当前相机从“球坐标 + 屏幕偏移”改成：

1. `orbitTarget`
2. `spherical(radius, theta, phi)`
3. `panOffset`
4. `velocityRotate`
5. `velocityPan`
6. `velocityZoom`

### 目标行为

1. **初始视角**
   - 镜头正对实验台正面
   - 中心点对准实验台主面板中心
   - 首屏必须看见实验台正面全貌

2. **滚轮**
   - 只控制 `radius`
   - 平滑插值

3. **中键拖动**
   - 只在 `event.button === 1` 时开启旋转
   - 左键保留给模型交互

4. **W/S/A/D**
   - 修改 `panOffset`
   - 沿“相机右方向”和“世界上方向/视图上方向”平移
   - 不是改半径，不是改方位角

5. **边界限制**
   - 依据 `Room` 包围盒计算相机和 target 可用区域
   - clamp `panOffset`
   - 禁止镜头看穿墙

### 推荐改动位置

- 主文件：`src/three-lab/createOrbitRig.js`
- 读取房间包围盒：`src/three-lab/createPhotoelectricLabScene.js`

### 实现要点

建议新增：

```js
const roomBounds = {
  minX,
  maxX,
  minY,
  maxY,
  minZ,
  maxZ,
}
```

然后在每次 `update()` 时：

1. 先根据 `orbitTarget + spherical + panOffset` 计算相机
2. 再 clamp `target/panOffset`
3. 最终生成 camera.position

### 推荐初始镜头参数

建议后续直接实测后固定一组：

```js
radius: 10.5 ~ 12.5
polar: 1.0 ~ 1.1
azimuth: 0
target: 实验台主面板中心
```

---

## 5.3 按钮、旋钮、滤光片、接线柱交互

### 当前现状

当前 3D 交互入口在：

- `src/three-lab/createPhotoelectricLabScene.js`
- `src/composables/usePhotoelectricSceneController.js`

目前已经有：

1. hover 高亮
2. 按钮按下弹起动画
3. 旋钮拖动
4. 滤光片切换
5. 线缆连接状态切换

但存在以下问题：

1. 滤光片仍保留了错误的“整体转盘瞬转”逻辑
2. 旋钮旋转轴当前直接写死到局部 `rotation.z`
3. 接线柱状态未拆分出“固定连接端”和“后方可切换端”
4. 电流量程当前是 8 档，不符合 6 档要求
5. 滤光片应改为拖动切档，而不是点击按钮切档

### 目标交互拆解

#### 1. 按钮 hover

要求：

1. 鼠标掠过高亮
2. 只做材质高亮，不改位置

建议：

- 保留现有 hover 流程
- 统一提高 `emissiveIntensity`

#### 2. 左键点击按钮

要求：

1. 播放约 10 帧“按下 -> 弹起”

建议：

- 当前 5 帧太短
- 调整为 10 帧
- 位移沿按钮本地法线或本地深度轴

#### 3. 左键拖动旋钮

要求：

1. 柱体类旋钮按圆柱轴线转动

建议：

- 不要再把“所有旋钮都当 `rotation.z`”
- 为每个旋钮记录：
  - `rotationAxis`
  - `angleMin`
  - `angleMax`
  - `stepAngles`

可在 `userData` 中存：

```js
{
  interaction: { type: 'knob' },
  axis: 'y',
  minAngle: -120,
  maxAngle: 120
}
```

然后统一通过一个函数驱动：

```js
setCylinderRotation(node, angleDeg)
```

#### 4. 滤光片拖动切换

目标：

1. 删除当前点击时的错误旋转动画
2. 改成左键按住并拖动切换 5 档
3. 循环切换

建议：

- `Filter_Rotator` 不再注册成普通 button
- 改成 `type: 'step-knob'`
- 拖动累计量超过阈值就切一档
- 只更新后端状态和显示窗，不做错误的持续自旋

#### 5. 电流量程旋钮

目标：

1. 共有 6 档
2. 第一档不能再左
3. 第六档不能再右

当前问题：

- `src/features/experiment/constants.js` 中是 8 档

建议：

1. 先把 `CURRENT_RANGES` 改成 6 项
2. 再给 `Knob_CurrentRange` 配一个固定角度表

例如：

```js
const CURRENT_RANGE_STEPS = [
  { value: '1e-6', angle: -120 },
  { value: '1e-7', angle: -72 },
  { value: '1e-8', angle: -24 },
  { value: '1e-9', angle: 24 },
  { value: '1e-10', angle: 72 },
  { value: '1e-11', angle: 120 },
]
```

#### 6. 电流调零旋钮

目标：

1. 按角度变化驱动 `zeroAdjust`
2. `zeroAdjust` 变化同步到 store

建议：

1. 建立“角度 <-> 数值”双向映射
2. 例如：
   - `-135deg -> -50`
   - `0deg -> 0`
   - `135deg -> 50`

### 接线柱状态重构

用户要求：

1. 汞灯处接线柱固定连接，不可改
2. 试验台后方接线柱可切换
3. 初始为断开且红光警示
4. 连上后绿光

当前问题：

- store 里只有一个 `cableConnected`

建议改成：

```js
apparatus: {
  lampSignalFixedConnected: true,
  rearCableConnected: false,
}
```

然后：

- `Conn_SignalOut` 对应固定连接端，只显示，不响应点击
- `Conn_SyncOut` 或后方端口对应可切换端
- `Line_current_input` 仅表现线缆状态

### 推荐改动位置

- Three 拾取与交互：`src/three-lab/createPhotoelectricLabScene.js`
- 状态映射：`src/composables/usePhotoelectricSceneController.js`
- 状态定义：`src/stores/experiment.js`
- 常量档位：`src/features/experiment/constants.js`

---

## 5.4 汞灯柱体状态与电源联动

### 目标要求

用户要求的两个状态：

1. 发红光：盖子未摘下
2. 发白光并产生光柱：盖子已摘下

并且：

1. 点击汞灯伸出柱体切换盖子状态
2. 点击汞灯主体盒子切换电源
3. 电源关闭时无论盖子状态如何都不发光

### 当前现状

当前代码里：

1. `Mercury_Lamp_Box` 被当成汞灯主体
2. `Illumination_Port` 已可作为汞灯前端发光位置

这是足够实现要求的，不一定需要重新建模。

### 建议状态规则

定义：

```js
lampPowerOn
lampCoverOpen
```

然后表现规则：

1. `lampPowerOn === false`
   - 柱体黑/暗
   - 无光柱
   - 无白光

2. `lampPowerOn === true && lampCoverOpen === false`
   - 柱体红光
   - 无光柱

3. `lampPowerOn === true && lampCoverOpen === true`
   - 柱体白光
   - 有光柱

### 推荐节点映射

- 汞灯主体盒子：`Mercury_Lamp_Box`
- 汞灯伸出柱体：`Illumination_Port`

### 推荐改动位置

- 交互绑定：`src/three-lab/createPhotoelectricLabScene.js`
- 状态切换：`src/composables/usePhotoelectricSceneController.js`
- 默认状态：`src/features/experiment/helpers.js`

---

## 5.5 两块小显示屏分别显示电流与电压

### 当前现状

当前代码已经开始做了这一层：

- `Screen_Left`
- `Screen_Right`
- `CanvasTexture` 贴片覆盖

但后续还需做细：

1. 位置再校正
2. 数码屏字体更稳定
3. 电流、电压分离显示更明确
4. 状态文本减少遮挡

### 推荐实现

#### 左屏

- 标题：`光电流`
- 主值：`currentPhotocurrentMicroamp`
- 单位：`uA`

#### 右屏

- 标题：`电压`
- 主值：`params.voltage`
- 单位：`V`

### 推荐封装

把当前数码屏逻辑从主场景文件中拆出来：

新增文件：

- `src/three-lab/scene/createScreenOverlay.js`

导出：

```js
createScreenOverlay(...)
updateScreenOverlay(...)
disposeScreenOverlay(...)
```

这样后续如果想改成七段数码字形，不需要再动场景主文件。

---

## 5.6 文档化后的推荐结构

当前 `createPhotoelectricLabScene.js` 已经太大，后续建议拆成：

```text
src/three-lab/
  createPhotoelectricLabScene.js        # 只负责拼装
  createOrbitRig.js                     # 相机控制
  scene/
    loadLabModel.js                     # 加载模型和 HDR
    bindLabNodes.js                     # 节点查找、命名校验
    registerInteractions.js             # 按钮/旋钮/接线柱语义绑定
    applyLighting.js                    # 房间灯光和补光
    applySceneState.js                  # 根据 store 状态刷新模型表现
    createScreenOverlay.js              # 数码屏覆盖层
    createBeamEffect.js                 # 光柱和汞灯局部发光
    createInteractionAnimations.js      # 按钮/旋钮动画
```

推荐原则：

1. **store 决定状态**
2. **controller 负责转译**
3. **scene 负责表现**

不要再把“状态计算 + 输入处理 + 动画 + 模型加载”继续堆进一个文件。

---

## 6. 后续建议的状态字段调整

当前 `apparatus` 字段不够表达用户要求，建议补成：

```js
apparatus: {
  photocellCoverOpen: false,
  lampCoverOpen: false,
  lampPowerOn: true,
  rearCableConnected: false,
  lampSignalFixedConnected: true,
  photocellDistance: 40,
  filterWavelength: 365,
  apertureSize: 4,
  currentRange: '1e-6',
  voltageRange: '-2-2',
  zeroAdjust: 0,
  workMode: 'manual',
  showPhotons: true,
  showElectrons: true,
  showField: true,
}
```

同时把依赖 `cableConnected` 的地方全部切到 `rearCableConnected`。

---

## 7. 修补优先级建议

建议按以下顺序修：

### 第1优先级：镜头与光照

先解决：

1. 场景过暗
2. 首屏看不到实验台正面
3. 中键旋转 / 滚轮缩放 / `W/S/A/D` 平移
4. 房间边界限制

原因：

这是所有后续 3D 交互的基础体验层。

### 第2优先级：状态拆分

先改 store：

1. `cableConnected` -> `rearCableConnected`
2. `CURRENT_RANGES` 8 档改 6 档
3. 增加接线端固定状态

原因：

不先拆状态，后续交互会继续混乱。

### 第3优先级：交互语义重做

依次重做：

1. 汞灯柱体点击
2. 汞灯主体点击
3. 接线柱点击
4. 滤光片拖动切档
5. 电流量程旋钮分档
6. 调零旋钮角度映射

### 第4优先级：数码屏和细节动画

最后再修：

1. 数码屏位置和材质
2. 按钮 10 帧动画
3. 更平滑的旋钮动画
4. 红绿警示灯效果

---

## 8. 明确的代码落点清单

### 必改文件

- `src/three-lab/createOrbitRig.js`
  - 中键旋转
  - 平滑平移
  - 房间边界限制

- `src/three-lab/createPhotoelectricLabScene.js`
  - 灯光补强
  - 汞灯状态表现
  - 交互语义拆分
  - 按钮/旋钮动画修正

- `src/composables/usePhotoelectricSceneController.js`
  - 3D 交互 -> store 更新逻辑

- `src/stores/experiment.js`
  - 线缆状态重构
  - 新档位、新状态同步

- `src/features/experiment/constants.js`
  - 电流量程改为 6 档

- `src/features/experiment/helpers.js`
  - 默认状态修正

### 建议新增文件

- `src/three-lab/scene/createScreenOverlay.js`
- `src/three-lab/scene/applyLighting.js`
- `src/three-lab/scene/registerInteractions.js`
- `src/three-lab/scene/applySceneState.js`

---

## 9. 当前已知不一致点

后续开发前必须先知道这些：

1. 当前 3D 页可交互，但还未完全符合用户指定交互语义。
2. 当前滤光片仍不是“拖动切档”的正式实现。
3. 当前电流量程是 8 档，和需求冲突。
4. 当前接线柱逻辑仍是单状态，不满足“固定连接端 + 可切换后端”。
5. 当前相机控制还不是“中键旋转 + 严格平移 + 房间边界限制”。

---

## 10. 建议的下一轮开发目标

下一轮建议只做下面四件事，不要并行扩散：

1. 重写 `createOrbitRig.js`
2. 拆分 `cableConnected` 状态
3. 改 `CURRENT_RANGES` 为 6 档
4. 把 `Filter_Rotator` 从点击切换改成拖动切档

这四件事做完以后，再继续修按钮、灯光和数码屏细节，整体节奏会更稳。

