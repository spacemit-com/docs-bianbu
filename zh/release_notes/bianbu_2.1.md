---
sidebar_position: 3
---

# Bianbu 2.1更新说明 [已停服]

Buildroot 2.1 于 2025年7月31日 停止维护，推荐使用 2.2 或后续版本，如有疑问，请联系我们。

基于 **Ubuntu 24.04.1** 源码构建。

Bianbu 2.1源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/v2.1 noble-security/snapshots/v2.1 noble-updates/snapshots/v2.1 noble-porting/snapshots/v2.1 noble-customization/snapshots/v2.1 bianbu-v2.1-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- 使用此源即可安装到后续的2.1.x（如2.1.1）发布的包，其存放在 `bianbu-v2.1-updates`。
- 如需下载源码，请将 `Types: deb` 改成 `Types: deb deb-src` 。

## v2.1.1

**发布日期：** 2025-3-13

对应的 **Buildroot** 版本: [v2.1](https://sdk.spacemit.com/release_notes/bl-v2.1.y#v21%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

**AI**

- **openwebui**：支持deepseek
- **spacemit-ollama-toolkit**：支持deepseek

## v2.1

**发布日期：** 2025-2-11

对应的 **Buildroot** 版本：[v2.1](https://sdk.spacemit.com/release_notes/bl-v2.1.y#v21%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

**AI**

- **ollama**：新增 `spacemit-ollama-toolkit`
- **open-webui**：支持 `openwebui`

**显示**

- **img-gpu-powervr**: 
  - 支持通过 `libOpenCL.so.1` 调用 PVR GPU 的 OpenCL 实现
  - 修复个别 PPT 幻灯片播放失败问题
- **mutter**：修复登录界面概率性黑屏问题

**多媒体**

- **GStreamer1.0**：新增 `SPACEMIT_GST_DEBUG_DUMP_DOT_DIR` 调试功能，支持生成 Pipeline Graphviz 图表
- **gst-plugins-base1.0**：优化 `glsinkbin` 送显帧率
- **gst-plugins-bad1.0**：
  - 新增 `spacemitsrc` 组件的主动丢帧功能
  - 修复 `spacemitsrc` 数据无法硬编码的问题
- **FFmpeg**：支持硬解码后叠加水印功能
- **mpv**：修复锁屏解锁后闪退问题
- **audacious**：修复任务栏显示两个点的问题

**应用**

- **chromium-browser-stable**：修复若干问题，优化启动速度
- **chromium**：升级至131，暂不支持视频硬件解码
- **LibreOffice**：调整编译优化等级为 `-O2`，提升交互流畅度
- **GNOME**：
  - 禁用缩略图
  - 优化过场动画
- **xfwm4**：修复软件渲染模式下的窗口管理器卡顿问题
- **zed**：新增 `spacemit-code-forge`
- **plymouth**：修复刷机后启动界面显示异常

**开发工具**

- **LLVM**：支持 LLVM 19
- **gcc-13**：支持 `zicbom_zicbop_zicboz_zicond_zfh_zfhmin` 扩展
- **sysprof**：修复调用栈无法显示的问题
