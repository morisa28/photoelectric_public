# 任务总结文档-new(21.2)-3D盖子状态同步

## 1. 任务目标

本轮需求是接入此次新增的两个 3D 盖子模型：

1. 光电管盖子模型。
2. 汞灯盖子模型。

要求两个盖子初始为盖上状态，点击后切换为打开状态，并且开合状态不能只停留在 Spline 场景内部，而要同步到全局实验状态，与现有光电流、记录校验、微观动画和 2D 实验台逻辑关联。

## 2. 关键模型与状态字段

本轮确认到的 Spline 对象名如下：

1. 光电管盖子：
   - `Photocell_cover`
   - `Photocell_cover_2`

2. 汞灯盖子：
   - `Mercury_Lamp_Box_cover`
   - `Mercury_Lamp_Box_cover_2`

继续复用现有 Pinia 全局状态字段：

1. `apparatus.photocellCoverOpen`
2. `apparatus.lampCoverOpen`

默认状态仍保持为 `false`，即初始盖上。

## 3. 本轮代码改动

### 3.1 全局 store 动作补齐

文件：`src/stores/experiment.js`

新增显式动作：

1. `setPhotocellCoverOpen(open, meta)`
2. `togglePhotocellCover(meta)`
3. `setLampCoverOpen(open, meta)`
4. `toggleLampCover(meta)`

作用：

1. 让 2D 和 3D 都通过同一套 store 动作修改盖子状态。
2. 保留 `updateApparatus()` 内已有的光电流噪声重采样、同步 revision、光电条件计算联动。
3. 避免 3D 盖子点击只触发 Spline 本地 Toggle，导致全局实验状态不知道盖子已打开。

### 3.2 2D 控制器改为调用统一动作

文件：`src/composables/useVirtualLabController.js`

将原来直接调用 `updateApparatus()` 的盖子 setter 改为：

1. `experimentStore.setPhotocellCoverOpen(value)`
2. `experimentStore.setLampCoverOpen(value)`

这样 2D 页面按钮、3D 模型点击、报告和动画看到的是同一份状态。

### 3.3 3D 场景接入盖子点击与 state 同步

文件：`src/components/SplineScene.vue`

新增盖子配置：

1. `COVER_OBJECT_NAMES`
2. `COVER_STATE_TARGET_NAMES`
3. `COVER_STATES`

将盖子对象加入按钮动作映射：

1. 点击 `Photocell_cover` / `Photocell_cover_2` -> `toggle-photocell-cover`
2. 点击 `Mercury_Lamp_Box_cover` / `Mercury_Lamp_Box_cover_2` -> `toggle-lamp-cover`

新增同步函数：

1. `setSplineObjectState(name, state)`
2. `syncCoverState(kind, open)`
3. `syncCoverStates()`

状态规则：

1. 全局状态为 `false`：模型 state 设为 `Base State`，表现为盖上。
2. 全局状态为 `true`：模型 state 设为 `State`，表现为打开。

同时新增 watcher：

1. 监听 `apparatus.photocellCoverOpen`
2. 监听 `apparatus.lampCoverOpen`

保证从 2D 页面切换盖子时，3D 模型也会同步开合。

### 3.4 诊断信息增强

文件：`src/components/SplineScene.vue`

`window.__VISUAL_INSPECTOR__.getDiagnostics()` 新增：

1. `covers`
2. `storeSnapshot.photocellCoverOpen`
3. `storeSnapshot.lampCoverOpen`
4. `storeSnapshot.lampPowerOn`
5. `storeSnapshot.photoelectricConditionsReady`
6. `storeSnapshot.generatedPhotocurrentAmp`

作用是后续自动化脚本可以直接判断盖子状态是否已经影响到光电流逻辑。

### 3.5 文档更新

文件：`README.md`

补充说明：

1. 3D 盖子点击后统一写回 Pinia。
2. 模型开合 state 由全局状态 watcher 反向同步。
3. 2D、3D、记录校验和微观动画使用同一份盖子状态。

## 4. 验证结果

### 4.1 构建验证

已执行：

```sh
npm run build
```

结果：构建通过。

### 4.2 bridge 模拟点击验证

通过 `window.__VISUAL_INSPECTOR__.simulateButtonAction()` 模拟点击：

1. `Photocell_cover`
2. `Mercury_Lamp_Box_cover`

结果：

1. `photocellCoverOpen` 从 `false` 变为 `true`。
2. `lampCoverOpen` 从 `false` 变为 `true`。
3. 两个盖子模型从 `Base State` 同步到 `State`。

### 4.3 真实鼠标点击验证

通过 Playwright 真实点击盖子投影位置验证：

初始状态：

```txt
photocellCoverOpen = false
lampCoverOpen = false
photoelectricConditionsReady = false
generatedPhotocurrentAmp = 0
```

点击两个盖子后：

```txt
photocellCoverOpen = true
lampCoverOpen = true
photoelectricConditionsReady = true
generatedPhotocurrentAmp != 0
```

说明盖子模型点击已经真正影响全局实验逻辑，不只是改变 3D 外观。

## 5. 涉及文件

1. `src/stores/experiment.js`
2. `src/composables/useVirtualLabController.js`
3. `src/components/SplineScene.vue`
4. `README.md`
5. `document/阶段记录/new14-new21-仪器与同步/new21.2-3D盖子状态同步总结.md`

## 6. 当前状态

本轮任务已完成。

两个新增 3D 盖子模型已经接入：

1. 初始盖上。
2. 点击打开。
3. 开合状态写入全局 Pinia store。
4. 全局状态变化反向同步 3D 模型 state。
5. 光电流生成条件、记录校验、2D 实验台和微观动画均可读取同一份状态。
