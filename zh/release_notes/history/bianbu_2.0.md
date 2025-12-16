---
sidebar_position: 2
---

# Bianbu 2.0更新说明 [已停服]

Bianbu 2.0 于 2025年7月31日 停止维护，推荐使用 2.2 或后续版本，如有疑问，请联系我们。

基于 **Ubuntu 24.04** 源码构建。

Bianbu 2.0源：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/<version> noble-security/snapshots/<version> noble-porting/snapshots/<version> noble-customization/snapshots/<version>
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- `<version>`要替换成版本号，例如 `v2.0.4`。
- 如需下载源码，请将`Types: deb`改成`Types: deb deb-src`。

## v2.0.4

**发布日期：** 2024-12-11

对应的 **Buildroot** 版本：[v2.0.4](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v204%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复 chromium 在线视频长时间老化网页崩溃的问题
- 修复录屏颜色异常
- 修复 remmina 连接闪退的问题
- LLVM 升级至 `18.1.8-11`
- Mesa 升级至 `24.01`
- 优化启动时间
- 优化桌面动画和搜索服务
- 支持 vscodium (`codium`)
- 支持 zed (`spacemit-code-forge`)
- 支持 GRUB
- 支持 Fcitx5 （默认输入法）

## v2.0.2

**发布日期：** 2024-11-11

对应的 **Buildroot** 版本：[v2.0.2](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v202%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- `systemd` 支持 wakeup count
- mesa 支持 AMD R600 驱动
- 修复 GDB vector 调试相关问题
- 修复 mutter 内外置显卡兼容性问题

## v2.0.1

发布日期：2024-10-28

对应的 **Buildroot** 版本：[v2.0.1](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v201%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 更新 BSP 相关组件

## v2.0

**发布日期：** 2024-10-22

对应的 **Buildroot** 版本：[v2.0](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v20%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复首次启动提示 `Failed to start plymouth-quit-wait` 提示
- 修复亮色主题状态栏颜色问题
- LLVM 18 升级到 18.1.8
- GCC 14 升级到 14.2
- 更新 `img-gpu-powervr` ，修复 `gnome-shell` 编译报错的问题

## v2.0rc2

**发布日期：** 2024-9-30

对应的 **Buildroot** 版本：[v2.0rc7](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v20rc7%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 修复 Chromium 播放视频崩溃的问题
- 修复 Chromium Cookie 失效的问题
- 更新 box64，支持 RVV，支持运行 WPS Office

## v2.0rc1

**发布日期：** 2024-9-7

对应的 **Buildroot** 版本：[v2.0rc6](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v20rc6%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E)

### 主要更新

- 支持 python3-gpiozerov
- 支持 code-server
- 支持 apport
- gnome-initial-setup支持设置hostname

## v2.0beta2

**发布日期：** 2024-9-2

对应的 **Buildroot** 版本：[v2.0rc5](https://sdk.spacemit.com/release_notes/bl-v2.0.y/#v20rc5%E6%9B%B4%E6%96%B0%E8%AF%B4%E6%98%8E%E5%BC%80%E5%8F%91%E4%B8%AD)

### 主要更新

- 修复 HDMI 作为主屏时，没接导致串口无法登录的问题
- 修复 SSH 不支持压缩的问题
