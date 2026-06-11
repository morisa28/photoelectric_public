# 物理框架审查第 5 轮：核心 store、动画引擎与诊断审查

## 本轮审查范围

本轮只读审查核心片段：

- `src/stores/experiment.js`
- `src/features/photoelectric-animation/engine.js`
- `src/services/dataDiagnostic.js`
- `src/components/animation/PhotoelectricAnimationPanel.vue`
- `src/components/AIAssistant.vue`
- 少量报告/图表组件对记录数据的处理方式

本文只记录动画和数值计算相关问题，不修改业务代码。

## 文件逐项审查

### `src/stores/experiment.js`

审查结论：store 已经承担统一真实实验状态源，并实现了截止电压、光强重算、电流量程、调零、记录和自动扫描等核心逻辑。频率-截止电压关系符合 K001-K005；光强、孔径、距离影响有效光强符合 K006/K012 的教学模型。但主光电流公式在反向过截止后产生负的“正常阴极光电流”，这是本轮最核心的物理风险。

关键代码证据：

- 理论截止电压：`PLANCK_EV * frequency - workFunction`，小于 0 时置 0：`src/stores/experiment.js:292`
- 光电效应成立条件要求灯亮、两盖打开、光强大于 0、截止电压大于 0：`src/stores/experiment.js:201`
- 饱和电流随光强变化：`(intensity / 100) * PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP`：`src/stores/experiment.js:214`
- 主 I-U 响应：`saturationCurrent * Math.tanh((voltage + cutoff) / 0.45)`：`src/stores/experiment.js:221`
- 光强随距离平方反比、随光阑大小变化：`src/stores/experiment.js:792`
- 截止电压记录把负外加电压取绝对值：`src/stores/experiment.js:126`、`src/stores/experiment.js:819`

### `src/features/photoelectric-animation/engine.js`

审查结论：引擎按动画状态生成光子、低能失败电子、正常光电子、暗电流、本底电流和寄生事件。低能背景事件在代码中被强制失败，不会进入正常飞行收集，符合 K002/K015 的教学演示边界；正常电子的飞行、返回和收集由 `stepElectronFlight()` 处理，符合 K009/K010。

关键代码证据：

- 低能事件在 `spawnEvent()` 中被强制 `success: false`、`forcedFailure: true`：`src/features/photoelectric-animation/engine.js:1208`
- 低能背景事件由 `lowEnergyBackgroundRate` 触发，但以 `energyKind: 'low'` 生成：`src/features/photoelectric-animation/engine.js:1681`
- 低能失败被统计到 `lowEnergyBlockedCount`：`src/features/photoelectric-animation/engine.js:1473`
- 正常光电子进入飞行后调用 `createElectronFlightState()` 和 `stepElectronFlight()`：`src/features/photoelectric-animation/engine.js:1498`
- 收集时才登记正常电子收集样本：`src/features/photoelectric-animation/engine.js:1582`
- 统计面板使用实时收集率、测量电流和残余电流收集比例：`src/features/photoelectric-animation/engine.js:2620`

### `src/services/dataDiagnostic.js`

审查结论：数据诊断服务的物理口径基本符合 K005/K013：检查截止电压随频率增加、拟合质量、普朗克常量误差和重复频率。但服务没有过滤 `cutoffVoltage === null` 的伏安扫描记录，且返回值没有暴露实际计算的 `rSquared`，容易让伏安记录和截止电压记录混在一起。

关键代码证据：

- 趋势检查直接比较 `curr.cutoffVoltage <= prev.cutoffVoltage`：`src/services/dataDiagnostic.js:24`
- 回归直接对全部 `records` 做 `sumY += r.cutoffVoltage`：`src/services/dataDiagnostic.js:57`
- 返回对象中 `rSquared` 固定为 `null`：`src/services/dataDiagnostic.js:110`

### `src/components/animation/PhotoelectricAnimationPanel.vue`

审查结论：讲解文案整体符合 K006/K010/K011/K015。文案明确说明光强决定事件密度，频率与逸出功决定 `Kmax`，电压改变电场力和轨迹，没有发现“光强越大电子能量越大”的核心误导。

关键代码证据：

- 低于阈频文案说明单个光子能量不足以克服逸出功：`src/components/animation/PhotoelectricAnimationPanel.vue:464`
- 常规状态文案说明 `Kmax` 由频率与阈频关系决定，光强决定事件密度，电压改变电子轨迹：`src/components/animation/PhotoelectricAnimationPanel.vue:478`
- 能量层文案展示 `phi`、`hnu`、`Kmax`：`src/components/animation/PhotoelectricAnimationPanel.vue:502`
- 误差层文案区分阳极污染、本底电流和暗电流：`src/components/animation/PhotoelectricAnimationPanel.vue:511`

### `src/components/AIAssistant.vue`

审查结论：AI 辅助分析把所有记录都当作截止电压记录处理，和自动伏安扫描记录不兼容。该问题属于数据解释风险，可能导致报错或错误诊断。

关键代码证据：

- `sendToAPI()` 对每条记录直接调用 `r.cutoffVoltage.toFixed(3)`：`src/components/AIAssistant.vue:182`
- `getSystemPrompt()` 把全部记录传给 `diagnoseData(recordsData)`：`src/components/AIAssistant.vue:228`

## 问题清单

| 编号 | 优先级 | 文件路径 | 函数/常量名 | 代码证据 | 对应理论条目 | 影响范围 | 证据等级 | 是否建议后续修改 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| B3-001 | P1 | `src/stores/experiment.js` | `generatedPhotocurrentAmp()` | 主光电流使用 `saturationCurrent * Math.tanh((voltage + cutoff) / 0.45)`；当 `voltage < -cutoff` 时，该值为负：`src/stores/experiment.js:221` | K003、K007、K009、K011 | 正常阴极光电流在超过遏止电压后应趋近 0，而不应变成负向电流；负向电流应由阳极寄生、本底或暗电流解释。该问题会影响表盘读数、伏安曲线、记录数据和微观动画真实阴极电流输入 | A | 是，建议把正常阴极光电流下限限制为 0，寄生/暗电流另行叠加 |
| B3-002 | P1 | `src/components/PrincipleAnimationSection.vue` | `trueCathodePhotocurrentAmp()` | 原理动画区复用了同类 `tanh((voltage + currentCutoffVoltage) / 0.45)` 公式：`src/components/PrincipleAnimationSection.vue:381` | K003、K009、K011 | 原理动画讲解中“真实阴极光电流”也会在反向过截止后变负，与 K011 中寄生项/误差项拆分口径冲突 | A | 是，建议与 store 同步修正 |
| B3-003 | P2 | `src/services/dataDiagnostic.js` | `diagnoseData()` | 趋势和回归直接使用全部记录的 `cutoffVoltage`，没有过滤 `null` 或非截止模式记录：`src/services/dataDiagnostic.js:24`、`src/services/dataDiagnostic.js:57` | K013 | 自动伏安扫描记录的 `cutoffVoltage` 为 `null`，进入诊断后会被当作 0 或触发无意义趋势/拟合判断 | B | 是，建议只使用 `measurementMode === 'cutoff'` 且 `cutoffVoltage` 有限的记录 |
| B3-004 | P2 | `src/components/AIAssistant.vue` | `sendToAPI()` / `getSystemPrompt()` | AI 数据摘要直接对所有记录执行 `r.cutoffVoltage.toFixed(3)`，并把全部记录传给 `diagnoseData()`：`src/components/AIAssistant.vue:182`、`src/components/AIAssistant.vue:228` | K013 | 伏安扫描记录 `cutoffVoltage === null` 时可能报错；即使不报错也会把 I-U 曲线数据误解释为 `U_c - nu` 数据 | B | 是，建议按测量模式拆分数据摘要和诊断 |
| B3-005 | P2 | `src/stores/experiment.js` | `recordData()` | 截止模式记录值来自 `measureCutoffVoltageFromAppliedVoltage(this.params.voltage)`，即直接取当前负电压绝对值：`src/stores/experiment.js:126`、`src/stores/experiment.js:819` | K003、K013 | 依赖 `recordBlockedReason` 和调零校验保证“当前电压确实对应截止附近”。如果通过 bypass 或异常路径记录，可能把任意反向电压写成截止电压 | B | 建议保留当前流程校验，同时在记录中明确 `measuredByAppliedVoltage` 或增加当前电流阈值字段 |
| B3-006 | P3 | `src/services/dataDiagnostic.js` | `diagnoseData()` | 函数内部计算了 `rSquared`，但返回对象固定 `rSquared: null`：`src/services/dataDiagnostic.js:75`、`src/services/dataDiagnostic.js:110` | K013 | 诊断结果不能被报告或 AI 复用拟合质量数值，只能读文本提示 | B | 建议返回实际拟合质量 |

## 合理简化清单

| 编号 | 文件路径 | 简化内容 | 为什么可接受 | 对应理论条目 | 后续确认点 |
| --- | --- | --- | --- | --- | --- |
| S3-001 | `experiment.js` | `currentCutoffVoltage` 小于 0 时置 0 | 符合低于阈频不发生光电发射的教学模型 | K002、K004 | 需要文案说明“理论截止电压为 0”代表无光电效应 |
| S3-002 | `experiment.js` | 光强按距离平方反比和光阑比例近似 | 符合 K012 的有效光通量教学模型 | K006、K012 | 不应声称为真实标定 |
| S3-003 | `engine.js` | 低能背景事件强制失败 | 用于展示吸收/激发但无法逸出的教学过程，不计入正常收集 | K002、K015 | 文案需保持“失败回落”口径 |
| S3-004 | `engine.js` | 正常电子收集率用窗口统计 | 符合 K010 的概率化宏观电流表达 | K010 | 后续可补充测试确保 cutoff 状态收集率归零 |
| S3-005 | `PhotoelectricAnimationPanel.vue` | 文案使用动画加速度 `px/s²` | 明确是动画单位，不是 SI 真实加速度 | K015 | 保持“动画加速度”表述 |

## 需要后续综合判断的点

1. B3-001 与 B3-002 是同类核心物理口径问题，应在最终报告中合并为“正常阴极光电流不应在反向过截止后变负”。
2. B2-001/B2-002 与 B3-001 有关联：如果正常阴极电流公式修正为非负，动画 cutoff 门控也应同步重新设计。
3. B3-003/B3-004 与报告/AI 数据解释有关，不是光电效应公式错误，但会导致学生看到错误诊断。
4. `IntegratedPlotPanel.vue` 和 `ExperimentReport.vue` 已经在多数地方过滤有限 `cutoffVoltage`，可作为修复数据诊断和 AI 摘要的参考。

## 下一步连续执行计划

下一步将生成综合审查报告草案：

```text
document/physics_review/06_physics_review_summary.md
```

综合范围：

- 汇总第 3-5 轮问题。
- 区分真实物理错误、教学简化、命名/单位风险、诊断数据流问题。
- 给出优先级、证据链和建议修改顺序。
