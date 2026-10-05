# 场景（Scene）壁纸与编辑器 GUI 参考

官方文档：https://docs.wallpaperengine.io/en/scene/overview.html

场景壁纸是 Wallpaper Engine 原生格式（图片分层 + GPU 特效），**功能最强、性能最好**，但创建依赖编辑器 GUI——AI 的角色是**准备素材 + 给出逐步操作指导**。

## 编辑器通用操作

- 打开编辑器：主界面左上角「壁纸编辑器」；或托盘图标右键 → 「制作壁纸」。
- 新建：「创建壁纸」→ 选择类型（场景/视频/网页/应用程序）。
- 场景壁纸创建后：把图片文件**直接拖进画布**即生成一个图像图层。
- 变换工具快捷键：移动 M、旋转 R、缩放 S；图层在右侧「图层」面板管理（可拖拽排序、调不透明度、混合模式）。
- 给图层加特效：选中图层 → 「效果」按钮 → 从效果浏览器选择（可叠加多个）。
- 预览：编辑器「预览」按钮直接应用为桌面壁纸； Esc/点击退出。
- 项目保存在 `<安装目录>/projects/myprojects/<项目名>/`。

## 常用效果（Effects）

| 效果 | 用途 |
|---|---|
| Parallax（视差） | 鼠标移动产生层次位移，最常用的"动感"来源 |
| Blur / Brightness / Color Tinting | 基础调色 |
| Light Rays / Glow | 光效 |
| Puppet Warp | 网格绑定骨骼，让静图局部摆动（头发、云、火焰） |
| Audio Visualizer | 随音乐跳动的可视化（含 Album Cover 变体） |
| Rain / Snow / Fire / Bubbles 等粒子 | 内置粒子发射器 |
| Timeline / Animation | 关键帧动画系统（属性打关键帧、编辑曲线） |
| Water / Fog / Sway | 环境动态 |

## AI 能自动化的部分

1. **素材准备**：用 Python/PIL 或 ImageMagick 把单张图切分成多层 PNG（前景/中景/背景）供视差分层；生成粒子贴图、遮罩。
2. **项目骨架**：可以在 `myprojects` 下手写场景项目的 `project.json`（`"type": "scene"`，`"file": "scene.json"`），但 `scene.json` 是编辑器私有格式，**不要手写**——必须由编辑器生成。
3. **操作指导**：按用户目标输出分步 GUI 操作说明（参考上文「编辑器通用操作」）。

## 进阶能力（指导用户了解即可）

- **SceneScript**：编辑器内用 JavaScript 编写图层脚本（官方推荐的脚本层），可实现自定义动画逻辑，文档：https://docs.wallpaperengine.io/en/scene/scenescript/overview.html
- **自定义着色器**：GLSL shader 图片/粒子效果。
- **模型支持**：可导入 glTF 3D 模型作为图层。
- **场景用户属性**：与 Web 壁纸同类（slider/checkbox/combo/color/text），另有场景专属的 `texture`（贴图选择）和 `usershortcut`（用户自选图片/视频）。场景壁纸在编辑器 GUI 中通过「编辑 → 更改项目设置 → 用户属性」添加。

## 何时建议用户改用 Web 壁纸

如果效果能用 Canvas/WebGL 实现（粒子、时钟、图表、代码生成图案），优先做 Web 壁纸——AI 可全自动完成，用户零 GUI 操作。场景壁纸只在需要"真实图片分层+内置特效"时才值得走 GUI 流程。
