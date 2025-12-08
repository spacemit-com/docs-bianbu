---
sidebar_position: 2
---

# Bianbu 2.0 Release Notes [End of Life]

Bianbu 2.0 will reach end of maintenance on July 31, 2025. We recommend using version 2.2 or later. If you have any questions, please contact us.

Build from Ubuntu **24.04** sources.

Bianbu 2.0 source：

```
Types: deb
URIs: https://archive.spacemit.com/bianbu/
Suites: noble/snapshots/<version> noble-security/snapshots/<version> noble-porting/snapshots/<version> noble-customization/snapshots/<version>
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

- Replace `<version>` with the actual version number (e.g., `v2.0.4`).
- To download source packages, modify `Types: deb` to Types: `deb deb-src`.

## v2.0.4

**Release Date:** 2024-12-11

The corresponding **Buildroot version:** [v2.0.4](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v204-release-note)

### Major Updates

- Resolved Chromium crash issue after extended online video playback
- Fixed screen recording color issues
- Fixed Remmina connection crashes
- Upgraded LLVM to `18.1.8-11`
- Upgraded Mesa to `24.01`
- Optimized startup time
- Optimized desktop animations and search services
- Added support for VSCodium (`codium`)
- Added support for Zed (`spacemit-code-forge`)
- Added support for GRUB
- Added support for Fcitx5 (default input method)

## v2.0.2

**Release Date:** 2024-11-11

The corresponding **Buildroot version:**: [v2.0.2](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v202-release-note)

### Major Updates

- `systemd` supports wakeup count
- Added AMD R600 driver support to Mesa
- Fixed GDB vector debugging issues
- Fixed mutter compatibility issues with internal and external graphics cards

## v2.0.1

**Release Date:** 2024-10-28

The corresponding **Buildroot version:** [v2.0.1](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v201-release-note)

### Major Updates

- Update BSP related components

## v2.0

**Release Date:** 2024-10-22

The corresponding **Buildroot version:** [v2.0](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v20-release-note)

### Major Updates

- Fixed the issue of "Failed to start `plymouth-quit-wait`" prompt on first boot
- Fixed the status bar color issue in light theme
- Upgraded LLVM 18 to 18.1.8
- Upgraded GCC 14 to 14.2
- Updated `img-gpu-powervr` to fix the `gnome-shell` compilation error

## v2.0rc2

**Release Date:** 2024-9-30

The corresponding **Buildroot version:** [v2.0rc7](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v20rc7-release-note)

### Major Updates

- Resolved Chromium crash issue during video playback  
- Fixed the issue of Chromium cookies becoming invalid
- Updated Box64 to support RVV, and WPS Office is supported

## v2.0rc1

**Release Date:** 2024-9-7

The corresponding **Buildroot version:** [v2.0rc6](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v20rc6-release-note)

### Major Updates

- Added support for `python3-gpiozero`
- Added support for `code-server`
- Added support for `apport`
- `gnome-initial-setup` now supports hostname configuration

## v2.0beta2

**Release Date:** 2024-9-2

The corresponding **Buildroot version:** [v2.0rc5](https://sdk.spacemit.com/en/release_notes/bl-v2.0.y#v20rc5-release-note)

### Major Updates

- Fixed issue preventing serial login when HDMI (set as primary display) is disconnected
- Resolved SSH compression compatibility issue
