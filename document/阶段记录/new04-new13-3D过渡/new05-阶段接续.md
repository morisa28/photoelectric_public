# new（5）阶段接续文档

## 1. 当前阶段结论

`new（5）` 当前已经从 `new（4）` 的 3D 页面继续迭代为一个更简洁的 3D 实验场景版本。

本阶段的重点不是继续增加说明面板，而是收敛界面，只保留：

1. 3D 场景本体
2. 顶部核心数据显示
3. 微观过程按钮与弹窗
4. 返回 2D 实验台按钮

## 2. 当前最重要的事实

接手时先记住下面几点：

1. 3D 页面入口仍是 `src/views/ThreeDView.vue`
2. 主场景承载组件是 `src/components/three-lab/PhotoelectricLabScene.vue`
3. Three 场景构建逻辑在 `src/three-lab/createPhotoelectricLabScene.js`
4. 相机与键鼠控制逻辑在 `src/three-lab/createOrbitRig.js`
5. 统一状态源仍然只有 `src/stores/experiment.js`
6. Vite 已配置为 `watch.usePolling = true`，这是为了解决共享目录下热更新不稳定的问题

## 3. 本阶段已完成内容

### 3.1 页面收敛

已经移除：

1. 左侧说明卡
2. 右侧流程/诊断卡
3. 底部控制与说明区
4. 大标题说明头部
5. 实验台上方英文标签 `PHOTOELECTRIC LAB`

当前保留：

1. 顶部状态条
2. 右上角返回 2D 实验台按钮
3. 右下角微观过程按钮
4. 微观过程弹窗

### 3.2 相机与交互

当前控制逻辑如下：

1. 鼠标拖动：围绕实验台中心旋转视角
2. 滚轮：缩放远近
3. `A / D`：左右平移视角位置
4. `W / S`：前后拉近/拉远
5. `↑ ↓ ← →`：与 `W / A / S / D` 采用相同的平滑位移语义
6. 小键盘 `2/4/6/8`：仍保留旋转辅助控制

注意：

1. 键盘移动已经做成平滑过渡，不是离散跳步
2. 鼠标旋转中心固定在实验台区域，不再把视线甩离实验台

### 3.3 初始视角

当前默认进入 3D 页面时，初始镜头已调整为更偏向实验台主体区域。

相关参数在：

1. `createPhotoelectricLabScene.js` 中创建 `orbitRig` 的默认参数
2. `focusPreset('overview')` 对应的预设参数

## 4. 本阶段改动过的关键文件

1. `src/views/ThreeDView.vue`
2. `src/components/three-lab/PhotoelectricLabScene.vue`
3. `src/three-lab/createPhotoelectricLabScene.js`
4. `src/three-lab/createOrbitRig.js`
5. `src/three-lab/config.js`
6. `vite.config.js`

## 5. 当前仍需注意的问题

### 5.1 热更新问题

共享目录下 Vite 有过“本地文件已修改，但浏览器仍返回旧模块内容”的情况。

本阶段已经通过下面方式缓解：

1. 在 `vite.config.js` 中开启 `usePolling`
2. 每次关键交互逻辑修改后，主动重启 `5175` 的开发服务器

后续如果再遇到“页面没变”，优先检查：

1. `5175` 是否还是旧进程
2. 浏览器访问的是否仍是 `http://127.0.0.1:5175/3d-view`

### 5.2 初始镜头仍可能需要微调

虽然已经多轮调整，但“实验台在首屏中的构图位置”仍可能需要继续按截图微调。

如果后续继续改，优先调整：

1. `target`
2. `radius`
3. `polar`
4. `azimuth`

位置都在 `createPhotoelectricLabScene.js`。

## 6. 当前验证情况

本阶段结束前已确认：

1. `npm run build` 成功
2. 删除的说明区和英文标签已从代码移除
3. 新的键盘平移/前后移动逻辑已写入并被开发服务返回
4. 开发服务器已重新运行在 `http://127.0.0.1:5175/`

## 7. 如果下一阶段继续做，最适合的方向

建议优先级如下：

1. 继续按截图微调初始构图
2. 优化实验台模型比例与摆位
3. 提升键盘与鼠标控制手感
4. 如有需要，再逐步接入更真实的正式模型

## 8. 一句话交接

`new（5）` 当前已经是“极简 3D 实验页 + 平滑键盘位移 + 固定实验台旋转中心”的版本，后续最值得继续投入的是首屏构图微调和器材空间布局细化。
