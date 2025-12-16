---
sidebar_position: 3
---

# Bianbu 2.2更新说明

基于 **Ubuntu 24.04.1** 源码构建。

Bianbu 2.2源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/v2.2 noble-security/snapshots/v2.2 noble-updates/snapshots/v2.2 noble-porting/snapshots/v2.2 noble-customization/snapshots/v2.2 bianbu-v2.2-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- 使用此源即可安装到后续的2.2.x（如2.2.1）发布的包，其存放在 `bianbu-v2.2-updates`。
- 如需下载源码，请将`Types: deb`改成`Types: deb deb-src`。

## v2.2.1

**发布日期：** 2025-8-16

对应的 **BSP** 版本：[v2.2.7](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### Bianbu OS基础组件主要更新

- 支持Lichee PI 3A开发板

## v2.2

**发布日期：** 2025-5-9

对应的 **BSP** 版本：[v2.2](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### LXQt桌面组件主要更新

**1. 新镜像 - bianbu lxqt桌面版本镜像**

- 基于 `lxqt 2.1.0`
- 默认使用 Wayland 协议
- **lxqt-wayland-session**：新增快捷键
  - ctrl + shift + 3 截取整个屏幕
  - ctrl + shift + 4 自定义区域截图
  - F11 快捷键切换应用全屏
  - Ctrl + Alt + ←/→ 切换左右工作区
  - logo + F1~F4 跳转到指定工作区

### GNOME桌面组件主要更新

该版本GNOME桌面组件暂无更新

### Bianbu OS基础组件主要更新

**1. 显示**

- **wlroots**：
  - 新增实时显示桌面帧率功能
  - 修复双屏同显时插拔后，HDMI 屏幕出现下半截红屏的问题

- **raindrop**：修复双屏拓展模式下，副屏桌面及图标概率性消失的问题

**2. 多媒体**

- **mpp**：修复 multiple encoding 时发生 segment fault 的问题
- **pipewire**：修复录屏时锁屏导致的报错退出问题
- **gst-plugins-bad1.0**：
  - 新增休眠时 `spacemitsrc` 插件主动退出功能
  - 新增 `spacemitsrc` 支持主动丢帧
  - 修复编码长时间无法退出的问题
- **mpv**：
  - 新增 `dmabuf-wayland` 显示后端
  - 修复播放120帧视频时出现撕裂现象的问题
  - 修复老化测试视频卡住的问题
- **k1x-cam**：
  - 新增 `max96716_max96717_imx556` SerDes 出图功能
  - 新增 `sc501ai` 出图功能并完成isp调优
  - 新增 ccic 双路 raw dump 功能
  - 修复 `ov5647` 曝光异常和高噪点问题
  - 修复上下电 `dvdd_powen_cnt` 计数不对应问题
- **ffmpeg**：
  - 新增输入一帧解码输出一帧的功能
  - 调整部分打印等级

**3. 开发工具**

- **gpiozero**: 支持 RV4B 40Pin 接口

**4. AI**

- **asr-llm**：支持将自动语音识别与大语言模型集成

**5. BSP**

- **bianbu-esos**:
  - 适配 `v2.2` 内核
  - 新增相关许可证信息
