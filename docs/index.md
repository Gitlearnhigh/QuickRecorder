# QuickRecorder 源码模块索引

这里将仓库里的源码按更利于“长期可维护/可拓展”的视角重新分组，并复制到 `docs/extracted/quickrecorder/` 下，便于直接挑选拷贝到你的项目中复用（文件内容与原始源码保持一致）。

注意：复用时请遵守仓库根目录的 `LICENSE`。

## 入口与概览

- `docs/extracted/quickrecorder/README.md`（提取说明与复用建议）

## App（应用入口/状态）

- `docs/extracted/quickrecorder/App/README.md`
- `docs/extracted/quickrecorder/App/QuickRecorderApp.swift`（来源：`QuickRecorder/QuickRecorderApp.swift`）

## RecordingCore（录制核心）

- `docs/extracted/quickrecorder/RecordingCore/README.md`
- `docs/extracted/quickrecorder/RecordingCore/SCContext.swift`（来源：`QuickRecorder/SCContext.swift`）
- `docs/extracted/quickrecorder/RecordingCore/AVContext.swift`（来源：`QuickRecorder/AVContext.swift`）
- `docs/extracted/quickrecorder/RecordingCore/RecordEngine.swift`（来源：`QuickRecorder/RecordEngine.swift`）

## UI（SwiftUI 视图/交互）

- `docs/extracted/quickrecorder/UI/README.md`
- `docs/extracted/quickrecorder/UI/ContentView.swift`（来源：`QuickRecorder/ViewModel/ContentView.swift`）
- `docs/extracted/quickrecorder/UI/ContentViewNew.swift`（来源：`QuickRecorder/ViewModel/ContentViewNew.swift`）
- `docs/extracted/quickrecorder/UI/SettingsView.swift`（来源：`QuickRecorder/ViewModel/SettingsView.swift`）
- `docs/extracted/quickrecorder/UI/StatusBar.swift`（来源：`QuickRecorder/ViewModel/StatusBar.swift`）
- `docs/extracted/quickrecorder/UI/PreviewView.swift`（来源：`QuickRecorder/ViewModel/PreviewView.swift`）
- `docs/extracted/quickrecorder/UI/AreaSelector.swift`（来源：`QuickRecorder/ViewModel/AreaSelector.swift`）
- `docs/extracted/quickrecorder/UI/ScreenSelector.swift`（来源：`QuickRecorder/ViewModel/ScreenSelector.swift`）
- `docs/extracted/quickrecorder/UI/WinSelector.swift`（来源：`QuickRecorder/ViewModel/WinSelector.swift`）
- `docs/extracted/quickrecorder/UI/AppSelector.swift`（来源：`QuickRecorder/ViewModel/AppSelector.swift`）
- `docs/extracted/quickrecorder/UI/AppBlockSelector.swift`（来源：`QuickRecorder/ViewModel/AppBlockSelector.swift`）
- `docs/extracted/quickrecorder/UI/CameraOverlayer.swift`（来源：`QuickRecorder/ViewModel/CameraOverlayer.swift`）
- `docs/extracted/quickrecorder/UI/MousePointer.swift`（来源：`QuickRecorder/ViewModel/MousePointer.swift`）
- `docs/extracted/quickrecorder/UI/ScreenMagnifier.swift`（来源：`QuickRecorder/ViewModel/ScreenMagnifier.swift`）
- `docs/extracted/quickrecorder/UI/VideoEditor.swift`（来源：`QuickRecorder/ViewModel/VideoEditor.swift`）
- `docs/extracted/quickrecorder/UI/QmaPlayer.swift`（来源：`QuickRecorder/ViewModel/QmaPlayer.swift`）
- `docs/extracted/quickrecorder/UI/SurpriseView.swift`（来源：`QuickRecorder/ViewModel/SurpriseView.swift`）
- `docs/extracted/quickrecorder/UI/iDeviceSelector.swift`（来源：`QuickRecorder/ViewModel/iDeviceSelector.swift`）

## Supports

- `docs/extracted/quickrecorder/Supports/README.md`
- `docs/extracted/quickrecorder/Supports/AppleScript.swift`（来源：`QuickRecorder/Supports/AppleScript.swift`）
- `docs/extracted/quickrecorder/Supports/GroupForm.swift`（来源：`QuickRecorder/Supports/GroupForm.swift`）
- `docs/extracted/quickrecorder/Supports/SleepPreventer.swift`（来源：`QuickRecorder/Supports/SleepPreventer.swift`）
- `docs/extracted/quickrecorder/Supports/Sparkle.swift`（来源：`QuickRecorder/Supports/Sparkle.swift`）
- `docs/extracted/quickrecorder/Supports/WindowAccessor.swift`（来源：`QuickRecorder/Supports/WindowAccessor.swift`）
- `docs/extracted/quickrecorder/Supports/WindowHighlighter.swift`（来源：`QuickRecorder/Supports/WindowHighlighter.swift`）

## AppConfig

- `docs/extracted/quickrecorder/AppConfig/README.md`
- `docs/extracted/quickrecorder/AppConfig/Info.plist`（来源：`QuickRecorder/Info.plist`）
- `docs/extracted/quickrecorder/AppConfig/QuickRecorder.entitlements`（来源：`QuickRecorder/QuickRecorder.entitlements`）

## 你的新项目（建议架构/数据格式/同步方案）

- `docs/new-project-blueprint.md`
