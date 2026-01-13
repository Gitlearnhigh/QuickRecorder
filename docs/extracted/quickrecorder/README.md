# QuickRecorder 源码提取（供复用）

该目录是从仓库 `QuickRecorder/` 复制出来的源码快照，并按更偏“架构模块”的视角重新归类，方便你在新项目中按需拷贝复用。

## 目录结构

- `App/`：应用入口、全局状态、AppDelegate（包含录制入口与大量 UI 侧 glue code）。
- `RecordingCore/`：基于 `ScreenCaptureKit` 的采集与基于 `AVAssetWriter` 的写文件/封装，以及部分音频与后处理逻辑。
- `UI/`：SwiftUI 视图与交互（如果你要重做 UI，通常整块可忽略）。
- `Supports/`：更新、窗口辅助、阻止睡眠等工具类。
- `AppConfig/`：`Info.plist` 与 entitlements（新项目可按需参考权限项）。

## 推荐复用方式（面向“长期可维护/可拓展”）

QuickRecorder 当前工程里，录制状态机与 UI/全局单例耦合较重；更适合“抽思路与关键实现”，不建议原封不动搬过去。

建议你在新项目里拆成独立模块（例如 `Capture` / `Encode` / `InputEvents` / `ProjectFormat` / `Editor`），然后只从 `RecordingCore/` 里挑选并改造成“无 UI、无全局状态”的纯服务层。

