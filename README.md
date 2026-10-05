# wallpaper-engine-editor

[中文](#简介) | [English](#english)

## 简介

一个让 AI 编码助手（ZCode / Claude Code 等）直接帮用户制作 **Wallpaper Engine（壁纸引擎）动态壁纸**的技能（Skill）。安装后，助手会掌握壁纸编辑器的完整用法：四种壁纸类型怎么选、`project.json` 怎么写、用户属性和音频可视化 API 怎么接、视频怎么用 ffmpeg 预处理、以及怎么发布到 Steam 创意工坊。

**最大亮点**：Web（网页）壁纸可以完全由 AI 编码创建——复制内置模板 → AI 改写 `index.html` → 用户在编辑器里一键导入，全程几乎零 GUI 操作。

## 功能

- 🧭 **壁纸类型路由**：Web / 视频 / 场景 / 应用程序壁纸怎么选，各自能自动化到什么程度
- 🌐 **Web 壁纸全自动开发**：`project.json` 全字段、`wallpaperPropertyListener`、5 种用户属性（slider/checkbox/combo/color/text）、显示条件、音频可视化（128 通道频谱）
- 🎬 **视频壁纸**：ffmpeg 预处理（H.264 编码、裁剪、压缩、无缝循环）
- 🖼️ **场景壁纸指导**：图层/特效/Puppet Warp/粒子/SceneScript 的操作指导 + AI 可自动化的素材准备
- 📦 **创意工坊发布**：发布/更新流程、预览图规格、自查清单
- 🧪 **内置模板**：开箱即用的音频律动粒子壁纸（含属性接线和浏览器调试 mock）

## 安装

### 方式一：git clone（推荐，便于更新）

```bash
# Windows (Git Bash / PowerShell)
git clone https://github.com/laoye666-6/wallpaper-engine-editor.git "$HOME/.agents/skills/wallpaper-engine-editor"
```

### 方式二：手动复制

下载本仓库，把 `wallpaper-engine-editor` 文件夹放进技能目录之一：

- 用户级：`~/.agents/skills/wallpaper-engine-editor/`（推荐，全局可用）
- 项目级：`<项目>/.agents/skills/wallpaper-engine-editor/`

重启会话后，提到 "Wallpaper Engine"、"动态壁纸"、"把视频做成壁纸" 等即可自动触发；也可用 `/skill wallpaper-engine-editor` 强制加载。

## 使用示例

- 「帮我做一个星空粒子网页壁纸，粒子数量和颜色可以调」→ 复制模板 → 改代码 → 指导导入
- 「我有个 mp4 想做成无缝循环的壁纸」→ ffmpeg 预处理 → 指导导入
- 「做一张我的照片的视差壁纸」→ 切分图层 → 场景编辑器操作指导
- 「壁纸怎么传到创意工坊？」→ 发布流程 + 自查清单

## 目录结构

```
wallpaper-engine-editor/
├── SKILL.md                     # 主文档：路由表、环境探测、三条快速工作流
├── references/
│   ├── web-wallpaper.md         # Web 壁纸完整开发参考（API/属性/调试）
│   ├── scene-editor.md          # 场景编辑器 GUI 与效果参考
│   ├── video-wallpaper.md       # 视频壁纸与 ffmpeg 预处理
│   └── publishing.md            # 创意工坊发布流程
└── assets/web-wallpaper-template/  # 最小可运行 Web 壁纸模板
    ├── project.json
    └── index.html
```

## English

A [ZCode](https://zcode.ai) / Claude Code skill that teaches AI assistants how to create **Wallpaper Engine** live wallpapers: choosing between Web/Video/Scene wallpaper types, authoring `project.json` and `wallpaperPropertyListener` user properties, audio-visualizer APIs, ffmpeg preprocessing for video wallpapers, and publishing to the Steam Workshop. Web wallpapers can be created entirely by AI code — copy the bundled template, let the AI rewrite `index.html`, then import it in the editor.

Clone into `~/.agents/skills/wallpaper-engine-editor/` and you're set.

## 参考

- 官方设计文档：https://docs.wallpaperengine.io
- Wallpaper Engine（Steam）：https://store.steampowered.com/app/431960/

## License

[MIT](LICENSE)
