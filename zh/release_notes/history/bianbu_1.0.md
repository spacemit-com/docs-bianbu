---
sidebar_position: 1
---

# Bianbu V1.0 更新说明 [已停服]

## Bianbu V1.0 停服公告

Bianbu V1.0 基于 Ubuntu 23.10 构建，Ubuntu 社区已停止对 23.10 的维护与更新。进迭时空基于 Ubuntu 24.04，针对 RISC-V 深度优化发布 Bianbu V2.0，最新版本迭代至 V2.0.4，现已决定对 Bianbu V1.0 停止维护（End of Life）。

| 版本  | 停止支持日期 | 延长生命周期（Extended Life-cycle Support, ELS）停止日期|
| ------------ | -------- | -------- |
|    Bianbu 1.x.x    |   2024/12/31     | NO |

建议用户更新 Bianbu V2.x版本使用，更新方式可以参考：**用户指南-系统升级**。

## V1.0更新说明

发布日期：2024-5-30

Buildroot版本：[v1.0](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v10%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要组件

#### 应用

- GNOME 45.0
- Chromium 122
- Nautilus
- Shotwell
- LibreOffice
- mpv
- Rhythmbox
- docker.io 24.0.5

#### 应用框架

- Electron 29.3.1
- QT 5.15.10
- GTK 4.12.2

#### 多媒体框架

- FFmpeg 6.0 (with Hardware Accelerated)
- GStreamer 1.22.5 (with Hardware Accelerated)
- PipeWire 0.3.79

#### 推理框架

- onnxruntime 1.15.1 (with Hardware Accelerated)

#### 运行时

- Python 3.11.6
- OpenJDK 17 (default)
- Node.js

#### 库

- OpenCV 4.6.0
- OpenSSL 3.0.10
- MPP，进迭时空多媒体处理平台，提供 C API 和 sample
- Mesa 3D 22.3.5

### 已知问题

- 长时间挂起后无法唤醒

## V1.0.3

发布日期：2024-6-19

Buildroot版本：[v1.0.3](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v103%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复长时间挂起后无法唤醒
- 修复XFCE桌面卡顿的问题
- 修复文件浏览器属性对话框crash的问题
- 修复Chromium中文重复输入的问题

## V1.0.5

发布日期：2024-6-26

Buildroot版本：[v1.0.5](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v105%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复LibreOffice打开word或excel、保存文件低概率闪退的问题
- 修复茄子相机在系统休眠唤醒后无响应的问题

## V1.0.7

发布日期：2024-7-11

Buildroot版本：[v1.0.7](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v107%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

## V1.0.8

发布日期：2024-7-16

Buildroot版本：[v1.0.8](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v108%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

## V1.0.9

发布日期：2024-7-20

Buildroot版本：[v1.0.9](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v109%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

## V1.0.11

发布日期：2024-8-1

Buildroot版本：[v1.0.11](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v1011%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

## V1.0.12

发布日期：2024-8-2

Buildroot版本：[v1.0.12](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v1012%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复目标检测和姿态跟踪无法打开的问题

## V1.0.13

发布日期：2024-8-16

Buildroot版本：[v1.0.13](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v1013%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 支持开关机动画
- 支持安装`llm-client`大语言模型客户端
- 支持安装`box64`
- 完善universe组件
- 修复版本号错误，反复在线升级的问题

## V1.0.14

发布日期：2024-8-31

Buildroot版本：[v1.0.14](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v1014%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复HDMI作为主屏时，没接导致串口无法登录的问题
- 修复ssh不支持压缩的问题

## V1.0.15

发布日期：2024-9-7

Buildroot版本：[v1.0.15](https://sdk.spacemit.com/release_notes/bl-v1.0.y#v1015%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复spacemit-uart-bt软件包版本过低的问题
- 支持升级到Bianbu 2.0（开发中的版本）
