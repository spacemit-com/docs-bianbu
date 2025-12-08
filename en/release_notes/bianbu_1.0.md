---
sidebar_position: 1
---

# Bianbu 1.0 Release Notes [End of Life]

## Bianbu 1.0 End of Life Announcement

SpacemiT **Bianbu v1.0**, which was built on **Ubuntu 23.10**, is no longer supported or updated by the Ubuntu community. SpacemiT has now released **Bianbu v2.0** (currently at version v2.0.4), which is based on **Ubuntu 24.04** and includes deep optimizations for RISC-V. Please note that Bianbu V1.0 is no longer maintained and has reached its **End of Life (EOL)**.

**Bianbu 1.0 End of Life (EOL) Time: 2024/12/31**

Please refer to the document for the Bianbu v2.0 upgrade：
[user_guide-->upgrade](https://bianbu.spacemit.com/en/user_guide/upgrade)

## v1.0 release note

**Release Date:** 2024-5-30

The corresponding **Buildroot version:** [v1.0](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v10-release-note)

### Main components

#### Applications

- GNOME 45.0
- Chromium 122
- Nautilus
- Shotwell
- LibreOffice
- mpv
- Rhythmbox
- docker.io 24.0.5

#### Applications Framework

- Electron 29.3.1
- QT 5.15.10
- GTK 4.12.2

#### Multimedia Framework

- FFmpeg 6.0 (with Hardware Accelerated)
- GStreamer 1.22.5 (with Hardware Accelerated)
- PipeWire 0.3.79

#### Inference Framework

- onnxruntime 1.15.1 (with Hardware Accelerated)

#### Runtime

- Python 3.11.6
- OpenJDK 17 (default)
- Node.js

#### Libraries

- OpenCV 4.6.0
- OpenSSL 3.0.10
- MPP, SpacemiT multimedia processing platform, provides C API and sample
- Mesa 3D 22.3.5

### Know issue

- Unable to wake up after suspend for a long time

## v1.0.3

**Release Date:** 2024-6-19

The corresponding **Buildroot version:** [v1.0.3](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v103-release-note)

### Major Updates

- Fixed the issue of unable to wake up after a long time of suspend
- Fixed the lag issue in XFCE desktop
- Fixed the crash of the properties dialog box in the file browser
- Fixed the issue of repeated Chinese character input in Chromium

## v1.0.5

**Release Date:** 2024-6-26

The corresponding **Buildroot version:** [v1.0.5](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v105-release-note)

### Major Updates

- Fixed the issue of LibreOffice crashing with a low probability when opening Word or Excel files or saving files
- Fixed the issue of cheese becoming unresponsive after the system wakes up from suspend

## v1.0.7

**Release Date:** 2024-7-11

The corresponding **Buildroot version:** [v1.0.7](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v107-release-note)

## v1.0.8

**Release Date:** 2024-7-16

The corresponding **Buildroot version:** [v1.0.8](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v108-release-note)

## v1.0.9

**Release Date:** 2024-7-20

The corresponding **Buildroot version:** [v1.0.9](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v109-release-note)

## v1.0.11

**Release Date:** 2024-8-1

The corresponding **Buildroot version:** [v1.0.11](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v1011-release-note)

## v1.0.12

**Release Date:** 2024-8-2

The corresponding **Buildroot version:** [v1.0.12](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v1012-release-note)

### Major Updates

- Fixed object detection and pose tracking startup failed issue

## v1.0.13

**Release Date:** 2024-8-16

The corresponding **Buildroot version:** [v1.0.13](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v1013-release-note)

### Major Updates

- Added support for boot and shutdown animations
- Added support for installation of the `llm-client` large language model client
- Added support for installation of the `box64`
- Added the universe component
- Fixed issues with incorrect version numbers and repeated online upgrades

## v1.0.14

**Release Date:** 2024-8-31

The corresponding **Buildroot version:** [v1.0.14](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v1014-release-note)

### Major Updates

- Fixed login issue over serial console when HDMI (configured as primary display) is disconnected
- Fix ssh unsupport compress issue

## v1.0.15

**Release Date:** 2024-9-7

The corresponding **Buildroot version:** [v1.0.15](https://sdk.spacemit.com/en/release_notes/bl-v1.0.y#v1015-release-note)

### Major Updates

- Resolved low version issue in `spacemit-uart-bt` package
- Added upgrade support to the in-development **Bianbu 2.0** release
