# Web（网页）壁纸开发参考

官方文档：https://docs.wallpaperengine.io/en/web/overview.html

Web 壁纸 = 一个普通网页项目。Wallpaper Engine 用内置 CEF（Chromium）浏览器渲染，每帧绘制到桌面。**这是唯一可以完全由 AI 编码创建的壁纸类型。**

## 项目结构

```
my-wallpaper/
├── project.json      # 必需：元数据 + 用户属性声明
├── index.html        # 必需：入口（project.json 的 "file" 指向它）
├── js/  css/  img/   # 可选：资源文件，相对路径引用
└── preview.jpg       # 可选：列表缩略图；发布时需正式预览图
```

## project.json 完整示例

```json
{
    "contentrating": "Everyone",
    "description": "一句话描述",
    "file": "index.html",
    "general": {
        "properties": {
            "schemecolor": {
                "order": 0,
                "text": "ui_browse_properties_scheme_color",
                "type": "color",
                "value": "0.815686 0.639216 0.905882"
            }
        }
    },
    "preview": "preview.jpg",
    "tags": ["Relaxing"],
    "title": "壁纸标题",
    "type": "web",
    "visibility": "private",
    "supporturl": "",
    "supportsaudioprocessing": false
}
```

| 字段 | 说明 |
|---|---|
| `type` | 必须为 `"web"` |
| `file` | 入口 HTML 文件名（不必叫 index.html） |
| `title` / `description` | 显示在壁纸列表与创意工坊 |
| `contentrating` | `Everyone` / `Questionable` / `Mature` |
| `visibility` | `private` / `unlisted` / `public`（仅发布时有意义） |
| `tags` | 标签数组，如 `["Game","Relaxing","Sci-Fi"]` |
| `general.properties` | 用户可调属性声明（见下） |
| `supportsaudioprocessing` | `true` 时启用音频监听（见下） |
| `preview` | 列表预览图（16:9） |

## 用户属性（User Properties）

在 `general.properties` 里声明，Wallpaper Engine 界面会出现对应控件。**每种类型的标准写法：**

```json
"amount":   { "order": 0, "type": "slider",   "text": "粒子数量", "value": 200, "min": 10,  "max": 1000 },
"toggle":   { "order": 1, "type": "checkbox", "text": "显示时钟", "value": true },
"mode":     { "order": 2, "type": "combo",    "text": "模式", "value": "auto",
              "options": [ { "label": "自动", "value": "auto" },
                           { "label": "手动", "value": "manual" } ] },
"schemecolor": { "order": 3, "type": "color", "text": "主色调", "value": "0.81 0.63 0.90" },
"username": { "order": 4, "type": "text",     "text": "签名文字", "value": "Hello" }
```

要点：
- `text` 是界面显示名（可中文）；`order` 控制排列顺序。
- **slider 的 value 是整数**；color 的 value 是 `"R G B"` 三个 0~1 浮点（空格分隔）；combo 的 value 必须等于某个 option 的 value。
- 键名（如 `amount`）即 JS 里收到的属性名。
- 显示条件：`"condition": "toggle.value == true"`——JS 布尔表达式，引用其他属性时带 `.value`，为 false 时该控件在 UI 中隐藏。

## wallpaperPropertyListener（接收属性值）

```javascript
window.wallpaperPropertyListener = {
    applyUserProperties: function (properties) {
        if (properties.amount) {
            var amount = properties.amount.value;   // 数值/布尔直接用
        }
        if (properties.schemecolor) {
            var rgb = properties.schemecolor.value.split(' ').map(parseFloat);
            // rgb = [0.81, 0.63, 0.90] → 转成 css 颜色自行处理
        }
    },
    applyGeneralProperties: function (properties) {
        if (properties.fps) { /* 引擎目标帧率，可据此降级特效 */ }
    }
};
```

**关键规则：**
- 回调在**页面加载完成后异步**触发，可能只触发一次或多次；不要在加载时同步读取属性。
- 每次用户改动设置都会**重新回调对应属性**，代码必须支持运行时热更新（如重设颜色、重建粒子数量）。
- 读取任何属性前先判空（`if (properties.xxx)`）。
- 保留 `schemecolor` 属性可让壁纸出现在"按颜色浏览"分类。

## 音频可视化

1. `project.json` 加 `"supportsaudioprocessing": true`
2. 代码：

```javascript
window.wallpaperRegisterAudioListener(function (audioArray) {
    // 128 个浮点数：索引 0-63 = 左声道，64-127 = 右声道
    // 典型范围 0~1（响度大时可 >1，用前 clamp）
    // 每帧调用一次；完全无声时不回调 —— 自己做衰减动画兜底
});
```

## 调试

1. **浏览器直接调试**：用 Chrome 打开 `index.html`，F12 看控制台。引擎专属 API 不存在会报错——用 mock：
   ```javascript
   if (!window.wallpaperRegisterAudioListener) {
       // 浏览器里假装有音频数据，方便开发
       window.wallpaperRegisterAudioListener = function (cb) {
           setInterval(function () {
               var fake = new Array(128).fill(0).map(function () { return Math.random(); });
               cb(fake);
           }, 50);
       };
   }
   ```
2. **编辑器预览**：编辑器打开项目后点「预览」即应用为壁纸；点画布/屏幕任意处可退出。
3. **性能检查**：Chrome 任务管理器（Shift+Esc）看该进程 CPU/GPU/内存；编辑器内也有性能显示。

## 性能与兼容注意

- 壁纸常驻后台：**严禁内存泄漏**（事件监听记得清理）、避免每帧创建大量对象（GC 卡顿）、避免过大的 DOM/CSS 动画。
- 动画统一走 `requestAnimationFrame`；尊重 `applyGeneralProperties` 的 fps 限制。
- 用户切换到全屏应用时引擎暂停渲染，但**页面不会卸载**——不要依赖"退出时清理"。
- CEF 支持现代 ES6+/WebGL/WebAudio；不要用通知、定位、弹窗等浏览器/桌面专属 API。
- 鼠标事件：Web 壁纸默认**不接收鼠标**（引擎把点击留给桌面）；设计上不要依赖 hover/click 交互。
- RGB 灯效（冷门）：`window.wpPlugins.led.setAllDevicesByImageData(rgbString, w, h)` 可同步 RGB 硬件，需在设置中启用 LED 插件，详见官方 [RGB 文档](https://docs.wallpaperengine.io/en/web/api/rgb.html)。
