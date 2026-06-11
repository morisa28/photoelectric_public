# 任务总结文档-new(17)-光电管拖拽与2D距离收口

## 1. 本轮目标

本轮工作的目标不是重做实验页，而是在现有 `new(17)` 基线上继续收口光电管相关交互，主要包括：

1. 将 3D 场景里光电管现存的两种拖动判定方式合并为一种。
2. 将 3D 光电管拖动步进调整为原来的一半，使拖动更细。
3. 修正 2D 界面中光电管距离方向与页面布局不一致的问题。
4. 将 2D 界面光电管距离上下限改为 `25cm ~ 50cm`。

---

## 2. 本轮修改文件

### 2.1 3D 场景

- `src/components/SplineScene.vue`

### 2.2 2D 距离映射与滑块

- `src/features/experiment/helpers.js`
- `src/composables/useVirtualLabController.js`

### 2.3 实验状态收口

- `src/features/experiment/constants.js`
- `src/stores/experiment.js`

### 2.4 过程文档

- `document/阶段记录/new14-new21-仪器与同步/new17-接续10-实体线缆与光电管拖拽修正.md`

---

## 3. 3D 光电管拖拽判定收口

### 3.1 原问题

旧实现中，光电管拖动命中存在两条路径：

1. `raycast`
2. `projected-bounds`

这会带来两个问题：

1. 悬停、按下开始拖拽、原生 `dragDrop` 拦截这几个阶段不一定使用同一标准。
2. 诊断面板里的“命中来源”会混入屏幕投影兜底，不利于继续排查真实命中链路。

### 3.2 本轮做法

在 `src/components/SplineScene.vue` 中，把光电管命中统一收口到：

- `resolvePhototubeDragHit(intersections)`

当前逻辑只保留：

1. 先排除滤光片命中
2. 再根据射线命中链中是否包含 `Phototube_Box`
3. 命中后统一返回 `Phototube_Box_side`

对应位置：

- `src/components/SplineScene.vue:1541`
- `src/components/SplineScene.vue:1569`
- `src/components/SplineScene.vue:1577`
- `src/components/SplineScene.vue:1613`

### 3.3 收口结果

现在以下三处复用的是同一个光电管命中结果：

1. `pointermove` 悬停反馈
2. `pointerdown` 开始拖拽判定
3. 原生 `dragDrop` 拦截

对应位置：

- `src/components/SplineScene.vue:1613`
- `src/components/SplineScene.vue:2755`
- `src/components/SplineScene.vue:3206`

这意味着当前版本中，“光电管命中”只表示真实 `raycast` 命中结果，不再混入屏幕投影包围盒兜底。

---

## 4. 3D 光电管拖动步进减半

### 4.1 原问题

用户反馈当前光电管拖动仍偏粗，调整时不够细。

### 4.2 本轮做法

本轮从两个层面同时收细：

1. 3D 拖动像素灵敏度减半
2. 实际距离写回步长减半

#### 像素换算

`SplineScene.vue` 中光电管拖动参数由：

- `pixelPerCm: 8`

改为：

- `pixelPerCm: 16`

对应位置：

- `src/components/SplineScene.vue:223`

这表示同样的鼠标横向位移，现在只会换来原来一半的距离变化。

#### 距离步进

新增统一常量：

- `PHOTOCELL_DISTANCE_STEP = 0.5`

对应位置：

- `src/features/experiment/constants.js:31`

`store` 中光电管距离不再按整厘米收口，而是按 `0.5cm` 档位归一化：

- `src/stores/experiment.js:732`

`SplineScene.vue` 中光电管离散步进写回也同步改成：

- `previous + direction * PHOTOCELL_DISTANCE_STEP`

对应位置：

- `src/components/SplineScene.vue:3015`

### 4.3 收口结果

现在 3D 光电管拖动的有效步长已经从整厘米降到 `0.5cm`，并且拖动灵敏度也同步变为原来的二分之一。

---

## 5. 2D 光电管距离方向修正

### 5.1 原问题

2D 页面中，光电管位于左侧，汞灯位于右侧。

因此页面上的真实空间关系应当是：

1. 光电管向左移动：离汞灯更远，距离增大
2. 光电管向右移动：离汞灯更近，距离减小

但旧版 2D 标尺换算方向与这个页面布局相反，导致拖动方向和距离含义不一致。

### 5.2 本轮做法

在 `src/features/experiment/helpers.js` 中，反转了 2D 距离与标尺位置的映射方向：

- `distanceToSliderPosition`
- `sliderPositionToDistance`

对应位置：

- `src/features/experiment/helpers.js:35`
- `src/features/experiment/helpers.js:41`

修正后，2D 标尺遵循：

1. 左边表示更远
2. 右边表示更近

与页面中“左侧光电管、右侧汞灯”的布局一致。

---

## 6. 2D 光电管距离范围改为 25cm 到 50cm

### 6.1 原问题

2D 界面此前沿用的是更宽的距离映射范围，不符合这一轮要求的：

- `25cm ~ 50cm`

如果只改单点逻辑，而不同时修改刻度、滑块和换算入口，后续很容易再次出现：

1. 刻度一套范围
2. 滑块一套范围
3. 写回 store 又是另一套范围

### 6.2 本轮做法

在 `src/features/experiment/helpers.js` 中新增 2D 专用常量：

1. `TWO_D_PHOTOCELL_DISTANCE_MIN = 25`
2. `TWO_D_PHOTOCELL_DISTANCE_MAX = 50`
3. `TWO_D_PHOTOCELL_DISTANCE_RANGE`

对应位置：

- `src/features/experiment/helpers.js:26`

2D 滑块位置换算全部改为复用这组常量：

- `src/features/experiment/helpers.js:35`
- `src/features/experiment/helpers.js:41`

同时 `useVirtualLabController.js` 中的标尺刻度不再手写一套独立位置公式，而是直接复用：

- `distanceToSliderPosition(cm)`

对应位置：

- `src/composables/useVirtualLabController.js:48`

### 6.3 当前结果

2D 光电管距离现在统一表现为：

1. 下限 `25cm`
2. 上限 `50cm`
3. 刻度显示为 `25 / 30 / 35 / 40 / 45 / 50`
4. 方向为“左远右近”

需要注意的是：

1. 本轮收的是 2D 专用滑块与刻度范围
2. `store` / 3D 仍保留更宽的全局物理夹取区间 `20cm ~ 60cm`

这样可以保证：

1. 2D 页面先满足当前界面需求
2. 不直接打断已有 3D 场景和其它逻辑的物理边界

---

## 7. 验证结果

### 7.1 构建验证

本轮修改后，多次执行：

```bash
npm run build
```

均已通过。

### 7.2 3D 诊断与交互验证

已验证内容包括：

1. `scene:report` 可正常生成
2. 光电管悬停命中可返回 `Phototube_Box_side`
3. 命中来源为 `raycast`
4. 页面内直接派发 `PointerEvent` 后，`photocellDistance` 可从 `40` 变化到 `52`

这说明：

1. 光电管拖拽命中合并后，前端监听链路仍然有效
2. 当前诊断结果已经只剩单一路径判定

---

## 8. 当前状态总结

本轮收口后，光电管相关交互已经形成以下状态：

1. 3D 光电管拖拽判定统一为单一 `raycast` 入口
2. 3D 光电管拖动步进已减半，距离精度为 `0.5cm`
3. 2D 光电管距离方向已修正为“左远右近”
4. 2D 光电管距离范围已改为 `25cm ~ 50cm`
5. 2D 刻度、滑块位置、距离反算已经统一复用同一套映射逻辑

---

## 9. 后续建议

如果后续继续做光电管距离相关收口，建议按下面顺序推进：

1. 评估是否要把 `store` 的全局物理夹取区间也同步改为 `25cm ~ 50cm`
2. 如果改全局区间，要同时检查 3D 光电管位移范围、线缆长度、光强公式和历史记录回放
3. 如果只继续做 2D 界面优化，则优先保持当前“2D 专用范围”和“全局物理范围”分离，避免误伤 3D

当前版本已经可以作为后续继续收口 2D/3D 距离统一策略的稳定基线。
