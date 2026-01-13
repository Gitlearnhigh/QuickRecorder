# AppConfig

包含：

- `Info.plist`
- `QuickRecorder.entitlements`

新项目复用时通常只需要参考：

- Screen Recording 权限（ScreenCaptureKit）。
- Microphone/Camera（如需录音/摄像头叠加）。
- Accessibility（如需更稳定捕获输入事件，尤其是全局鼠标/键盘事件）。

