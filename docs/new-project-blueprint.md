# 录屏软件（长期可维护/可拓展）技术蓝图

目标：录制屏幕（视频 + 可选系统音频/麦克风）并同步记录鼠标行为数据；进入编辑器后根据鼠标行为自动生成“缩放/平移”效果，再导出成新视频。

下面内容以 QuickRecorder 的录制思路为参考，并对齐你给的 `.focusee` 工程格式样例（目录结构与 JSON 字段）。

## 建议技术栈（Mac 原生）

- 语言/UI：Swift；UI 你可选 SwiftUI 或 AppKit（编辑器复杂交互可以混用）。
- 采集：
  - 视频/系统音：`ScreenCaptureKit`（`SCStream`）。
  - 麦克风：优先 `AVAudioEngine`（低延迟、易处理），或 `AVCaptureSession`（多设备/更传统）。
  - 输入事件：建议用 Quartz Event Services 的 event tap（`CGEventTap`）捕获全局鼠标移动/点击；仅用 `NSEvent.addGlobalMonitorForEvents` 往往不够“可回放与可编辑”（且对权限/焦点更敏感）。
- 编码/导出：
  - 录制期写盘：`AVAssetWriter`（视频 + 1~2 条音轨）。
  - 编辑期渲染导出：简单缩放可用 `AVVideoComposition`；复杂光标/特效建议 Metal 渲染（`MTLTexture` + `AVAssetWriterInputPixelBufferAdaptor`）。

## 模块划分（强建议从一开始就解耦）

- `CaptureKit`：只做 ScreenCaptureKit 采样、产出视频/系统音 sample buffer。
- `MicCaptureKit`：只做麦克风采样（PCM 或 CMSampleBuffer）。
- `InputEventsKit`：只做鼠标/键盘事件采集与落盘（不依赖 UI）。
- `TimelineKit`：统一时间基与对齐（把 “processTimeMs / PTS / 录制起点” 串起来）。
- `ProjectFormat`：工程包（目录、元数据、事件 JSON、媒体文件）的读写。
- `Editor`：基于事件数据生成缩放曲线/关键帧；可叠加手动编辑。
- `Exporter`：读取工程包 + 渲染导出最终视频。

## 录制：视频与音频源的获取

### 视频（ScreenCaptureKit）

- 用 `SCContentFilter` 选择 display/window/app/region。
- `SCStreamConfiguration` 里明确：
  - 分辨率（注意 Retina：point/pixel scale）。
  - `minimumFrameInterval`/帧率控制（如果你提供 30/60 可选）。
  - `queueDepth`（过小会抖，过大延迟上升；QuickRecorder HDR 走 8）。
  - 色彩空间与像素格式（SDR 多用 BGRA；HDR 走 BT.2100 PQ + 10bit YUV 更省带宽）。

### 系统音频（ScreenCaptureKit）

- macOS 13+：在 `SCStream` 增加 `.audio` 输出，并设置 `capturesAudio/sampleRate/channelCount`。
- 注意：系统音是随 ScreenCaptureKit 的时间线走的，通常更容易与视频对齐。

### “本地应用音频”（由 UI 传入）

如果你的产品需要从 UI 选择“只录某个应用的声音”（而不是全系统声音），建议在录制参数里显式建模：

- `AudioCaptureMode.system`：系统音频（跟随当前 `SCContentFilter` 覆盖的内容）。
- `AudioCaptureMode.app(bundleIDs: [String])`：本地应用音频（UI 传入 bundle id 列表）。

实现路径（建议从简到繁）：

- **优先方案：仍用 ScreenCaptureKit 的 `.audio` 输出 + 通过 `SCContentFilter` 做“排除/包含应用”**。经验上它会影响音频捕获范围（被排除的应用通常也不会出现在音频里）。要做到“只录某个应用”，可以构造 `excludingApplications` 为“除目标 app 之外的所有 running apps”，或者在“窗口录制”模式下只包含目标窗口/应用。
- **增强方案（未来可扩展）：基于 CoreAudio 的 per-process tap/Audio Capture API**（不同 macOS 版本能力差异较大，工程成本也更高）。如果你先做 MVP，建议把这一块留在 `InputEventsKit/ProjectFormat` 的抽象后面，未来替换实现而不动上层。

注意点：

- “只录某应用音频”在实际系统上可能受限制（应用走独占/外设/DRM 等），需要做好 UI 降级与提示。
- 无论是系统音还是应用音，都建议统一采样率（例如 48kHz）并走同一时间基对齐逻辑（见“音画同步”章节）。

### 麦克风（AVAudioEngine / AVCaptureSession）

- 采集时建议直接落到 `AVAssetWriterInput` 的音频轨，避免中间写临时文件再混合（减少漂移与二次转码）。

## 录制期音轨策略：始终两条音轨（强建议）

按你的需求，录制时就生成两条音轨并写入同一个容器（例如 `screen_record.mp4`）：

- Track A：系统音频 **或** 本地应用音频（由 UI 选择并传入）。
- Track B：麦克风音频。

推荐做法：

- `AVAssetWriter` 上创建 **两个** `AVAssetWriterInput(mediaType: .audio, ...)`，分别对应 Track A/Track B，并 `expectsMediaDataInRealTime = true`。
- ScreenCaptureKit 的 `.audio` sample buffer 只喂给 Track A。
- 麦克风采集链路产出的 sample buffer 只喂给 Track B。
- 在 `metadata.json` 里记录音频模式与所选 app（如 `audioCaptureMode: "system"` 或 `audioCaptureMode: {"app":["com.xxx"]}`），确保编辑器可复现。

## 音画同步：统一时间基（关键）

你给的 `.focusee` 样例里，事件与媒体都使用 `processTimeMs`（看起来是单调时钟的毫秒值），并在 `metadata.json` 里记录每段 session 的 `processTimeStartMs/EndMs` 与 `durationMs`。

建议你也采用类似思路：

- 统一使用“单调时钟”作为对齐基准（例如 `CGEventTimestamp`/mach time 转毫秒），记为 `processTimeMs`。
- 在录制开始时记录一个锚点：
  - `t0_processMs`：开始采集那一刻的单调时间（毫秒）。
  - `t0_videoPTS`：写入器开始 session 的 PTS（通常取第一帧视频的 PTS）。
- 任意事件（鼠标/键盘）的 timelineTime：
  - `eventTimeMs = event.processTimeMs - t0_processMs`
- 任意帧/音频样本的 timelineTime：
  - `sampleTimeMs = (samplePTS - t0_videoPTS)` 转毫秒
- 这样编辑器只需要面对一个“从 0 开始的时间轴”，避免把 UI/系统时间混进来。

暂停/继续：

- 你需要在 `TimelineKit` 里维护“暂停区间”的累计偏移，并对视频/音频/事件都应用同一套 offset；否则很容易出现音画不同步或事件对不上画面。

## 鼠标事件捕获：建议落盘格式（参考 `.focusee`）

`.focusee/recording/` 里你给的文件结构是：

- `screen_record.mp4`
- `microphone_record.m4a`（可选）
- `metadata.json`（总索引）
- `mousemoves.json` / `mouseclicks.json` / `keystrokes.json`
- `cursors.json` + `cursors/*.png`

建议你沿用类似拆分，原因是：

- 事件文件可增量写入、可按需加载（编辑器只读必要部分）。
- 光标资源与事件解耦（减少重复存储）。

### `mousemoves.json`（示例字段）

每条记录建议包含（与样例一致）：

- `processTimeMs`：单调时间（毫秒）。
- `x/y`：屏幕坐标（建议用“points”，并在 `metadata.json` 里写清 `bounds` 与 `recordingScale`）。
- `type`：如 `mouseMoved` / `mouseDragged`（可扩展）。
- `cursorId`：当前光标 ID（对应 `cursors.json`）。

### `mouseclicks.json`

- `type`：`mouseDown` / `mouseUp`（你也可以加 `click`、`doubleClick`、`rightClick`）。
- `button`：`left/right/other`
- 其余同上（`processTimeMs/x/y/cursorId`）。

### 坐标与缩放注意事项

- 录制视频通常是像素尺寸；鼠标事件通常更自然是 points（逻辑坐标）。
- 需要在 `metadata.json` 里记录：
  - `bounds`（points）
  - `recordingScale`（例如 Retina 为 2）
  - 以及最终视频像素宽高（可从视频轨读回写入元数据，或录制期就写入）
- 编辑器生成缩放效果时，需要把 points→像素→最终输出坐标做清晰、可测试的转换。

## 工程包格式（建议）

建议你直接把工程当作“目录包”（像 `.focusee` 一样）：

```
<name>.yourproj/
  configure.yourproj   # UI/特效配置（可选）
  recording/
    metadata.json
    screen_record.mp4
    microphone_record.m4a   # 可选
    mousemoves.json
    mouseclicks.json
    keystrokes.json
    cursors.json
    cursors/*.png
  resource/
    background.png      # 可选
  bundle/               # 预留（缓存/渲染产物/索引）
```

优点：便于增量写入、跨版本兼容、也便于你未来把 “编辑产物/预渲染 cache” 放进 `bundle/`。

## `metadata.json`（建议结构，参考 `.focusee`）

建议用一个顶层索引文件把“媒体文件 + 事件文件 + 时间锚点/分段”描述清楚，最小可行结构可以接近你给的样例：

```json
{
  "platform": "Mac",
  "uniqueID": "UUID",
  "version": "1.0",
  "sessions": [
    {
      "processTimeStartMs": 54876119.2657,
      "processTimeEndMs": 54893939.6073,
      "durationMs": 17820.3415
    }
  ],
  "recorders": [
    {
      "id": "display",
      "type": "display",
      "sessions": [
        {
          "bounds": {"x": 0, "y": 0, "width": 1440, "height": 900},
          "recordingScale": 2,
          "outputFilename": "screen_record.mp4",
          "processTimeStartMs": 54876119.2657,
          "processTimeEndMs": 54893939.6073,
          "durationMs": 17820.3415
        }
      ]
    },
    {
      "id": "microphone",
      "type": "microphone",
      "sessions": [
        {
          "outputFilename": "microphone_record.m4a",
          "processTimeStartMs": 54876129.9658,
          "processTimeEndMs": 54893939.3809,
          "durationMs": 17809.4150
        }
      ]
    },
    {
      "id": "input",
      "type": "input",
      "sessions": [
        {
          "mouseMovesFilename": "mousemoves.json",
          "mouseClicksFilename": "mouseclicks.json",
          "keyStrokesFilename": "keystrokes.json",
          "processTimeStartMs": 54875827.6196,
          "processTimeEndMs": 54893981.3223,
          "durationMs": 18153.7027
        }
      ]
    },
    {
      "id": "cursor",
      "type": "cursor",
      "cursorsInfoFile": "cursors.json",
      "cursorImagesFolder": "cursors"
    }
  ]
}
```

几点实现建议：

- `sessions` 支持多段：暂停/继续时可以切分为多个 session（每段都有自己的 start/end/duration），编辑器更容易处理时间跳变。
- `bounds/recordingScale` 明确约定坐标系：建议 `bounds` 用 points（逻辑坐标），并用 `recordingScale` 说明 points→pixels 的倍数。
- 大文件写入：`mousemoves.json` 如果按数组写会非常大且难以流式追加；工程上更建议改成 JSON Lines（每行一条记录）并在 `metadata.json` 里标注格式版本。若你必须兼容 `.focusee` 风格的数组文件，可以录制期先写 JSONL，结束时再一次性转成数组 JSON。

## 输入事件采集实现建议（Mouse/Keyboard）

为了可编辑与更高可靠性，建议用 event tap：

- `CGEventTapCreate` 捕获 `kCGEventMouseMoved`、`kCGEventLeftMouseDown/Up`、`kCGEventRightMouseDown/Up`、`kCGEventOtherMouseDown/Up`、以及拖拽事件。
- 时间戳：用 `CGEventGetTimestamp(event)`（纳秒，单调时间）转毫秒作为 `processTimeMs`，与 `.focusee` 的 `processTimeMs` 语义更一致。
- 坐标：用 `CGEventGetLocation(event)` 获取全局屏幕坐标（points）；跨多显示器要记录当前 display 的 `bounds` 与鼠标所在屏幕。
- 权限：event tap 往往需要辅助功能权限（Accessibility）；需要清晰的权限引导与失败降级策略（例如退回 `NSEvent` 全局 monitor，但要告知精度/可靠性下降）。
