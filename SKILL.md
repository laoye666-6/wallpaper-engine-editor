---
name: wallpaper-engine-editor
description: Wallpaper Engine（壁纸引擎）壁纸编辑器使用指南：制作网页/Web 壁纸、视频壁纸、场景壁纸，编写 project.json 与 wallpaperPropertyListener 用户属性、音频可视化壁纸，并发布到 Steam 创意工坊。当用户提到 Wallpaper Engine、壁纸引擎、动态壁纸、live wallpaper、把 HTML/视频/图片做成桌面动态壁纸、上传创意工坊壁纸时使用——即使用户没有明确说"Wallpaper Engine"。
---

# Wallpaper Engine 壁纸编辑器 Skill

Wallpaper Engine 是 Steam 上的动态壁纸软件，内置壁纸编辑器。壁纸项目本质是一个文件夹（含 `project.json` + 素材），编辑器负责导入、预览和上传创意工坊。

**核心判断**：Web（网页）壁纸完全由 HTML/CSS/JS 组成，可以**全部用代码创建**；视频壁纸只需准备好 mp4；场景壁纸依赖编辑器 GUI 操作（AI 只能准备素材和给步骤指导）。

## 壁纸类型路由

| 用户需求 | 类型 | 自动化程度 | 详细文档 |
|---|---|---|---|
| 网页/HTML/Canvas/WebGL/粒子/交互效果/时钟/音频可视化 | **Web** | ⭐ 完全可代码化 | [references/web-wallpaper.md](references/web-wallpaper.md) |
| 把已有视频做成壁纸、视频循环、视频压缩/编码 | **Video** | 视频处理可自动化（ffmpeg） | [references/video-wallpaper.md](references/video-wallpaper.md) |
| 图片分层视差、3D 效果、光影特效、Puppet Warp、编辑器粒子 | **Scene** | GUI 操作，AI 给素材+步骤指导 | [references/scene-editor.md](references/scene-editor.md) |
| 发布/更新到创意工坊、预览图、标签 | 任意 | 指导用户操作 | [references/publishing.md](references/publishing.md) |

## 第一步：环境探测

先确认 Wallpaper Engine 是否安装、找到项目目录，再决定写入位置。按顺序执行：

```bash
# 1) 从注册表找 Steam 根目录（Git Bash 下 reg 可用）
reg query "HKCU\Software\Valve\Steam" /v SteamPath

# 2) 解析所有 Steam 库路径
grep -i '"path"' "<Steam根目录>/steamapps/libraryfolders.vdf"

# 3) 逐个库检查壁纸引擎及其项目目录
ls "<库路径>/steamapps/common/wallpaper_engine/projects/myprojects"

# 4) 顺便确认 Wallpaper Engine 是否正在运行
tasklist | grep -i wallpaper
```

- **已安装**：项目直接写入 `<安装目录>/projects/myprojects/<项目名>/`。若 Wallpaper Engine 正在运行，主界面会自动扫描出新项目（必要时重启应用或在「已安装」列表刷新）。
- **未安装**：在用户工作目录（如当前项目目录）创建完整项目文件夹，写好 `project.json` + 素材，然后指导用户：打开 Wallpaper Engine → 「壁纸编辑器」→ 「创建壁纸」→ 选择项目的入口文件（Web 壁纸选 index.html）。

**编辑器入口**（指导用户用）：主界面左上角「壁纸编辑器」按钮；或任务栏托盘图标右键 → 「制作壁纸」。

## 快速工作流 A：Web 壁纸（首选，AI 全自动）

1. 复制模板起步：`cp -r <SKILL_ROOT>/assets/web-wallpaper-template <项目目录>`，模板已含可运行的 `project.json` + `index.html`（含属性接线 + 音频可视化示例）。
2. 阅读 [references/web-wallpaper.md](references/web-wallpaper.md) 了解全部字段与 API，然后改写 `index.html` 实现效果。
3. 设计分辨率按 **1920x1080** 实现（body 充满视口即可，引擎自适应分辨率）。
4. 测试：直接用浏览器打开 `index.html` 验证（需 mock 引擎 API，见 reference 的「调试」一节），再让用户在编辑器/桌面预览。
5. 有用户个性化需求（颜色、开关、下拉选项等）→ 在 `project.json` 的 `general.properties` 里声明，代码里通过 `wallpaperPropertyListener` 接收，速查见下文。

## 快速工作流 B：视频壁纸

1. 阅读 [references/video-wallpaper.md](references/video-wallpaper.md)，用 ffmpeg 预处理：转 H.264 mp4（`-pix_fmt yuv420p`）、按需裁剪/压缩/做无缝循环。
2. 指导用户：编辑器 → 「创建壁纸」→ 「视频壁纸」→ 选择 mp4；或直接把 mp4 拖到主界面。
3. 导入后可设置静音、播放速度、循环等。

## 快速工作流 C：场景壁纸（指导式）

AI 无法直接操作编辑器 GUI。阅读 [references/scene-editor.md](references/scene-editor.md)，然后：
1. AI 负责准备素材（分层 PNG、粒子贴图、生成图片等）。
2. 给用户逐步操作指导：创建场景壁纸 → 拖入图层 → 加效果 → 配用户属性 → 预览。

## 用户属性速查（project.json → JS）

```json
"general": { "properties": {
    "schemecolor": { "order": 0, "type": "color", "text": "主色调", "value": "0.81 0.63 0.90" },
    "amount":      { "order": 1, "type": "slider", "text": "数量", "value": 200, "min": 10, "max": 1000 },
    "toggle":      { "order": 2, "type": "checkbox", "text": "显示时钟", "value": true },
    "mode":        { "order": 3, "type": "combo", "text": "模式", "value": "auto",
                     "options": [ { "label": "自动", "value": "auto" }, { "label": "手动", "value": "manual" } ] }
} }
```

```javascript
window.wallpaperPropertyListener = {
    applyUserProperties: function (p) {
        // 在页面加载完成后异步调用；p.amount.value、p.toggle.value（bool）、
        // p.schemecolor.value 为 "R G B" 0~1 字符串需自行 split
    },
    applyGeneralProperties: function (p) { /* p.fps 引擎目标帧率 */ }
};
```

- 属性值变化时回调会再次触发，代码必须支持动态更新。
- 显示条件：属性可加 `"condition": "toggle.value == true"`（JS 布尔表达式，引用其他属性时带 `.value`）。

## 音频可视化速查

`project.json` 加 `"supportsaudioprocessing": true`，JS 注册监听：

```javascript
window.wallpaperRegisterAudioListener(function (a) {
    // a = 128 个浮点数：0-63 左声道、64-127 右声道，典型值 0~1（可超 1，需 clamp）
    // 每帧调用一次；无声音时不触发——注意做衰减动画兜底
});
```

## 发布检查清单（→ [references/publishing.md](references/publishing.md)）

编辑器菜单「创意工坊」→「准备上传/Prepare for publishing」：标题、描述、预览图（16:9、≥1280x720，GIF ≤1MB）、内容分级（Everyone/Questionable/Mature）、至少一个标签、可见性（默认 private 建议先私有自查）。

## 开发原则

1. **项目位置**：业务壁纸项目放在用户工作目录（或已安装时的 `myprojects`），**禁止写进本 skill 目录**，保持 skill 纯净可升级。
2. **性能优先**：壁纸常驻后台，避免内存泄漏和过高的 CPU/GPU 占用；动画用 `requestAnimationFrame`；尊重 `applyGeneralProperties` 传来的 fps；不用壁纸时引擎会暂停渲染但页面不会卸载。
3. **兼容性**：引擎内置浏览器是 CEF（Chromium），现代 ES6+ 可用；但不要依赖浏览器专属 API（如通知、定位）。
4. **参考来源**：官方设计文档 https://docs.wallpaperengine.io （场景/网页两套指南）；拿不准的 API 细节以上游文档为准。
