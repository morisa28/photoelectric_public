# new（3）本地 3D 模型接入说明

## 1. 目标

本说明面向继续迭代 `new（3）` 的开发者，用于说明本地 3D 模型接入层已经落地到哪些目录、页面如何识别模型、后续如何替换成真实资源。

## 2. 已建立的接入层

当前项目已新增以下结构：

- `/home/just_monika/win_share/new（3）/public/3d/models/manifest.json`
- `/home/just_monika/win_share/new（3）/public/3d/models/README.md`
- `/home/just_monika/win_share/new（3）/public/3d/models/lamp/`
- `/home/just_monika/win_share/new（3）/public/3d/models/photocell/`
- `/home/just_monika/win_share/new（3）/public/3d/models/aperture/`
- `/home/just_monika/win_share/new（3）/public/3d/models/meter/`
- `/home/just_monika/win_share/new（3）/public/3d/models/cable/`
- `/home/just_monika/win_share/new（3）/public/3d/js/model-loader.js`

每个器材目录当前包含：

1. `placeholder.gltf`
2. `preview.svg`

这些文件是接入层占位资源，不代表最终真实模型。

## 3. 运行机制

3D 页面启动后会执行 `model-loader.js`：

1. 读取 `models/manifest.json`
2. 检查每条记录的本地模型文件和预览文件是否可访问
3. 在页面右侧“本地模型接入”面板展示状态
4. 将结果绑定到对应器材槽位，例如：
   - `lamp-group`
   - `photocell-group`
   - `aperture-group`
   - `meter-group`
   - `cable-hotspot`
5. 如果本地模型不可用，则自动回退到当前程序化 3D 器材

## 4. 替换真实模型的方法

建议按以下步骤替换：

1. 保持 `manifest.json` 中的 `id` 和 `slot` 不变
2. 将 `placeholder.gltf` 替换为真实 `glb` 或 `gltf`
3. 如果文件名变化，同步修改 `modelPath`
4. 如果有预览图，可替换 `preview.svg`
5. 刷新 3D 页面，确认“本地模型接入”面板显示为“本地模型已接入”

## 5. 当前边界

当前版本已经具备：

1. 本地文件目录规范
2. 模型清单加载
3. 本地文件可用性检测
4. 器材槽位绑定
5. 模型缺失时的程序化兜底

当前版本尚未具备：

1. 真正的 glTF / GLB 网页渲染
2. 材质、贴图、骨骼动画解析
3. 用真实模型替换当前 DOM 器材几何

## 6. 下一步建议

如果继续朝真实模型渲染推进，建议顺序为：

1. 保留当前 `manifest.json` 作为唯一模型配置源
2. 引入本地化 Three.js 或 Babylon.js
3. 先替换汞灯、光电管、试验仪三类关键器材
4. 再把当前程序化 3D 动画与真实模型节点绑定
