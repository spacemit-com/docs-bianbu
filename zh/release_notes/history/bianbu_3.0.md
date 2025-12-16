---
sidebar_position: 4
---

# Bianbu 3.0更新说明

基于 **Ubuntu 25.04** 源码构建。

Bianbu 3.0源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: plucky/snapshots/v3.0 plucky-security/snapshots/v3.0 plucky-updates/snapshots/v3.0 plucky-porting/snapshots/v3.0 plucky-customization/snapshots/v3.0 bianbu-v3.0-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- 使用此源即可安装到后续的3.0.x（如3.0.1）发布的包，其存放在 `bianbu-v3.0-updates`。
- 如需下载源码，请将`Types: deb`改成`Types: deb deb-src`。

## v3.0.1

**发布日期：** 2025-8-16

对应的 **BSP** 版本：[v2.2.7](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### 主要更新

- 支持Lichee PI 3A开发板

## v3.0

**发布日期：** 2025-7-31

对应的 **BSP** 版本：[v2.2.6](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### Bianbu OS基础组件主要更新

**1. 工具链**

- **gcc-14**: 默认开启扩展指令集"--with-arch=rv64gc_zba_zbb_zbc_zbs_zicsr_zifencei"和"--with-tune=spacemit-x60"

**2. 系统还原功能**

- **read-only-rootfs-config**：新增支持只读根文件系统 [使用指南](../../development/system-restore.md)

**3. 显示**

- **wlroots**：修复 Vulkan 使用 Drm render node 渲染失败
- **raindrop**：修复双屏拓展模式下，副屏桌面及图标概率性消失的问题
- **img-gpu-powervr**：新增 OpenGL 通过 Zink 到 Vulkan 的 API 转换支持; 修复 Godot Vulkan 后端初始化失败
- **xwayland, xserver-xorg-core**：新增 XWayland/Xorg 中 OpenGL->Vulkan 的 API 转换支持（需配置 /etc/environment: XWAYLAND_NO_GLAMOR=0）

**4. 多媒体**

- **freerdp3**：新增 rvv_YUV420ToRGB_8u_P3AC4R

- **mpp**：
  - 修复 解码输出yuv420p时bytesperline参数异常
  - 修复 mpp函数指针变量与ffmpeg函数同名
- **mpv**：
  - 指定使用opengl渲染
  - 修复LXQt桌面版本无法显示视频
- **ffmpeg**：
  - 新增解码支持输出DRM_FORMAT_YUV420
  - 修复avcodec_send_packet失败，解码异常退出

**5. 库**

- **zlib**
  - 修复 sigsegv
  - 优化 chunkcopy_rvv

**6. BSP**

- **bianbu-esos**: 支持小内存方案
