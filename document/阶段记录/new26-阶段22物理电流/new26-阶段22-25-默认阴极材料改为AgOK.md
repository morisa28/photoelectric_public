# 阶段文档22-25-new(26)-默认阴极材料改为AgOK

## 1. 阶段目标

根据用户要求，将项目默认阴极材料从 Cs 改为 Ag-O-K。

本阶段延续前两个阶段的材料校准结果：

```text
Ag-O-K 逸出功：1.65 eV
Ag-O-K 阴极颜色：#f6d54a
```

## 2. 修改内容

1. 实验主默认状态改为：

```text
params.material = agOK
```

2. 实验主状态的材料 fallback 改为 Ag-O-K。

3. 原理动画默认控制材料改为 Ag-O-K。

4. 原理动画材料 fallback 改为 Ag-O-K。

5. 更新默认材料相关单元测试：

```text
初始材料：agOK
默认逸出功：1.65 eV
```

6. 更新当前物理框架说明、物理协作者版说明和物理修改指导文档中的默认材料口径。

## 3. 影响范围

- 2D 实验台启动后默认材料为 Ag-O-K。
- 3D 区读取实验主 store，因此启动后默认材料也为 Ag-O-K。
- 原理动画独立控制区启动后默认材料为 Ag-O-K。
- Cs 仍保留在材料列表中，可手动切换。
- 本阶段不改变 Ag-O-K 的逸出功、颜色、电流公式、阳极寄生公式或截止模式 A 路径。

## 4. 物理影响

默认 365 nm 场景下，理论截止电压将从 Cs 的约 `1.45 V` 转为 Ag-O-K 的约 `1.75 V`。这更接近用户提供的常见教学仪器实验表格区间。

## 5. 验证结果

已运行：

```bash
npm run test:unit -- --run src/stores/experiment.spec.js src/components/__tests__/PrincipleAnimationSection.spec.js src/components/__tests__/ThreeDMicroAnimationPanel.spec.js src/features/photoelectric-animation/mapping.spec.js
```

结果：

```text
Test Files  4 passed
Tests       45 passed
```

已运行：

```bash
npm run build
```

结果：

```text
构建通过。
Vite 仍提示部分 chunk 大于 500 kB，这是既有构建体积警告。
```
