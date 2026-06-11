# 任务总结文档-new(21)-光电流随机与微观动画修正

## 1. 本轮目标

本轮工作围绕光电效应实验的数据真实性和微观动画表现继续收口，主要目标包括：

1. 调整主光电流生成中的随机误差逻辑。
2. 统一记录数据、图表和报告中的截止电压数据源。
3. 将默认滤光片状态改为 `365nm`。
4. 修正微观动画中“遏止 / 截止 / 无约束 / 加速”的判定和电子运动表现。

---

## 2. 光电流随机误差调整

### 2.1 原问题

旧的基础光电流误差使用稳定 hash 噪声：

- 同一组参数下误差固定。
- 改回同一参数后仍可能得到同一误差。
- 不符合“每次重新生成光电流时应有细微随机差异”的需求。

### 2.2 本轮做法

本轮将主光电流误差改为状态内保存的一次性随机值：

- 新增 `photocurrentNoiseRatio`。
- 新增 `sampleBasePhotocurrentNoise()`。
- `generatedPhotocurrentAmp` 使用 `current * (1 + photocurrentNoiseRatio)`。
- 随机误差幅度改为 `-5% ~ +5%`。

当前不会在 getter 内直接调用 `Math.random()`，避免 2D、3D、记录值在同一刷新周期内不一致。

### 2.3 重采样规则

会重新抽样基础光电流误差的条件：

1. 频率变化。
2. 光强变化。
3. 材料变化。
4. 灯、盖子、滤光片、光阑、光电管距离等会改变光电流生成条件的器材状态变化。
5. 仪器电源状态变化。

不会重新抽样的条件：

1. 电压变化。
2. 自动扫描过程中每一步电压变化。
3. 测量模式切换导致的电压夹紧。

这保证了 I-U 曲线扫描时，电压只改变物理响应曲线，不额外改变基础随机误差。

---

## 3. 记录数据与截止电压数据源统一

### 3.1 原问题

旧记录逻辑中：

- `record.voltage` 是当前实际外加电压。
- `record.cutoffVoltage` 却直接保存理论截止电压 `currentCutoffVoltage`。

因此历史记录、Uc-ν 图表和报告中显示的“截止电压”并不是用户实际测得的电压。

### 3.2 本轮做法

本轮将 `record.cutoffVoltage` 改为实际测得截止电压：

- 在截止电压测试模式下，记录时按 `Uc = max(0, -voltage)` 生成测得截止电压。
- 额外保存 `theoreticalCutoffVoltage`。
- 额外保存 `cutoffVoltageDeviation = measuredUc - theoreticalUc`。
- 保存 `measurementMode`，用于区分截止电压测试与伏安特性测试。

伏安特性测试记录不再参与 Uc-ν 拟合。

### 3.3 图表与报告收口

本轮同步调整：

1. Uc-ν 图表只读取截止电压测试记录。
2. Uc-ν 图表显示的是测得截止电压。
3. 历史记录和控制台最近记录支持无截止电压的 I-U 记录显示为 `—`。
4. 实验报告拟合明细显示“测得截止电压 / 理论截止电压 / 测量偏差”。
5. 分析摘要的记录数改为参与 Uc-ν 拟合的截止电压记录数。

---

## 4. 默认滤光片状态

默认状态已从 `577nm` 改为 `365nm`：

1. `params.frequency` 改为 `FILTERS[0].frequency * 1e14`。
2. `apparatus.filterWavelength` 改为 `365`。

这样 2D、3D、动画和记录中的默认滤光片波长与后台频率保持一致。

---

## 5. 微观动画状态与电子运动修正

### 5.1 状态判定规则

微观动画现在按以下逻辑判定：

1. `|photocurrentAmp| / 20e-13 <= 5%`：状态为“截止”。
2. `|photocurrentAmp| / 20e-13 > 5%` 且电压为负：状态为“遏止”。
3. 电压为 `0` 附近：状态为“无约束”。
4. 电压为正：状态为“加速”。

### 5.2 截止状态

截止状态不再表示电子完全无法逸出金属板。

当前表现为：

1. 电子仍可逸出金属板。
2. 电子会进入真空飞行区飞行一段距离。
3. 电子随后被反向电场拉回或折返。
4. 没有任何电子能到达阳极。
5. 收集率固定为 `0`。

### 5.3 遏止状态

遏止状态下：

1. 反向电压绝对值越大，收集率越低。
2. 反向电压绝对值越大，电子在真空飞行区的飞行速度越小。
3. 仍保留“每 3 秒至少随机一个电子可以到达阳极”的视觉保底。

这个保底只在遏止状态生效，不会影响截止状态。

### 5.4 无约束与加速状态

无约束状态：

- 电压为 `0` 附近，电子不受反向电场影响。

加速状态：

- 电压为正。
- 正向电压越大，电子飞行速度越快。
- 已逸出的电子收集率为 `100%`。

---

## 6. 主要修改文件

本轮主要修改：

1. `src/features/experiment/constants.js`
2. `src/features/experiment/helpers.js`
3. `src/stores/experiment.js`
4. `src/components/IntegratedPlotPanel.vue`
5. `src/components/HistoryRecords.vue`
6. `src/components/ExperimentReport.vue`
7. `src/components/IntegratedVirtualConsole.vue`
8. `src/features/photoelectric-animation/mapping.js`
9. `src/features/photoelectric-animation/engine.js`
10. `src/components/animation/PhotoelectricAnimationPanel.vue`

---

## 7. 验证结果

已执行多次生产构建：

- `npm run build`

最终构建通过。

额外做过轻量映射检查：

1. 基础光电流小于等于 5% 时，动画状态为“截止”，收集率为 `0`。
2. 基础光电流大于 5% 且负压时，动画状态为“遏止”。
3. 负压绝对值增大时，收集率与电子速度下降。
4. 零压状态为“无约束”。
5. 正压状态为“加速”，收集率为 `100%`。

已知旧单元测试问题：

- `npm run test:unit` 仍会因 `src/components/__tests__/HelloWorld.spec.js` 引用不存在的 `../HelloWorld.vue` 失败。
- 该问题属于仓库旧测试残留，不是本轮改动引入。

---

## 8. 后续注意事项

1. 如果后续要让“测得截止电压”带独立电压测量误差，应在 `recordData()` 中新增电压误差采样，而不是复用电流误差。
2. 当前基础光电流误差和记录电流误差是两层不同误差：基础误差影响实时光电流，记录误差只影响 `record.current`。
3. Uc-ν 拟合当前只使用截止电压测试模式记录；自动扫描得到的 I-U 数据只用于 I-U 曲线。
4. 微观动画的 5% 判定阈值基于 `PHOTOCURRENT_LEVEL_REFERENCE_AMP = 20e-13`，后续若调整饱和电流参考值，需要同步检查动画状态边界。
