---
sidebar_position: 4
---

# Bianbu V3.0 Release Notes

Built based on **Ubuntu 25.04** source code.

Bianbu V3.0 Repository:

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: plucky/snapshots/v3.0 plucky-security/snapshots/v3.0 plucky-updates/snapshots/v3.0 plucky-porting/snapshots/v3.0 plucky-customization/snapshots/v3.0 bianbu-v3.0-updates
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- Using this repository allows installation of subsequent 3.0.x releases (such as 3.0.1), which are stored in `bianbu-v3.0-updates`.
- To download source code, change `Types: deb` to `Types: deb deb-src`.

## V3.0.1

**Release Date:** August 16, 2025

Corresponding **BSP** version: [v2.2.7](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### Major Updates

- Added support for the Lichee Pi 3A board

## V3.0

**Release Date:** July 31, 2025

Corresponding **BSP** version: [v2.2.6](https://sdk.spacemit.com/release_notes/bl-v2.2.y)

### Key updates to Bianbu OS basic components

**1. Toolchain**

- **gcc-14**: Default enabled extension instruction sets "--with-arch=rv64gc_zba_zbb_zbc_zbs_zicsr_zifencei" and "--with-tune=spacemit-x60"

**2. System Restore Function**

- **read-only-rootfs-config**: Added support for read-only root filesystem [Usage Guide](../../development/system-restore.md)

**3. Display**

- **wlroots**: Fixed Vulkan rendering failure when using Drm render node
- **raindrop**: Fixed probabilistic disappearance of secondary screen desktop and icons in dual-screen extended mode
- **img-gpu-powervr**: Added OpenGL to Vulkan API conversion support via Zink; Fixed Godot Vulkan backend initialization failure
- **xwayland, xserver-xorg-core**: Added OpenGL->Vulkan API conversion support in XWayland/Xorg (requires configuration /etc/environment: XWAYLAND_NO_GLAMOR=0)

**4. Multimedia**

- **freerdp3**: Added rvv_YUV420ToRGB_8u_P3AC4R

- **mpp**:
  - Fixed bytesperline parameter exception when decoding output yuv420p
  - Fixed mpp function pointer variables having the same name as ffmpeg functions
- **mpv**:
  - Specified to use opengl rendering
  - Fixed video display issue in lxqt desktop version
- **ffmpeg**:
  - Added decoding support for DRM_FORMAT_YUV420 output
  - Fixed avcodec_send_packet failure and abnormal decoding exit

**5. Libraries**

- **zlib**
  - Fixed sigsegv
  - Optimized chunkcopy_rvv

**6. BSP**

- **bianbu-esos**: Support for low memory solutions
