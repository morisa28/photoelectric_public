# 阶段文档22-24-new(26)-AgOK阴极颜色接近Cs

## 1. 阶段目标

根据用户要求，将 Ag-O-K 材料下的阴极显示颜色调整为接近 Cs 的颜色。

当前 Cs 颜色为：

```text
#ffd700
```

本阶段将 Ag-O-K 颜色调整为：

```text
#f6d54a
```

该颜色与 Cs 同属金黄色系，但不是完全相同，便于在界面上仍能分辨 Ag-O-K 与 Cs。

## 2. 修改内容

1. 更新实验材料表中的 Ag-O-K 颜色。
2. 更新原理动画材料颜色映射中的 Ag-O-K 颜色。
3. 更新物理修改指导文档中的材料表示例，避免后续按旧颜色恢复。

## 3. 影响范围

- 只影响 Ag-O-K 材料的阴极可视化颜色。
- 不改变 Ag-O-K 的逸出功。
- 不改变 Cs 的颜色。
- 不改变任何电流、截止电压、阳极寄生或 A 路径阴极尾流计算。

## 4. 验证结果

已运行：

```bash
npm run test:unit -- --run src/features/photoelectric-animation/mapping.spec.js src/stores/experiment.spec.js
```

结果：

```text
Test Files  2 passed
Tests       30 passed
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
