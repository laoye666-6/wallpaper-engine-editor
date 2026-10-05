# 视频（Video）壁纸参考

视频壁纸 = 把 mp4 设为动态壁纸。AI 能完成全部视频处理（ffmpeg）；导入只需用户一次点击。

## 导入方式（指导用户）

- 编辑器：「创建壁纸」→「视频壁纸」→ 选择 mp4；或
- 直接把 mp4 拖进 Wallpaper Engine 主界面。

导入后可设置：静音、播放速度、剪裁、循环、清晰度模式等。

## ffmpeg 预处理

视频壁纸是性能大户，先处理再导入。常用命令：

```bash
# 标准转换：H.264 + yuv420p（兼容性最好的像素格式）
ffmpeg -i input.mov -c:v libx264 -crf 20 -preset slow -pix_fmt yuv420p -an output.mp4

# 压缩体积（适合手机投放/低配机）
ffmpeg -i input.mp4 -c:v libx264 -crf 26 -preset medium -vf "scale=1920:-2" -an output.mp4

# 裁剪成屏幕比例（16:9 例子）
ffmpeg -i input.mp4 -vf "crop=ih*16/9:ih" -c:v libx264 -crf 20 -pix_fmt yuv420p -an output.mp4

# 去掉音轨（多数壁纸不需要声音，省资源）
# 上面命令里的 -an 即去掉音频；要保留配乐就去掉 -an
```

要点：
- `-pix_fmt yuv420p` 必加，否则部分机器黑屏。
- 分辨率对齐屏幕（1920x1080 / 2560x1440），帧率保持原样或 30/60fps。
- 码率用 `-crf` 控制：18~20 高质量、23~26 通用。

## 无缝循环

- **首尾帧相同**的素材（Loop 素材）直接用。
- 普通视频：截取中间一段规避首尾（`-ss` / `-t`），或做交叉淡化：

```bash
# 交叉淡化 1 秒的无缝循环（先取尾段，再拼头段，再 xfade）
ffmpeg -i input.mp4 -ss 0 -t 8 head.mp4 -ss 9 -t 1 tail.mp4
ffmpeg -i head.mp4 -i tail.mp4 -filter_complex "xfade=transition=fade:duration=1:offset=7" loop.mp4
```

- 效果不佳时建议改用 Web 壁纸（HTML5 video 由代码控制循环/过渡，且可以做"视频+上层动效"混合）。

## 何时选视频 vs Web

| | 视频壁纸 | Web 壁纸 |
|---|---|---|
| 素材 | 已有实拍/渲染视频 | 代码生成效果 |
| 性能 | 解码开销，高分辨率高帧率较耗资源 | 取决于实现，可控 |
| 交互/个性化 | 几乎没有 | 完全可编程（属性、时钟、音频响应） |
| AI 自动化程度 | 视频处理全自动 | 全自动 |
