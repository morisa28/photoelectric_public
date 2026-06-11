# new(15) 任务总结文档：3D 右上角数据记录与查看弹窗

## 1. 本轮任务结论

本轮任务已完成，当前 3D 界面右上角已经增加以下两个按钮：

1. `记录当前数据`
2. `查看数据`

其中：

1. `记录当前数据` 已接入与 2D 界面同一套记录逻辑
2. `查看数据` 会打开一个小窗
3. 小窗中已包含以下信息：
   - 数据曲线
   - 拟合公式
   - 当前电流量程
   - 当前滤光片波长
   - 当前测量模式
   - 当前工作模式
4. 数据曲线已直接复用 2D 的完整图表组件，不再使用缩略曲线

## 2. 本次工作目录

- Windows 路径：`F:\file\wsl_shared_files\event\new(15)`
- WSL 路径：`/home/just_monika/win_share/event/new(15)`

## 3. 需求落实结果

### 3.1 3D 右上角按钮

已在 `src/views/ThreeDView.vue` 的右上角工具栏中增加：

1. `记录当前数据`
2. `查看数据`
3. 原有 `返回 2D 实验台`
4. 原有 `打开诊断模式`

当前工具栏已支持自动换行，避免在较窄屏幕上挤压错位。

### 3.2 记录当前数据

3D 页面中的 `记录当前数据` 按钮当前实现与 2D 一致：

1. 先检查 `experimentStore.canRecordData`
2. 若当前不满足记录条件，则弹出 `recordBlockedReason`
3. 若满足条件，则调用 `experimentStore.recordData()`

这意味着 3D 页面不会绕过 2D 现有的调零、接线、测量模式和记录校验规则。

### 3.3 查看数据弹窗

点击 `查看数据` 后，会在 3D 页面上方弹出一个数据小窗。  
该小窗已包含：

1. 拟合公式
2. 当前电流量程
3. 当前滤光片波长
4. 当前测量模式
5. 当前工作模式
6. 完整数据曲线区域

弹窗支持以下关闭方式：

1. 点击右上角关闭按钮
2. 点击遮罩空白区域
3. 按 `Esc`

## 4. 数据曲线为何与 2D 保持一致

这次没有单独在 3D 页面重写一套图表逻辑，而是直接复用：

- `src/components/IntegratedPlotPanel.vue`

因此当前 3D 小窗中的数据曲线与 2D 界面中的数据曲线保持同一套实现，具体一致性包括：

1. 同样的图表切换方式
   - `Uc-ν 曲线`
   - `I-U 曲线`
2. 同样的 ECharts 配置逻辑
3. 同样的实验记录来源
4. 同样的拟合结果来源
5. 同样的图例、坐标轴、提示框和显示口径

也就是说，本次实现满足“显示 2D 中的完整数据曲线，而不是缩略曲线”的要求。

## 5. 关键实现位置

本轮主要修改文件：

1. `src/views/ThreeDView.vue`

其中完成了以下内容：

1. 新增 3D 工具栏按钮
2. 新增数据查看弹窗结构
3. 新增拟合公式和当前实验状态信息展示
4. 接入 `IntegratedPlotPanel`
5. 增加 `Esc` 关闭弹窗逻辑
6. 增加弹窗与工具栏样式

本轮直接复用、未改动核心逻辑的相关文件：

1. `src/components/IntegratedPlotPanel.vue`
2. `src/stores/experiment.js`

## 6. 关键实现口径

### 6.1 拟合公式来源

拟合公式取自：

- `experimentStore.analysisSummary`

展示口径为：

```txt
Uc = kν + (b)
```

当数据不足以完成拟合时，弹窗显示：

```txt
待至少两种频率数据后生成
```

### 6.2 当前状态信息来源

弹窗中的各项状态直接读取统一 store：

1. 电流量程：`apparatus.currentRange`
2. 滤光片波长：`apparatus.filterWavelength`
3. 测量模式：`measurementModeLabel`
4. 工作模式：`apparatus.workMode`

这样可以保证 2D、3D、图表和报告区始终读取同一份实验状态。

## 7. 验证结果

本轮已执行：

```sh
npm run build
```

结果：

1. 构建通过
2. 新增的 `ThreeDView.vue` 模板、脚本和样式没有编译错误
3. `IntegratedPlotPanel` 在 3D 页面中的引入与打包正常

## 8. 后续接手建议

如果后续还要继续扩展 3D 页面中的数据能力，优先沿用当前策略：

1. 记录、分析、拟合全部继续复用 `src/stores/experiment.js`
2. 图表优先复用 `src/components/IntegratedPlotPanel.vue`
3. 不要在 3D 页面里再写一套独立的缩略图逻辑
4. 如需增加更多实验元信息，优先在 `ThreeDView.vue` 中继续读取 store 派生值

这样可以避免 2D 与 3D 的数据展示口径再次分叉。
