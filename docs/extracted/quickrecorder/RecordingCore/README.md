# RecordingCore

本目录是 QuickRecorder 的录制核心，包含：

- `ScreenCaptureKit` 采集（`SCStream`、`SCContentFilter`、`SCStreamConfiguration`）。
- `AVAssetWriter` 写入视频/音频轨（实时写文件）。
- 麦克风采集（`AVAudioEngine` / `AVCaptureSession`）与可选 AEC（`AECAudioStream`）。
- 录制暂停/继续的时间戳处理（`timeOffset` / `adjustTime`）。
- 录制结束后的音轨混合导出（`mixAudioTracks`，基于 `AVAssetExportSession`）。

关键入口（便于快速定位）：

- 录制准备/选择过滤目标：`prepRecord(...)`（在 `RecordEngine.swift` 的 `AppDelegate` 扩展里）。
- 创建 `SCStream` 并开始采集：`record(filter:fastStart:)`（`RecordEngine.swift`）。
- 初始化写入器与轨道：`initVideo(conf:)`（`RecordEngine.swift`）。
- 样本回调（写入视频/系统音频）：`stream(_:didOutputSampleBuffer:of:)`（`RecordEngine.swift`）。
- 停止录制与收尾：`SCContext.stopRecording()`（`SCContext.swift`）。

注意点（如果你要做“可编辑工程 + 自动缩放”会更关键）：

- 当前暂停/继续主要在视频侧做了 `timeOffset` 调整，音频与输入事件若不共享同一时间基，容易在多次暂停后出现偏移。
- 麦克风与系统音频来自不同采集链路（`AVAudioEngine/AVCaptureSession` vs `ScreenCaptureKit`），建议在新项目中统一时间基与起始对齐策略。

