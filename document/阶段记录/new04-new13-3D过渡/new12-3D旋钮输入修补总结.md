# new(12) 本轮总结文档：3D 旋钮输入修补与视角拖动分流

## 1. 本轮处理的实际问题

用户手动测试后，当前项目暴露出三个直接问题：

1. 普通左键同时承担了视角拖动和旋钮拖拽，交互冲突
2. 电流量程旋钮、调零旋钮拖动无反应
3. 无法直观看到鼠标是否已经命中可操作区域，排查成本高

本轮目标不是继续扩展 3D 功能，而是把输入层先修到“能稳定用、能稳定查”。

---

## 2. 本轮最终改动文件

本轮最终只改了一个文件：

- `src/components/SplineScene.vue`

原因是这轮问题全部集中在 3D 场景输入分流、命中检测、悬停反馈和诊断桥接上，不需要再改 store、2D 控制器或常量层。

---

## 3. 这次最终做成了什么

### 3.1 鼠标输入分流

现在的输入规则已经改成：

1. 普通左键：
   - 不再用于 orbit 视角拖动
   - 只用于按钮点击和旋钮拖拽
2. `Alt + 左键`：
   - 保留 orbit 视角拖动
3. 中键：
   - 不受 `Alt` 限制
   - orbit 保持可用
4. 滚轮：
   - 不受 `Alt` 限制

也就是说，本轮最终落地的是“只限制左键的视角拖动功能”，而不是把整个 orbit controls 都绑到 `Alt` 上。

### 3.2 旋钮拖拽恢复

量程旋钮和调零旋钮拖拽已经恢复可用。

拖拽链路现在分成两层：

1. 第一层：优先使用 runtime 内部 raycaster 命中 `range_plane` / `zero_plane`
2. 第二层：如果当前视图下射线仍然打不到透明面，就退回到屏幕热点兜底方案

这意味着当前版本已经不再依赖 Spline `mouseDown` 事件本身是否可靠命中透明拖拽面。

### 3.3 悬停反馈与诊断增强

为了方便继续排查，这轮又补了两类诊断：

1. 鼠标掠过可操作区时，光标会变成手型
2. 诊断面板新增：
   - `Alt 视角键`
   - `轨道控制`
   - `悬停透明面`

另外还把下面两个调试接口挂进了 `window.__VISUAL_INSPECTOR__`：

1. `debugRaycastAt(x, y)`
2. `projectObjectToScreen(name)`

后续如果还要继续查 3D 命中问题，可以直接复用这两个桥接方法。

---

## 4. 为什么最初拖拽一直没反应

本轮最重要的排查结论有两个：

1. 诊断面板里 `eventTargetCount` 只有 `1`
2. 用 runtime 内部 raycaster 对控制区做扫描时，绝大多数点命中的是：
   - `Room`
   - 少量机箱或台面对象
   - 基本打不到 `range_plane` / `zero_plane`
   - 也基本打不到两个旋钮对象

因此，之前“拖动无反应”并不是后台状态链路没接上，而是前端输入命中层本身不稳定：

1. Spline 事件对象数量明显不足
2. 透明拖拽面在当前 runtime 下几乎不可稳定射线命中
3. 单靠 `mouseDown -> target.name === range_plane/zero_plane` 这条路不够可靠

所以本轮最终没有继续死磕 Spline 事件，而是改成了“射线优先 + 屏幕热点兜底”的双层命中方案。

---

## 5. 当前屏幕热点兜底参数

当前兜底热点定义在 `src/components/SplineScene.vue`，参数如下：

1. 电流量程旋钮：
   - `centerXRatio: 0.358`
   - `centerYRatio: 0.694`
   - `radiusX: 56`
   - `radiusY: 52`
2. 调零旋钮：
   - `centerXRatio: 0.359`
   - `centerYRatio: 0.821`
   - `radiusX: 54`
   - `radiusY: 50`

这套参数是基于当前默认正视图、当前画布比例和当前视觉位置校出来的。

---

## 6. 本轮实际验证过什么

### 6.1 构建验证

执行：

```bash
npm run build
```

结果：

1. 构建通过
2. 本轮输入层修补没有引入新的编译错误

### 6.2 程序化交互验证：热点命中与拖拽生效

通过本地 Playwright 验证过以下结果：

1. 悬停量程区后：
   - `hoveredSurface = range_plane`
2. 拖动量程区后：
   - `currentRange` 从 `1e-8` 变成 `1e-10`
3. 悬停调零区后：
   - `hoveredSurface = zero_plane`
4. 拖动调零区后：
   - `zeroAdjust` 从 `0` 变成 `0.4`
   - `currentPhotocurrentMicroamp` 变成 `0.0004`

说明：

1. 悬停反馈已经能识别两个可操作区
2. 量程旋钮和调零旋钮两条后台链路都已恢复

### 6.3 程序化交互验证：左键 / 中键 / Alt+左键 分流

同样通过 Playwright 验证过：

1. 初始：
   - `orbitEnabled = true`
2. 中键按下：
   - `orbitEnabled = true`
3. 普通左键按下：
   - `orbitEnabled = false`
4. `Alt + 左键` 按下：
   - `orbitEnabled = true`

这说明当前输入规则已经符合用户要求：

1. 只限制左键的视角拖动
2. 中键与滚轮不受 `Alt` 限制

---

## 7. 本轮新增的代码落点

关键落点如下：

1. 诊断面板新增状态展示：
   - `src/components/SplineScene.vue`
2. orbit 开关与左键分流：
   - `setOrbitEnabled`
   - `syncOrbitAvailability`
3. 透明面命中检测：
   - `intersectSceneAtPointer`
   - `hitTestControlSurface`
4. 屏幕热点兜底：
   - `CONTROL_HOTSPOT_FALLBACKS`
   - `fallbackControlSurfaceAtPointer`
5. 悬停反馈：
   - `updateHoverState`
   - `updateCanvasCursor`
6. 调试桥接：
   - `debugRaycastAt`
   - `projectObjectToScreen`

---

## 8. 本轮没有彻底解决的根因

这轮虽然把交互恢复了，但有一个根因问题并没有真正修掉：

1. 当前 Spline runtime 下，这两个透明拖拽面和两个旋钮对象并不容易被稳定射线命中

这意味着目前的“可用”主要建立在：

1. 射线命中优先
2. 屏幕热点兜底兜住当前默认视角

所以这不是最终最优方案，只是当前最务实、最稳定的收口方案。

---

## 9. 当前剩余风险

后续最需要注意的风险点是：

1. 屏幕热点兜底依赖当前默认正视角
2. 如果用户按 `Alt` 把相机旋转太多，热点位置可能漂移
3. 如果后续改了：
   - 相机参数
   - 默认 zoom
   - 场景构图
   - 控制台模型位置
   则热点比例需要重新校准
4. `projectObjectToScreen()` 当前投影结果明显不能直接拿来做动态热点
5. 透明拖拽面为什么在 runtime 下几乎不可稳定命中，这个底层原因仍未彻底查清

---

## 10. 本轮结论

本轮已经把当前最影响可用性的三个问题先收住了：

1. 左键视角拖动与旋钮拖拽冲突：已分流
2. 旋钮拖动无反应：已恢复
3. 无法直观看到命中状态：已补悬停反馈和诊断

当前版本已经适合继续人工验收和继续迭代，但如果后续要把这套方案做到完全稳健，下一步就应该继续处理“透明面 / 旋钮射线命中不稳定”的底层问题，而不是无限依赖屏幕热点兜底。
