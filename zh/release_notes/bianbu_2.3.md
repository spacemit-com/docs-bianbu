---
sidebar_position: 3
---

# Bianbu 2.3更新说明

基于 **Ubuntu 24.04.1** 源码构建。

Bianbu 2.3源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/v2.3 noble-security/snapshots/v2.3 noble-updates/snapshots/v2.3 noble-porting/snapshots/v2.3 noble-customization/snapshots/v2.3 bianbu-v2.3-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- 使用此源即可安装到后续的2.3.x（如2.3.1）发布的包，其存放在 `bianbu-v2.3-updates`。
- 如需下载源码，请将`Types: deb`改成`Types: deb deb-src`。


## V2.3.0

**发布日期：** 2025-12-15

对应的 **BSP** 版本：[v2.2](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### LXQt桌面组件主要更新

**1. UI**
- 优化状态栏用户体验
- 定制 Calamares 安装程序界面
- 支持 SpacemiT SDDM 主题
- 简化 Panel 操作
- 简化文件浏览器操作
- 支持 Wayland 锁屏应用
- 支持 SpacemiT Qt系统主题

**2. 应用**
- 支持永中Office（可安装）

**3. 性能**
- 优化系统启动时间
- libreoffice使用Qt6 GPU加速
