# new(14) 任务总结文档：光电流、调零流程与量程显示修复

## 1. 本轮任务结论

本轮任务已完成，当前项目已完成以下收口：

1. 修复“接入高频电流线前光电流恒为 0”的问题
2. 将调零流程改为“先调零到 0 附近，再接线，最后确认调零”
3. 将“确认调零”改为锁定当前电流作为零点基准
4. 将“记录数据”改为在零点基准允许误差内才可通过
5. 为基础光电流生成加入约 1% 稳定误差
6. 为每次记录值加入一次独立的 `-3% ~ 3%` 测量误差
7. 将电压显示升级为 3 位小数，最小步进支持到 `0.001`
8. 将确认调零前的电压显示统一隐藏为 `----`
9. 将电流量程真正纳入仪器显示逻辑，不同档位按倍率放大显示
10. 当电流显示超出量程显示范围时，仪器显示 `OF`

本轮没有使用视觉分析组件，全部基于项目代码、状态流和组件调用关系完成。

## 2. 当前问题原因分析

### 2.1 接线前光电流为 0 的旧原因

旧逻辑把 `currentPhotocurrentMicroamp` 直接绑定到“是否已经 ready to measure”。  
只要没有完成接线与调零确认，getter 就不会返回真实实验条件下的光电流，结果就是接线前几乎只剩调零偏置，默认表现接近 0。

### 2.2 接线后仍为 0 的原因

第一次修复时，把“调零基准电流”和“接线后进入试验台的光电流”混成了一层，导致调零时把偏置调到抵消光电流后，接线后仍继续被抵消，看起来还是 0。

### 2.3 接线前电流再次变 0 的原因

第二次修复时，把接线前显示电流简化成了“旋钮补偿量本身”，但没有把仪器自身存在的基准电流加回去。  
因此当 `zeroAdjust = 0` 时，接线前又回到了 `0.0000`。

### 2.4 量程未真正进入显示逻辑的原因

项目原有“电流量程”只参与按钮状态和 3D 旋钮状态，没有参与电流表最终读数换算。  
所以换档后，显示数字几乎不变，不符合实验仪器表现。

## 3. 整体修改方案

本轮采用“最小必要改动 + 统一状态源”的方案，原则如下：

1. 物理电流、仪器基准电流、仪器显示读数分层处理
2. 量程只影响“仪器显示读数”，不污染理论电流与记录值
3. 流程约束统一收敛到 `src/stores/experiment.js`
4. 2D 控制台、2D 主页、3D 屏幕全部只消费 store 的派生状态
5. 阈值、误差、显示规则统一提取为常量或 helper

本轮最终形成了 4 层口径：

1. `generatedPhotocurrentMicroamp`
   - 光电管内部在当前实验条件下生成的真实光电流
2. `instrumentBiasCurrentMicroamp`
   - 仪器本身自带的基准电流
3. `referenceCurrentMicroamp`
   - 基准电流与调零旋钮补偿后的结果
4. `currentMeterDisplayText`
   - 再叠加量程换算和溢出处理后的仪器最终显示值

## 4. 关键改动文件

### 4.1 状态与物理逻辑

1. `src/stores/experiment.js`
2. `src/features/experiment/constants.js`
3. `src/features/experiment/helpers.js`

### 4.2 2D 界面

1. `src/components/IntegratedVirtualConsole.vue`
2. `src/components/VirtualLab.vue`
3. `src/composables/useVirtualLabController.js`
4. `src/components/IntegratedPlotPanel.vue`

### 4.3 3D 界面

1. `src/components/SplineScene.vue`

## 5. 关键代码改动摘要

### 5.1 基础光电流生成

在 `src/stores/experiment.js` 中新增并稳定了以下 getter：

1. `generatedPhotocurrentMicroamp`
   - 在实验条件成立时，根据频率、光强、材料、电压、距离、光阑等生成基础光电流
   - 加入约 1% 的稳定误差

2. `instrumentBiasCurrentMicroamp`
   - 表示仪器本身自带的不可忽略基准电流
   - 当前采用一个小幅稳定扰动模型，保证同一实验上下文中基准值稳定

3. `referenceCurrentMicroamp`
   - `instrumentBiasCurrentMicroamp + zeroAdjust 补偿电流`

4. `currentPhotocurrentMicroamp`
   - 未接高频线时，显示 `referenceCurrentMicroamp`
   - 接入高频线后，显示 `generatedPhotocurrentMicroamp + referenceCurrentMicroamp`

这样就实现了：

1. 接线前不会机械为 0
2. 接线后会把真实光电流送入试验台显示
3. 调零时调的是“仪器自身基准电流”，不是把真实光电流删掉

### 5.2 调零流程重构

调零流程改成了严格顺序：

1. 先把基准电流调到 0 附近
2. 再接入高频线
3. 最后确认调零

对应逻辑：

1. `rearCableConnectionBlockedReason`
   - 未调零到位时不允许接线

2. `zeroConfirmBlockedReason`
   - 未接线时不允许确认
   - 基准电流未归零时不允许确认

3. `confirmZero()`
   - 点击确认时锁定当前电流为 `zeroReferenceCurrent`

### 5.3 记录数据前校验

记录前新增零点基准校验：

1. 未确认调零，不允许记录
2. 若基准值较小，使用绝对误差容差
3. 若基准值非极小，使用 5% 相对误差容差

对应逻辑：

1. `evaluateZeroReference()`
2. `zeroReferenceValidation`
3. `recordBlockedReason`
4. `canRecordData`

### 5.4 两类误差分离

当前项目中两类误差已经分离：

1. 基础生成误差
   - 来源：`generatedPhotocurrentMicroamp`
   - 级别：约 1%
   - 作用：模拟实验条件下基础光电流的不理想性

2. 测量记录误差
   - 来源：`recordData()`
   - 级别：每次 `-3% ~ 3%`
   - 作用：只作用于记录值，不污染基础理论值

### 5.5 电压显示精度与隐藏逻辑

当前电压显示已统一为：

1. 显示格式：3 位小数
2. 步进最小值：`0.001`
3. 确认调零前：统一显示 `----`
4. 确认调零后：显示真实电压

2D 与 3D 已统一接到同一派生 getter：

1. `voltageDisplayText`

### 5.6 电流量程进入显示逻辑

新增 `formatCurrentMeterDisplay(currentMicroamp, currentRange)`，规则如下：

1. 量程为 `10^-13`
   - 直接显示真实电流
2. 量程为 `10^-12`
   - 显示真实电流 `×10`
3. 量程为 `10^-11`
   - 显示真实电流 `×100`
4. 依次类推直到 `10^-8`
5. 若放大后绝对值大于 `9.9999`
   - 显示 `OF`

注意：  
这里只改变仪器显示读数，不改变：

1. 物理电流本身
2. 零点基准判断
3. 记录数据
4. 拟合分析

## 6. 新增或修改的状态变量 / 常量说明

### 6.1 新增状态

1. `apparatus.zeroReferenceCurrent`
   - 确认调零时锁定的零点基准

2. `apparatus.zeroReferenceCapturedAt`
   - 零点基准锁定时间

### 6.2 新增派生状态

1. `generatedPhotocurrentMicroamp`
2. `instrumentBiasCurrentMicroamp`
3. `referenceCurrentMicroamp`
4. `currentMeterDisplayText`
5. `zeroReferenceValidation`
6. `recordBlockedReason`
7. `canRecordData`

### 6.3 新增常量

1. `BASE_PHOTOCURRENT_NOISE_RATIO = 0.01`
2. `MEASUREMENT_CURRENT_NOISE_RANGE = { min: -0.03, max: 0.03 }`
3. `ZERO_CURRENT_TOLERANCE_MICROAMP = 0.05`
4. `ZERO_REFERENCE_RELATIVE_TOLERANCE = 0.05`
5. `ZERO_REFERENCE_ABSOLUTE_TOLERANCE_MICROAMP = 0.05`
6. `CURRENT_DISPLAY_DECIMALS = 4`
7. `CURRENT_DISPLAY_MAX = 9.9999`
8. `CURRENT_DISPLAY_OVERFLOW = 'OF'`

### 6.4 修改常量

1. `VOLTAGE_PRECISION_LEVELS`
   - 增加 `0.001`

2. `ZERO_ADJUST.currentMicroampPerUnit`
   - 调整为更有实际可调性的比例

## 7. 验证步骤

本轮建议按下面顺序验收：

### 7.1 光电流生成

1. 打开仪器电源、汞灯电源、汞灯盖和光电管盖
2. 在未接高频线时观察电流显示
3. 确认接线前显示为“仪器自带基准电流”，不是固定 0
4. 改变滤光片、距离、光阑、材料、电压时，基础光电流逻辑仍可工作

### 7.2 调零流程

1. 未调零到位前尝试接入高频线
2. 确认系统阻止接线并给出提示
3. 将基准电流调到 0 附近
4. 接入高频线
5. 点击确认调零
6. 确认零点基准被锁定

### 7.3 接线后光电流进入试验台

1. 完成上面调零后接线
2. 观察仪器光电流显示
3. 确认接线后显示的是“光电流 + 基准电流”，不再停留在 0

### 7.4 电压显示

1. 确认调零前，电压显示始终为 `----`
2. 确认调零后，电压显示恢复真实值
3. 电压步进改为 `0.001`
4. 按钮、滑块、显示文本和 3D 屏幕统一支持三位小数

### 7.5 数据记录校验

1. 未确认调零前点击记录
2. 应被阻止
3. 确认调零后，在偏差较小时记录
4. 应允许通过
5. 将当前电流调离零点基准超过容差后再记录
6. 应被阻止并有提示

### 7.6 量程显示

1. 将档位调到 `10^-13`
   - 显示原始读数
2. 调到 `10^-12`
   - 显示原值 `×10`
3. 调到 `10^-11`
   - 显示原值 `×100`
4. 依次验证到 `10^-8`
5. 当放大后超出显示范围时
   - 应显示 `OF`

### 7.7 构建验证

本轮结束前已执行：

1. `npm run build`

结果：通过。

## 8. 潜在风险与仍不确定之处

### 8.1 自动扫描与零点基准

当前为了保留原有 I-U 自动扫描主流程，自动扫描仍然主要按“流程已完成”来校验，不像手动截止电压记录那样严格依赖零点基准偏差。  
这属于有意保守处理，避免一次改动把原扫描流程全部打断。

### 8.2 仪器基准电流模型仍是简化模型

目前 `instrumentBiasCurrentMicroamp` 采用的是稳定简化值模型，不是严格从硬件噪声或更复杂仪器电路推导。  
如果后续需要进一步拟真，可以把它扩展为：

1. 与量程相关
2. 与测量模式相关
3. 与时间漂移相关
4. 与环境温度或电源预热相关

### 8.3 量程只作用于显示

当前量程切换只影响“仪器显示读数”，不会改变记录值本身。  
这是本轮有意采用的保守实现，因为这样不会破坏原有拟合、历史记录和实验报告逻辑。  
如果后续要求“记录值也必须是量程后的读数”，需要重新定义数据口径。

### 8.4 电压 4 位显示与大范围模式的矛盾

当前项目同时保留了 `-2V ~ +30V` 模式，而需求里希望电压按“1 位整数 + 3 位小数”风格显示。  
本轮为避免破坏原有量程，实际采用的是“三位小数显示”，没有强行截断为单整数位。

## 9. 本轮最重要的交接结论

如果后续继续开发，最关键的事实如下：

1. `src/stores/experiment.js` 仍然是唯一可信状态源
2. 光电流逻辑现在已经分成“物理电流 / 仪器基准电流 / 仪器显示读数”三层
3. 量程只作用于仪器显示层，不影响记录数据层
4. 调零流程已经被改为显式顺序校验，后续不要再回到“断线后确认调零”的旧逻辑
5. 若后续继续做拟真，优先扩展 `instrumentBiasCurrentMicroamp` 和显示层，而不是直接改记录数据层

## 10. 一句话总结

`new(14)` 当前已经完成“接线前后光电流生成修复 + 调零流程重构 + 零点基准校验 + 电压高精度显示 + 量程真实进入电流表显示逻辑”的一轮完整收口，可作为后续继续做实验细节拟真的稳定基线。
