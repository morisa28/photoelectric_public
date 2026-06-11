# 光电效应实验模拟仿真平台说明文档

## 1. 项目说明

本次整合后的项目目录为：

- `/home/just_monika/win_share/new`

该目录是在原有两个项目基础上完成的整合版本：

- `photoelectric-sim`：原 2D 部分
- `photoelectric-simulation`：原 3D 部分

整合目标是将 3D 部分接入到 2D 平台的“模拟试验台”中，作为一个按钮入口。用户点击按钮后，可跳转到 3D 模拟试验台页面。

本说明文档放置在 `new` 目录外部，路径为：

- `/home/just_monika/win_share/new-项目说明文档.md`

## 2. 整合结果

已完成以下内容：

1. 基于 `photoelectric-sim` 复制并构建了新的整合工程 `new`
2. 将 `photoelectric-simulation` 的静态 3D 页面资源复制到 `new/public/3d`
3. 在 2D 虚拟实验台页面增加“进入3D模拟试验台”按钮
4. 将原 3D 占位页改造成真实的 3D 承载页面
5. 完成一次最终打包验证，并生成 `dist` 产物

## 3. 关键实现位置

### 3.1 2D 按钮入口

文件：

- `/home/just_monika/win_share/new/src/components/VirtualLab.vue`

实现内容：

- 在“操作按钮组”中新增“进入3D模拟试验台”按钮
- 新增 `goToThreeD()` 方法
- 点击后通过路由跳转到 `/3d-view`

### 3.2 3D 页面承载

文件：

- `/home/just_monika/win_share/new/src/views/PlaceholderView.vue`

实现内容：

- 将原“3D视图开发中”的占位页面替换为正式页面
- 使用 `iframe` 加载 `public/3d/index.html`
- 提供“返回2D实验台”按钮

### 3.3 3D 静态资源位置

目录：

- `/home/just_monika/win_share/new/public/3d`

其中包含：

- `index.html`
- `css/style.css`
- `js/simulation.js`
- `js/three-scene.js`
- `js/charts.js`
- `js/controls.js`
- 两张图片资源

## 4. 项目目录说明

### 4.1 源码目录

- `/home/just_monika/win_share/new/src`

### 4.2 3D 静态资源目录

- `/home/just_monika/win_share/new/public/3d`

### 4.3 打包产物目录

- `/home/just_monika/win_share/new/dist`

## 5. 运行方式

建议使用以下 Node 版本：

- `Node 20.19+`
- 或 `Node 22.12+`

项目 `package.json` 中声明的版本要求为：

```json
"engines": {
  "node": "^20.19.0 || >=22.12.0"
}
```

在 `new` 目录下执行：

```bash
npm install
npm run dev
```

开发模式启动后：

1. 打开 2D 主页面
2. 进入“虚拟实验台”
3. 点击“进入3D模拟试验台”按钮
4. 跳转到 3D 页面进行操作

## 6. 打包方式

在 `new` 目录下执行：

```bash
npm install
npm run build
```

打包完成后产物位于：

- `/home/just_monika/win_share/new/dist`

## 7. 本次打包验证结果

本次已经完成实际构建验证，构建成功。

构建输出结果如下：

```text
✓ 39 modules transformed.
dist/index.html                   0.43 kB
dist/assets/index-ZVqJjSQi.css   22.42 kB
dist/assets/index-DZ5z44R5.js   132.27 kB
✓ built in 1.43s
```

已确认以下产物存在：

- `/home/just_monika/win_share/new/dist/index.html`
- `/home/just_monika/win_share/new/dist/3d/index.html`
- `/home/just_monika/win_share/new/dist/assets/index-DZ5z44R5.js`
- `/home/just_monika/win_share/new/dist/assets/index-ZVqJjSQi.css`

## 8. 注意事项

1. `new` 位于共享目录中，某些环境下 `node_modules` 的安装可能不稳定
2. 如果本地直接安装依赖异常，建议换到普通本地目录执行安装和打包
3. 3D 页面当前以静态子页面方式接入 2D 主系统，优点是改动小、兼容性高、风险低
4. 如果后续需要更深度联动，可以继续把 3D 参数与 2D 实验状态做双向同步

## 9. 当前交付内容

本次交付包括：

- 整合后的项目目录：`/home/just_monika/win_share/new`
- 已完成的打包产物：`/home/just_monika/win_share/new/dist`
- 当前说明文档：`/home/just_monika/win_share/new-项目说明文档.md`

