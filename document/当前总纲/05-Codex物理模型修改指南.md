# 光电效应项目物理模型修改指南（面向 Codex）

> 适用分支：`animation-demo`  
> 项目约束：本项目光电管已固定为**非包围式光电管**，后续不再实现或讨论包围式光电管分支。  
> 目标：让伏安特性、电流饱和点、光强/频率影响、电子动画轨迹、实验记录和报告拟合更接近已上传论文中的真实实验规律。

---

## 0. 修改总目标

当前项目的物理主干是正确的：使用爱因斯坦光电效应方程、通过频率计算截止电压、用光强影响光电流大小。但伏安特性和动画层仍有明显经验化处理，尤其是：

1. 伏安模式下电流在 0 V 附近过早接近饱和。
2. 正常阴极光电流可能被算成负值。
3. 光阑影响按直径线性处理，而论文实验要求按光阑面积处理。
4. 电子动画轨迹没有严格由 `v0、theta、U、L` 决定。
5. 光频率和光强变化时，饱和点没有合理变化。
6. 阳极污染、暗电流、本底电流与真实阴极光电流的分层还不够清晰。

非包围式光电管的伏安曲线应满足：

- 0 V 附近不能直接饱和。
- 必须加一定正向电压才能让更多随机方向逸出的光电子到达阳极。
- 光强更强时，饱和光电流更大，并且达到饱和所需正向电压可以略增大。
- 光频率更高时，电子最大初动能更大，有限阳极收集所有电子通常需要更大正向电场。
- 达到饱和后，电流不应随电压继续明显增加。

---

## 1. 论文依据摘要

### 1.1 非包围式光电管必须加正向电压才会饱和

032《光电效应伏安特性曲线的几个细节问题》明确讨论了包围式与非包围式光电管的区别：非包围式结构中，光电子出射方向随机，部分光电子不能直接到达阳极，需要正向电场“吸引”这些电子到达阳极；当所有可收集电子都到达阳极后，光电流才达到饱和，再增大正向电压不会继续增加光电流。该文还指出，同频率不同光强时，非包围式光电管中强光达到饱和所需正向电压更高，伏安曲线负压区应有拐点。fileciteturn42file2

### 1.2 伏安曲线实验数据支持 10–30 V 才接近饱和

031《光电效应的伏安特性曲线》给出电子运动方程：

\[
\frac12 m_e v_0^2=h\nu_L-W_I
\]

\[
L=v_0t\cos\theta+\frac12\frac{eU}{m_eL}t^2
\]

\[
r=v_0t\sin\theta
\]

并用实验数据拟合伏安曲线。其 557 nm、4 mm 光阑数据中，0 V 电流约为 0.158×10^-10 A，14 V 电流约为 3.230×10^-10 A，说明 0 V 仅约为高正压电流的 5%，远未饱和。fileciteturn42file0

009《光电效应实验教学中饱和光电流与入射光强成正比的实验探讨》使用 ZKY-GD-3 仪器，电压扫描范围为 -2 V 到 30 V，并把 30 V 处“变化相对趋于平缓”的电流近似视为饱和光电流。该文还明确指出光强与光阑面积成正比，并与光源到光电管距离平方成反比。fileciteturn42file13

### 1.3 饱和光电流主要随光强变化，截止电压理论上不随光强变化

029《基于光电效应的普朗克常数的测定与分析》强调，光强表示光子流密度，光越强单位时间内吸收光子的电子数越多，饱和光电流越大；单个电子获得的能量与光强无关，而与频率有关。fileciteturn42file6

021《光电效应实验中阳极光电流的测量》也指出，在保持入射光频率不变时，饱和光电流大小与入射光强成正比；理论截止电压由 `Us = hν/e - W/e` 决定，改变光强时 `Us` 不变。fileciteturn42file3

### 1.4 阳极光电流应作为反向寄生项处理

021 论文把实测电流分成暗电流、阳极光电流和阴极光电流。阳极上可能附着阴极光电材料，并受漫反射光照射产生反向阳极光电流。该文给出近似关系：

\[
i_A'(-u)\approx-\frac{i_K(u)}{K}
\]

即阳极光电流方向与阴极光电流相反，其量级约为阴极光电流的 `1/K`。fileciteturn42file3

### 1.5 暗电流、本底电流、反向电流和数据处理方式会显著影响实验结果

027《光电效应实验的影响因素及误差分析》归纳暗电流、本底电流、反向电流、接触电位差、光源与光电管距离、数据处理方式为主要误差来源，并指出 40 cm 距离、手动选取合适拐点的半自动处理方式可获得较小误差。fileciteturn42file1

018《光电效应实验中截止电压随光强变化原因的探索》指出，测得截止电压随光强变化不是爱因斯坦理论失效，而是测量系统存在附加电流；零电流法测得的是总电流为零时的端电压，并不等于理想阴极光电流为零。该文给出等效电阻/附加电流模型，并指出附加电流与光阑面积有关。fileciteturn42file12

---

## 2. 当前项目中需要修正的关键问题

### 2.1 伏安曲线过早饱和

当前 `src/stores/experiment.js` 和 `src/components/PrincipleAnimationSection.vue` 都使用类似：

```js
current = saturationCurrent * Math.tanh((voltage + cutoff) / 0.45)
```

这会导致在 `V = 0` 时，如果 `cutoff ≈ 1.4 V`，则：

\[
I(0)/I_s=\tanh(1.4/0.45)\approx0.996
\]

即 0 V 附近几乎饱和。这只适合极端理想化的包围式光电管，不符合本项目固定的非包围式光电管。

### 2.2 正常阴极光电流可能为负

上述 `tanh` 公式在 `V < -Vs` 时会给出负的正常阴极光电流。物理上，正常阴极光电流应满足：

\[
I_K(V)\ge0
\]

负读数只能来自阳极寄生光电流、暗电流、本底电流、接触电位差或仪器零漂，不能来自正常阴极光电子流。

### 2.3 光阑影响按直径线性处理

当前 `recalculateIntensity()` 近似使用：

```js
const apertureFactor = this.apparatus.apertureSize / 4
```

这会让 2 mm、4 mm、8 mm 光阑光强比例变为 `1:2:4`。论文实验要求光强与光阑面积成正比，即：

\[
I_s\propto S\propto D^2
\]

所以 2 mm、4 mm、8 mm 应为 `1:4:16`。

### 2.4 电子轨迹没有按论文运动方程闭合

031 论文给出的轨迹由 `v0、theta、U、L` 决定，而项目当前使用多个经验常数：

- `EV_TO_ANIMATION_SPEED`
- `FIELD_TO_ANIMATION_ACCELERATION`
- `VOLT_AMP_INITIAL_SPEED_SCALE`
- `VOLT_AMP_ACCELERATION_SCALE`
- `voltageReferenceMax = 30`

这会导致电子是否返回、是否被收集，不严格由能量守恒和非包围式几何收集决定。

### 2.5 默认材料与教学仪器论文数据对齐

论文中常见 GDh-1、ZKY-GD、DH-GD 等仪器光电阴极多为 Ag-O-K 或复合光电阴极。当前项目将 Ag-O-K 作为初始默认阴极材料，使默认截止电压更接近常见教学仪器表格；Cs 1.95 eV 仍作为可选材料保留。

---

## 3. 建议新增统一物理模型文件

新增文件：

```text
src/features/experiment/photoelectricModel.js
```

该文件作为实验读数、伏安曲线、动画映射、报告拟合的统一物理内核。

### 3.1 推荐实现

```js
import {
  PLANCK_EV,
  PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP,
} from './constants'

export function clamp(value, min, max) {
  return Math.max(min, Math.min(max, value))
}

export function toFiniteNumber(value, fallback = 0) {
  const numeric = Number(value)
  return Number.isFinite(numeric) ? numeric : fallback
}

export function resolvePhotonEnergyEv(frequencyHz) {
  return Math.max(0, PLANCK_EV * toFiniteNumber(frequencyHz))
}

export function resolveMaxKineticEnergyEv({ frequencyHz, workFunctionEv }) {
  return Math.max(0, resolvePhotonEnergyEv(frequencyHz) - toFiniteNumber(workFunctionEv))
}

export function resolveStoppingPotentialV(input) {
  // 以 eV 表示 Kmax 时，数值上 Vs[V] = Kmax[eV]
  return resolveMaxKineticEnergyEv(input)
}

export function resolveNormalizedLightFlux({
  apertureMm,
  distanceCm,
  apertureReferenceMm = 4,
  distanceReferenceCm = 40,
  spectralTransmission = 1,
  manualIntensityRatio = null,
} = {}) {
  if (manualIntensityRatio !== null && manualIntensityRatio !== undefined) {
    return Math.max(0, toFiniteNumber(manualIntensityRatio))
  }

  const aperture = Math.max(0, toFiniteNumber(apertureMm, apertureReferenceMm))
  const distance = Math.max(0.001, toFiniteNumber(distanceCm, distanceReferenceCm))
  const apertureRatio = aperture / Math.max(0.001, apertureReferenceMm)
  const distanceRatio = distanceReferenceCm / distance

  return Math.max(0, spectralTransmission) * apertureRatio ** 2 * distanceRatio ** 2
}

export function resolveSaturationCurrentAmp({
  normalizedFlux,
  quantumEfficiency = 1,
  referenceCurrentAmp = PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP,
  currentLimitAmp = null,
} = {}) {
  const flux = Math.max(0, toFiniteNumber(normalizedFlux))
  const qe = Math.max(0, toFiniteNumber(quantumEfficiency, 1))
  const linearCurrent = Math.max(0, toFiniteNumber(referenceCurrentAmp)) * qe * flux

  if (!currentLimitAmp || currentLimitAmp <= 0) {
    return Number(linearCurrent.toExponential(6))
  }

  const limited = currentLimitAmp * (1 - Math.exp(-linearCurrent / currentLimitAmp))
  return Number(limited.toExponential(6))
}

export function resolveVoltAmpRiseVoltage({
  frequencyHz,
  workFunctionEv,
  referenceFrequencyHz = 5.49e14,
  referenceWorkFunctionEv = workFunctionEv,
  referenceRiseVoltage = 5.6,
  frequencyExponent = 0.35,
  minRiseVoltage = 3.5,
} = {}) {
  const kineticEv = resolveMaxKineticEnergyEv({ frequencyHz, workFunctionEv })
  const referenceKineticEv = Math.max(
    1e-6,
    resolveMaxKineticEnergyEv({
      frequencyHz: referenceFrequencyHz,
      workFunctionEv: referenceWorkFunctionEv,
    }),
  )

  if (kineticEv <= 0) {
    return referenceRiseVoltage
  }

  const kineticFactor = (kineticEv / referenceKineticEv) ** frequencyExponent

  return Math.max(minRiseVoltage, referenceRiseVoltage * kineticFactor)
}

export function resolveNonEnclosingCollectionFraction({
  voltage,
  stoppingPotentialV,
  riseVoltage,
  zeroBiasCollection = 0.05,
  retardingExponent = 1.8,
  positiveShapeExponent = 1,
} = {}) {
  const V = toFiniteNumber(voltage)
  const Vs = Math.max(0, toFiniteNumber(stoppingPotentialV))
  const c0 = clamp(toFiniteNumber(zeroBiasCollection, 0.05), 0, 0.5)

  if (Vs <= 0) {
    return 0
  }

  if (V <= -Vs) {
    return 0
  }

  if (V < 0) {
    const u = clamp((V + Vs) / Vs, 0, 1)
    return c0 * u ** Math.max(0.1, retardingExponent)
  }

  const Ur = Math.max(0.001, toFiniteNumber(riseVoltage, 5.6))
  const m = Math.max(0.1, toFiniteNumber(positiveShapeExponent, 1))
  const positiveCollection = c0 + (1 - c0) * (1 - Math.exp(-((V / Ur) ** m)))
  return clamp(positiveCollection, 0, 1)
}

export function resolveSaturationVoltage({
  saturationRatio = 0.95,
  riseVoltage,
  zeroBiasCollection = 0.05,
  positiveShapeExponent = 1,
} = {}) {
  const c0 = clamp(toFiniteNumber(zeroBiasCollection, 0.05), 0, 0.95)
  const p = clamp(toFiniteNumber(saturationRatio, 0.95), c0 + 1e-6, 0.999)
  const Ur = Math.max(0.001, toFiniteNumber(riseVoltage, 5.6))
  const m = Math.max(0.1, toFiniteNumber(positiveShapeExponent, 1))

  return Ur * (-Math.log((1 - p) / (1 - c0))) ** (1 / m)
}

export function resolveCathodePhotocurrentAmp({
  frequencyHz,
  workFunctionEv,
  voltage,
  normalizedFlux,
  referenceCurrentAmp,
  currentLimitAmp,
  quantumEfficiency = 1,
  model = {},
} = {}) {
  const stoppingPotentialV = resolveStoppingPotentialV({ frequencyHz, workFunctionEv })
  const saturationCurrentAmp = resolveSaturationCurrentAmp({
    normalizedFlux,
    quantumEfficiency,
    referenceCurrentAmp,
    currentLimitAmp,
  })
  const riseVoltage = resolveVoltAmpRiseVoltage({
    frequencyHz,
    workFunctionEv,
    normalizedFlux,
    ...model,
  })
  const collectionFraction = resolveNonEnclosingCollectionFraction({
    voltage,
    stoppingPotentialV,
    riseVoltage,
    zeroBiasCollection: model.zeroBiasCollection,
    retardingExponent: model.retardingExponent,
    positiveShapeExponent: model.positiveShapeExponent,
  })

  return {
    cathodePhotocurrentAmp: Number((saturationCurrentAmp * collectionFraction).toExponential(6)),
    collectionFraction,
    riseVoltage,
    saturationCurrentAmp,
    stoppingPotentialV,
    saturationVoltage95: resolveSaturationVoltage({
      saturationRatio: 0.95,
      riseVoltage,
      zeroBiasCollection: model.zeroBiasCollection,
      positiveShapeExponent: model.positiveShapeExponent,
    }),
    saturationVoltage99: resolveSaturationVoltage({
      saturationRatio: 0.99,
      riseVoltage,
      zeroBiasCollection: model.zeroBiasCollection,
      positiveShapeExponent: model.positiveShapeExponent,
    }),
  }
}

export function resolveAnodeParasiticPhotocurrentAmp({
  voltage,
  cathodeForwardCurrentAtOppositeVoltageAmp,
  contaminationRatio = 0,
  strayLightRatio = 0,
  cathodeToAnodeLightRatioK = 60,
} = {}) {
  const V = toFiniteNumber(voltage)
  if (V >= 0) {
    return 0
  }

  const contamination = clamp(toFiniteNumber(contaminationRatio), 0, 1)
  const stray = clamp(toFiniteNumber(strayLightRatio), 0, 1)
  const K = Math.max(1, toFiniteNumber(cathodeToAnodeLightRatioK, 60))
  const forwardEquivalent = Math.max(0, toFiniteNumber(cathodeForwardCurrentAtOppositeVoltageAmp))

  return Number((-(forwardEquivalent / K) * contamination * stray).toExponential(6))
}

export function samplePhotoelectronKineticEnergyEv(maxKineticEnergyEv, random = Math.random) {
  const Kmax = Math.max(0, toFiniteNumber(maxKineticEnergyEv))
  return Kmax * clamp(random(), 0, 1)
}

export function resolveEnergyConsistentMotion({
  kineticEnergyEv,
  emissionAngleRadians = 0,
  voltage,
  plateDistancePx,
  speedScalePxPerSPerSqrtEv = 360,
} = {}) {
  const K = Math.max(0, toFiniteNumber(kineticEnergyEv))
  const theta = toFiniteNumber(emissionAngleRadians)
  const V = toFiniteNumber(voltage)
  const d = Math.max(1, toFiniteNumber(plateDistancePx, 1))
  const S = Math.max(1, toFiniteNumber(speedScalePxPerSPerSqrtEv, 360))

  const speedPxPerS = S * Math.sqrt(K)
  const vx = speedPxPerS * Math.cos(theta)
  const vy = speedPxPerS * Math.sin(theta)

  // 令 eV 的能量差在动画坐标中与 0.5*a*d 对应，保持截止/返回条件一致。
  const accelerationPxPerS2 = Math.sign(V) * (S ** 2 * Math.abs(V)) / (2 * d)

  return {
    accelerationPxPerS2,
    longitudinalKineticEnergyEv: K * Math.cos(theta) ** 2,
    speedPxPerS,
    vx,
    vy,
  }
}
```

---

## 4. 修改 `constants.js`

路径：

```text
src/features/experiment/constants.js
```

### 4.1 默认使用复合光电阴极并保留 Cs

保留原有材料和论文复现实验用的复合光电阴极材料：

```js
export const MATERIALS = {
  agOK: { name: '银氧钾复合光电阴极 (Ag-O-K)', workFunction: 1.65, color: '#f6d54a' },
  cesium: { name: '铯 (Cs)', workFunction: 1.95, color: '#ffd700' },
  potassium: { name: '钾 (K)', workFunction: 2.3, color: '#c0c0c0' },
  sodium: { name: '钠 (Na)', workFunction: 2.75, color: '#e8e8e8' },
  zinc: { name: '锌 (Zn)', workFunction: 4.33, color: '#7a7a7a' },
  copper: { name: '铜 (Cu)', workFunction: 4.65, color: '#b87333' },
}
```

初始默认材料使用 `agOK`，`cesium` 作为可选材料。

### 4.2 新增非包围式光电管模型参数

```js
export const NON_ENCLOSING_PHOTOTUBE_MODEL = {
  zeroBiasCollection: 0.05,
  referenceRiseVoltage: 5.6,
  referenceFrequencyHz: 5.49e14,
  frequencyExponent: 0.35,
  retardingExponent: 1.8,
  positiveShapeExponent: 1,
  saturationRatioForUi: 0.95,
  saturationRatioForNearFlat: 0.99,
  minRiseVoltage: 3.5,
}
```

### 4.3 新增电流预设

当前 `PHOTOCURRENT_SATURATION_CURRENT_MAX_AMP = 20e-13` 更像动画演示值，不适合复现 031/009 论文中的 `10^-10 A` 量级。建议新增：

```js
export const PHOTOCURRENT_PRESETS = {
  normalizedTeaching: {
    key: 'normalizedTeaching',
    label: '归一化教学演示',
    referenceCurrentAmp: 20e-13,
    currentLimitAmp: null,
  },
  zkyGd3Like: {
    key: 'zkyGd3Like',
    label: 'ZKY-GD-3 论文数据近似',
    referenceCurrentAmp: 1.0e-10,
    currentLimitAmp: 3.5e-10,
  },
}
```

若暂时不做 UI 预设，先在伏安模式中使用 `zkyGd3Like`，截止电压演示可保留 `normalizedTeaching`。

---

## 5. 修改 `helpers.js` 和默认状态

路径：

```text
src/features/experiment/helpers.js
```

### 5.1 默认材料改为 `agOK`

```js
params: {
  voltage: -1.998,
  frequency: FILTERS[0].frequency * 1e14,
  intensity: 50,
  material: 'agOK',
  mode: 'basic',
}
```

### 5.2 增加仪器预设状态

在 `apparatus` 中新增：

```js
photocurrentPreset: 'zkyGd3Like',
```

或如果暂时不想改 UI，先写死在模型调用处。

---

## 6. 修改 `experiment.js`

路径：

```text
src/stores/experiment.js
```

### 6.1 引入新模型函数

```js
import {
  resolveCathodePhotocurrentAmp,
  resolveNormalizedLightFlux,
} from '../features/experiment/photoelectricModel'
```

并引入：

```js
import {
  NON_ENCLOSING_PHOTOTUBE_MODEL,
  PHOTOCURRENT_PRESETS,
} from '../features/experiment/constants'
```

### 6.2 新增物理光通量 getter

```js
physicalLightFluxRatio(state) {
  if (!state.apparatus.lampPowerOn) {
    return 0
  }

  return resolveNormalizedLightFlux({
    apertureMm: state.apparatus.apertureSize,
    distanceCm: state.apparatus.photocellDistance,
    apertureReferenceMm: 4,
    distanceReferenceCm: 40,
  })
}
```

### 6.3 替换 `generatedPhotocurrentAmp()`

替换当前 `tanh` 公式。建议实现：

```js
generatedPhotocurrentAmp() {
  if (!this.isInstrumentPowered || !this.photoelectricConditionsReady) {
    return 0
  }

  const preset = PHOTOCURRENT_PRESETS[this.apparatus.photocurrentPreset]
    || PHOTOCURRENT_PRESETS.zkyGd3Like

  const result = resolveCathodePhotocurrentAmp({
    frequencyHz: this.params.frequency,
    workFunctionEv: this.materialConfig.workFunction,
    voltage: Number(this.params.voltage),
    normalizedFlux: this.physicalLightFluxRatio,
    referenceCurrentAmp: preset.referenceCurrentAmp,
    currentLimitAmp: preset.currentLimitAmp,
    model: NON_ENCLOSING_PHOTOTUBE_MODEL,
  })

  const noiseRatio = Number(this.photocurrentNoiseRatio || 0)
  const adjusted = Math.max(0, result.cathodePhotocurrentAmp * (1 + noiseRatio))
  return Number(adjusted.toExponential(6))
}
```

要求：正常阴极光电流必须始终 `>= 0`。

### 6.4 修改 `recalculateIntensity()`

当前 UI 的 `params.intensity` 可以继续作为 0–100 显示值，但光阑必须按面积处理：

```js
recalculateIntensity() {
  if (!this.apparatus.lampPowerOn) {
    this.params.intensity = 0
    return
  }

  const distanceFactor = Math.pow(40 / this.apparatus.photocellDistance, 2)
  const apertureFactor = Math.pow(this.apparatus.apertureSize / 4, 2)
  const intensity = 50 * distanceFactor * apertureFactor
  this.params.intensity = Math.min(100, Math.max(1, Math.round(intensity)))
}
```

注意：`params.intensity` 只是 UI 百分比；真实电流应使用 `physicalLightFluxRatio`，不要因为 `params.intensity` 被 clamp 到 100 而丢失 8 mm 与 4 mm 的真实面积差异。

---

## 7. 修改 `PrincipleAnimationSection.vue`

路径：

```text
src/components/PrincipleAnimationSection.vue
```

### 7.1 引入模型函数

```js
import {
  resolveCathodePhotocurrentAmp,
} from '../features/experiment/photoelectricModel'

import {
  NON_ENCLOSING_PHOTOTUBE_MODEL,
  PHOTOCURRENT_PRESETS,
} from '../features/experiment/constants'
```

### 7.2 替换 `trueCathodePhotocurrentAmp`

当前公式同样使用 `tanh`，需要替换：

```js
const trueCathodePhotocurrentAmp = computed(() => {
  if (!photoelectricConditionsReady.value) {
    return 0
  }

  const preset = PHOTOCURRENT_PRESETS.zkyGd3Like
  const normalizedFlux = Math.max(0, animationControls.value.intensity / 50)

  const result = resolveCathodePhotocurrentAmp({
    frequencyHz: frequency.value,
    workFunctionEv: materialConfig.value.workFunction,
    voltage: animationControls.value.voltage,
    normalizedFlux,
    referenceCurrentAmp: preset.referenceCurrentAmp,
    currentLimitAmp: preset.currentLimitAmp,
    model: NON_ENCLOSING_PHOTOTUBE_MODEL,
  })

  return result.cathodePhotocurrentAmp
})
```

### 7.3 显示饱和点指标

建议新增只读指标：

```js
const voltAmpModelResult = computed(() => {
  const preset = PHOTOCURRENT_PRESETS.zkyGd3Like
  const normalizedFlux = Math.max(0, animationControls.value.intensity / 50)

  return resolveCathodePhotocurrentAmp({
    frequencyHz: frequency.value,
    workFunctionEv: materialConfig.value.workFunction,
    voltage: animationControls.value.voltage,
    normalizedFlux,
    referenceCurrentAmp: preset.referenceCurrentAmp,
    currentLimitAmp: preset.currentLimitAmp,
    model: NON_ENCLOSING_PHOTOTUBE_MODEL,
  })
})
```

UI 文案建议：

```text
95% 饱和电压：{{ voltAmpModelResult.saturationVoltage95.toFixed(1) }} V
99% 近饱和电压：{{ voltAmpModelResult.saturationVoltage99.toFixed(1) }} V
```

不要把 `U95` 与截止电压混淆。

---

## 8. 修改 `mapping.js`

路径：

```text
src/features/photoelectric-animation/mapping.js
```

### 8.1 返回字段改名

当前返回：

```js
cutoffVoltage: kineticEnergyEv
```

建议保留兼容字段，但新增清晰字段：

```js
stoppingPotentialV: kineticEnergyEv,
maxKineticEnergyEv: kineticEnergyEv,
cutoffVoltage: kineticEnergyEv, // compatibility only
```

### 8.2 伏安模式收集率改为非包围式模型

`resolveVoltAmpModeCollection()` 不要再只依赖 `estimateVoltAmpCollectionRatio()` 的经验比例。建议改为：

```js
import {
  resolveCathodePhotocurrentAmp,
  resolveNormalizedLightFlux,
} from '../experiment/photoelectricModel'
import {
  NON_ENCLOSING_PHOTOTUBE_MODEL,
  PHOTOCURRENT_PRESETS,
} from '../experiment/constants'
```

在 `resolvePhotoelectricAnimationState()` 中计算：

```js
const normalizedFluxForModel = Math.max(0, normalizedIntensity / 0.5)
const preset = PHOTOCURRENT_PRESETS.zkyGd3Like
const cathodeModel = resolveCathodePhotocurrentAmp({
  frequencyHz: frequency,
  workFunctionEv,
  voltage,
  normalizedFlux: normalizedFluxForModel,
  referenceCurrentAmp: preset.referenceCurrentAmp,
  currentLimitAmp: preset.currentLimitAmp,
  model: NON_ENCLOSING_PHOTOTUBE_MODEL,
})
```

然后在伏安模式中使用：

```js
collectorSuccessRatio: cathodeModel.collectionFraction,
voltAmpRiseVoltage: cathodeModel.riseVoltage,
voltAmpSaturationVoltage95: cathodeModel.saturationVoltage95,
voltAmpSaturationVoltage99: cathodeModel.saturationVoltage99,
```

### 8.3 当前电流层级

若 `trueCathodePhotocurrentAmp` 有显式输入，优先使用输入；否则使用模型结果。

```js
const trueCathodePhotocurrentAmp = hasTrueCathodeCurrentInput
  ? trueCathodeInput
  : cathodeModel.cathodePhotocurrentAmp
```

要求：

```js
trueCathodePhotocurrentAmp >= 0
```

### 8.4 阳极污染建议改为 021 模型

保留当前视觉开关，但数值应接近：

\[
I_A(-u)\approx-I_K(u)/K
\]

实现时可在 `voltage < 0` 时取 `oppositeVoltage = Math.abs(voltage)`，计算阴极在 `+|V|` 下的电流，然后乘污染和漏光比例：

```js
const oppositeCathodeModel = resolveCathodePhotocurrentAmp({
  frequencyHz: frequency,
  workFunctionEv,
  voltage: Math.abs(voltage),
  normalizedFlux: normalizedFluxForModel,
  referenceCurrentAmp: preset.referenceCurrentAmp,
  currentLimitAmp: preset.currentLimitAmp,
  model: NON_ENCLOSING_PHOTOTUBE_MODEL,
})

const parasiticAnodePhotocurrentAmp = resolveAnodeParasiticPhotocurrentAmp({
  voltage,
  cathodeForwardCurrentAtOppositeVoltageAmp: oppositeCathodeModel.cathodePhotocurrentAmp,
  contaminationRatio: anodeContaminationRatio,
  strayLightRatio: anodeStrayLightRatio,
  cathodeToAnodeLightRatioK: 60,
})
```

---

## 9. 修改 `voltAmpTrajectory.js`

路径：

```text
src/features/photoelectric-animation/voltAmpTrajectory.js
```

### 9.1 删除或弱化经验电压比例

当前：

```js
const DEFAULT_REFERENCE_VOLTAGE = 30
const VOLT_AMP_INITIAL_SPEED_SCALE = 72
const VOLT_AMP_ACCELERATION_SCALE = 4200
```

建议改为能量一致模型：

```js
const DEFAULT_PLATE_DISTANCE_PX = 560
const VOLT_AMP_SPEED_SCALE = 360
```

### 9.2 采样动能而不是速度倍率大于 1

避免 `K > Kmax`。推荐：

```js
const ENERGY_FACTORS = Object.freeze([0.08, 0.18, 0.35, 0.6, 0.85, 1.0])
```

每个样本：

```js
const sampleKineticEnergyEv = kineticEnergyEv * energyFactor
```

### 9.3 收集条件

非包围式几何下，先计算运动是否可到达阳极，再检查阳极窗口。

反向电压时，应满足：

\[
K_x + V > 0
\]

其中：

\[
K_x = K\cos^2\theta
\]

用 eV 和 V 表示时可直接相加。

建议：

```js
const longitudinalKineticEnergyEv = sampleKineticEnergyEv * Math.cos(angleRadians) ** 2
const passesRetardingField = voltage >= 0 || longitudinalKineticEnergyEv + voltage > 0
```

只有 `passesRetardingField` 为真时才可能到达阳极。

---

## 10. 修改 `physics.js`

路径：

```text
src/features/photoelectric-animation/physics.js
```

### 10.1 保留教学字段，但不要伪装成真实 SI 加速度

当前 `calculateElectronAcceleration()` 中的 `accelerationMagnitudeProxy` 使用 `V/px` 推出 `N/kg`，这不是 SI 真实加速度。建议：

- 保留 `animationAccelerationPxPerS2`。
- 删除或改名 `accelerationMagnitudeProxy` 为 `animationAccelerationProxy`。
- 新增 `resolveEnergyConsistentMotion()` 后，动画轨迹优先用它。

### 10.2 初速度函数不要外部再乘 2

如果 `resolveInitialVelocityFromKineticEnergy()` 仍保留，应明确它返回的是最大速度比例，不应再乘大于 1 的倍率。

---

## 11. 修改 `engine.js`

路径：

```text
src/features/photoelectric-animation/engine.js
```

### 11.1 删除大于 1 的速度随机倍率

当前：

```js
const ELECTRON_INITIAL_SPEED_FACTOR_MIN = 1
const ELECTRON_INITIAL_SPEED_FACTOR_MAX = 2
```

建议替换为动能采样。若短期内不重构采样，则至少改成：

```js
const ELECTRON_INITIAL_SPEED_FACTOR_MIN = 0.25
const ELECTRON_INITIAL_SPEED_FACTOR_MAX = 1
```

更推荐使用：

```js
samplePhotoelectronKineticEnergyEv(maxKineticEnergyEv)
```

### 11.2 动画飞行状态使用能量一致加速度

在伏安模式中，电子生成时应包含：

```js
sampleKineticEnergyEv
emissionAngleRadians
longitudinalKineticEnergyEv
```

并由 `resolveEnergyConsistentMotion()` 给出 `vx, vy, accelerationPxPerS2`。

### 11.3 0 V 附近的收集动画必须稀疏

非包围式光电管默认 `zeroBiasCollection = 0.05`。因此 0 V 附近动画中到达阳极的正常光电子比例应很低，不能大多数电子都立即到达阳极。

---

## 12. 修改图表和 UI 文案

### 12.1 伏安模式显示字段

建议在伏安模式指标区显示：

```text
饱和电流 I∞
当前收集率 C(U)
95% 饱和电压 U95
99% 近饱和电压 U99
理论截止电压 Vs
```

### 12.2 明确区分三个电压

- `Vs`：截止电压/遏止电压，由频率和逸出功决定。
- `U95`：达到 95% 饱和电流的正向电压。
- `U99`：达到 99% 近饱和电流的正向电压。

不要把正向饱和点写成截止电压。

### 12.3 伏安曲线参考行为

默认非包围式模型应表现为：

| 条件 | 预期行为 |
|---|---|
| `V <= -Vs` | 正常阴极光电流为 0 |
| `V = 0` | 约 5% 饱和电流，不饱和 |
| `V = 1 V` | 电流明显上升但远未饱和 |
| `V = 10–18 V` | 接近 90–95% 饱和 |
| `V = 24–30 V` | 接近 99% 饱和 |

---

## 13. 实验记录与报告模块建议

### 13.1 记录数据增加字段

路径：

```text
src/stores/experiment.js -> recordData()
```

建议在 record 中加入：

```js
normalCathodePhotocurrentAmp,
anodeParasiticPhotocurrentAmp,
darkCurrentAmp,
backgroundCurrentAmp,
measuredPhotocurrentAmp,
collectionFraction,
saturationCurrentAmp,
voltAmpRiseVoltage,
saturationVoltage95,
saturationVoltage99,
cutoffVoltageSource,
cutoffQuality,
```

### 13.2 截止电压记录不能直接等于任意负电压

当前 `measureCutoffVoltageFromAppliedVoltage()` 可保留为手动输入辅助，但记录参与拟合时必须带质量标记：

```js
cutoffQuality: 'valid' | 'suspect' | 'invalid'
```

只有当净阴极电流接近 0，或由拐点法/外推法/拟合法得到，才应作为 `valid` 数据参与普朗克常量拟合。

### 13.3 3D 电流显示屏口径

3D 区的仪器状态应独立于“原理动画”局部控制状态，但电流计算口径保持一致：

```js
measuredPhotocurrentAmp = normalCathodePhotocurrentAmp + anodeParasiticPhotocurrentAmp
currentPhotocurrentAmp = measuredPhotocurrentAmp + referenceCurrentAmp
```

要求：

- 正常阴极光电流始终非负。
- 阳极寄生电流在反向电压、阳极污染和杂散光条件满足时可为负。
- 3D 区暂不把暗电流和本底电流计入仪器屏读数，二者在记录字段中保留为 0。
- 3D 微观小窗应复用 experiment store 中的 `anodeParasiticPhotocurrentAmp` 与 `measuredPhotocurrentAmp`，不要维护另一套本地寄生电流公式。

---

## 14. 单元测试建议

新增：

```text
src/features/experiment/photoelectricModel.test.js
```

### 14.1 正常阴极电流非负

```js
expect(result.cathodePhotocurrentAmp).toBeGreaterThanOrEqual(0)
```

测试电压包括：`-2, -1, -0.5, 0, 1, 10, 30`。

### 14.2 0 V 不饱和

```js
const result = resolveCathodePhotocurrentAmp({ ...voltage: 0 })
expect(result.collectionFraction).toBeCloseTo(0.05, 2)
```

### 14.3 95% 饱和点在合理区间

参考 546.1 nm、Ag-O-K、4 mm、40 cm：

```js
expect(result.saturationVoltage95).toBeGreaterThan(12)
expect(result.saturationVoltage95).toBeLessThan(20)
```

### 14.4 99% 近饱和点接近 30 V

```js
expect(result.saturationVoltage99).toBeGreaterThan(22)
expect(result.saturationVoltage99).toBeLessThan(35)
```

### 14.5 光阑面积比例

```js
const g2 = resolveNormalizedLightFlux({ apertureMm: 2, distanceCm: 40 })
const g4 = resolveNormalizedLightFlux({ apertureMm: 4, distanceCm: 40 })
const g8 = resolveNormalizedLightFlux({ apertureMm: 8, distanceCm: 40 })
expect(g4 / g2).toBeCloseTo(4)
expect(g8 / g4).toBeCloseTo(4)
```

### 14.6 理论截止电压不随光强变化

```js
const low = resolveStoppingPotentialV({ frequencyHz, workFunctionEv })
const high = resolveStoppingPotentialV({ frequencyHz, workFunctionEv })
expect(low).toBe(high)
```

### 14.7 饱和电流随光强增大

```js
expect(highFlux.saturationCurrentAmp).toBeGreaterThan(lowFlux.saturationCurrentAmp)
```

### 14.8 频率升高时饱和点上升或不下降

```js
expect(highFrequency.saturationVoltage95).toBeGreaterThanOrEqual(lowFrequency.saturationVoltage95)
```

### 14.9 饱和点不受光强影响且不设置人为上限

```js
expect(highFlux.saturationVoltage95).toBeCloseTo(lowFlux.saturationVoltage95, 12)
expect(extremeFrequency.riseVoltage).toBeGreaterThan(8)
```

---

## 15. 推荐修改顺序

### Phase 1：物理内核

1. 新建 `photoelectricModel.js`。
2. 新增 `photoelectricModel.test.js`。
3. 确认所有模型函数测试通过。

### Phase 2：实验读数

1. 修改 `constants.js`：新增 `agOK`、`NON_ENCLOSING_PHOTOTUBE_MODEL`、`PHOTOCURRENT_PRESETS`。
2. 修改 `helpers.js`：默认材料改为 `agOK`。
3. 修改 `experiment.js`：替换 `tanh` 电流模型；修正光阑面积。

### Phase 3：原理动画和映射

1. 修改 `PrincipleAnimationSection.vue`：替换 `trueCathodePhotocurrentAmp`。
2. 修改 `mapping.js`：增加 `stoppingPotentialV`、`saturationVoltage95/99`、`voltAmpRiseVoltage`。
3. 保证动画状态和真实实验状态都从统一模型派生。

### Phase 4：轨迹动画

1. 修改 `voltAmpTrajectory.js`：移除大于 1 的速度倍率，改用动能采样。
2. 修改 `physics.js`：新增或引用能量一致运动模型。
3. 修改 `engine.js`：用采样动能和出射角生成伏安电子轨迹。

### Phase 5：报告与数据记录

1. `recordData()` 增加模型字段。
2. `buildAnalysisSummary()` 只使用 `cutoffQuality === 'valid'` 的截止电压。
3. UI 提示“理论截止电压”和“测得零电流点”不同。

---

## 16. 验收标准

Codex 完成修改后，至少满足：

1. 伏安模式下 `V = 0` 时正常阴极光电流约为饱和电流的 3%–8%，不能接近 100%。
2. `V = 10–18 V` 时接近 90%–95% 饱和。
3. `V = 24–30 V` 时接近 99% 近饱和。
4. 2/4/8 mm 光阑的物理光强比例为 1:4:16。
5. 正常阴极光电流任何情况下都不为负。
6. 负电流只能来自阳极寄生项、暗电流、本底电流或仪器偏置。
7. 理论截止电压只随频率和材料功函数变化，不随光强变化。
8. 光强增大时，饱和电流增大，但 `U95/U99` 不因光强改变。
9. 频率增大时，光电子最大初动能增大，`U95` 应上升或至少不下降，且不设置人为最大饱和电压上限。
10. 动画中电子最大动能不得超过 `Kmax = hν - φ`。

---

## 17. 不建议做的事

1. 不要继续使用 `tanh((V + cutoff) / 0.45)` 作为伏安电流主模型。
2. 不要让正常阴极光电流出现负值。
3. 不要按光阑直径线性改变光强。
4. 不要把 0 V 附近画成饱和。
5. 不要把正向饱和电压 `U95/U99` 与反向截止电压 `Vs` 混用。
6. 不要在伏安模式中强制隐藏所有误差项；应允许理想曲线、实测曲线、校正曲线并存。
7. 不要让动画速度随机倍率导致电子能量超过 `Kmax`。

---

## 18. 最小可交付改动清单

若时间有限，Codex 至少完成以下 5 项：

1. 新增 `photoelectricModel.js`。
2. 用 `resolveCathodePhotocurrentAmp()` 替换 `experiment.js` 和 `PrincipleAnimationSection.vue` 中的 `tanh` 电流公式。
3. 把光阑因子从 `apertureSize / 4` 改为 `(apertureSize / 4) ** 2`。
4. 在 UI 和动画状态中新增 `saturationVoltage95`、`saturationVoltage99`。
5. 新增单元测试验证：0 V 不饱和、30 V 近饱和、阴极光电流非负、光阑面积比例正确。

完成这 5 项后，项目的伏安特性曲线会从“0 V 附近饱和的包围式近似”转向“正向高压逐渐饱和的非包围式光电管模型”，与上传论文中的实验数据和物理解释明显更一致。
