---
sidebar_position: 3
---

# Bianbu V4.0 更新说明

基于Ubuntu 26.04源码构建

Bianbu 4.0源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu4/
Suites: resolute resolute-security resolute-updates resolute-backports resolute-porting resolute-customization
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- 使用此源即可安装到后续的 V4.0.x（如 V4.0.1）发布的包。
- 如需下载源码，请将`Types: deb`改成`Types: deb deb-src`。

## V4.0.6 更新说明

发布日期：2026-8-26

对应的**BSP**版本：[V1.0.7](https://spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt桌面主要更新

- 修复snapshot快速点击快门导致窗口最大化的问题
- 修复亮度控制不可用时，背光插件卡住问题
- 修复lxqt-seeseion重置失败、删除按钮不合理以及滑动行为异常的问题
- 修复网络applet移除标题栏后窗口无法拖动
- 更新以太网和Wi-Fi设置的中文翻译

### 基础组件主要更新

- 修复wlroot休眠后黑屏问题
- 修复了以太网连接列表中偶尔会异常多出多余有线网络连接的问题

## V4.0.4 更新说明

发布日期：2026-7-23

对应的**BSP**版本：[V1.0.5](https://spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt桌面主要更新

- 新增会话设置
- 修复多屏场景下状态栏弹窗显示位置错乱的问题

### 基础组件主要更新

- 修复系统唤醒后mpv播放视频卡住问题
- 修复部分模块休眠唤醒老化问题
- 修复lxqt休眠唤醒老化后黑屏问题
- 修复系统监视器选择CPU颜色后意外退出问题
- 修复Type-c兼容性导致K3 Pico板无法启动问题
- 修复K3 Pico OTA过程中黑屏问题

## V4.0.1 更新说明

发布日期：2026-5-29

对应的**BSP**版本：[V1.0.2](https://spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt桌面主要更新

- 新增外观设置
- 新增桌面工作空间切换
- 引导页面支持兼容小分辨率屏幕

### 基础组件主要更新

**应用**

- 软件源升级适配为Ubuntu 26.04正式源

**Boot**

- 修复reboot fastboot无法烧录的问题

**显示**

- wlroot修复休眠唤醒后桌面背景概率性丢失问题

## V4.0.0 更新说明

发布日期：2026-4-30

**注意：Bianbu V4.0版本镜像仅支持K3。**

对应的**BSP**版本：[V1.0.0](https://spacemit.com/community/document/info?lang=zh&nodepath=software/SDK/buildroot/k3_buildroot/release_notes/bl-v1.0.y.md)

### LXQt桌面

- 自研应用 bianbu-control-center（配置中心）：新增语言和区域、桌面设置、通知设置、软件更新和关于模块。
- 状态栏通知默认为勿扰模式。
- 默认安装 snapshot（相机）、VLC media player（VLC 媒体播放器）、Zed 和 gnome system monitor（系统监视器），不再默认安装 cheese（茄子）。

### Bianbu v4.0基础组件与应用

**应用**

- Chromium 143
- Libreoffice
- VSCodium
- mpv
- fcitx5
- snapshot（相机）
- VLC media player（VLC 媒体播放器）
- Zed
- gnome system monitor（系统监视器）

**应用框架**

- QT 5.15.8
- QT 6.10.2
- GTK 3.24.51
- GTK 4.21.6

**多媒体框架**

- FFmpeg 8.0 (with Hardware Accelerated)
- GStreamer 1.28.0 (with Hardware Accelerated)
- PipeWire 1.6.0

**AI推理框架**

- spacemit-onnxruntime
- llama.cpp-tools-spacemit

**运行时**

- Python 3.14.3
- OpenJDK
- Node.js

**库**

- OpenCV 4.14.0
- OpenSSL 3.5.3
- MPP，进迭时空多媒体处理平台，提供 C API 和 sample
- Mesa 3D 24.01

**工具链**

- gcc15