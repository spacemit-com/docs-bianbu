---
sidebar_position: 3
---

# Applications and Software Management

This section introduces the commonly preinstalled applications in Bianbu LXQt and explains how to manage software using the APT package manager.

## Common Applications Overview

| Application | Package Name | Icon | Description |
|------------|--------------|------|-------------|
| Web Browser | chromium-browser-stable | ![chromium-browser](static/chromium-browser.svg) | Open-source web browser based on Chromium |
| TigerVNC Viewer | tigervnc-viewer | ![tigervnc](static/tigervnc.svg) | VNC remote desktop client |
| LibreOffice | libreoffice | ![LibreOffice](static/libreoffice.svg) | LibreOffice launcher (office suite entry point) |
| LibreOffice Calc | libreoffice-calc | ![libreoffice-calc](static/libreoffice-calc.svg) | Spreadsheet editor |
| LibreOffice Draw | libreoffice-draw | ![libreoffice-draw](static/libreoffice-draw.svg) | Vector drawing and flowchart tool |
| LibreOffice Impress | libreoffice-impress | ![libreoffice-impress](static/libreoffice-impress.svg) | Presentation editor |
| LibreOffice Math | libreoffice-math | ![libreoffice-math](static/libreoffice-math.svg) | Formula editor |
| LibreOffice Writer | libreoffice-writer | ![libreoffice-writer](static/libreoffice-writer.svg) | Document and word processor |
| Okular | okular | ![okular](static/okular.svg) | Multi-format document viewer (PDF, EPUB, DjVu, etc.) |
| Skanlite | skanlite | ![org.kde.skanlite](static/org.kde.skanlite.svg) | Lightweight image scanning utility |
| 2048-Qt | 2048-qt | ![2048-qt](static/2048-qt.svg) | Classic 2048 puzzle game (Qt version) |
| Audacious | audacious | ![audacious](static/audacious.svg) | Lightweight audio player with plugin support |
| Cheese | cheese | ![cheese](static/cheese.svg) | Webcam photo and video capture tool |
| FeatherPad | featherpad | ![featherpad](static/featherpad.svg) | Lightweight plain-text editor with tabs |
| LXImage-Qt | lximage-qt | ![lximage-qt](static/lximage-qt.svg) | Default LXQt image viewer |
| mpv | mpv | ![mpv](static/mpv.svg) | High-performance media player with hardware acceleration |
| PulseAudio Volume Control | pavucontrol-qt | ![multimedia-volume-control](static/multimedia-volume-control.svg) | Audio device and volume management (Qt) |
| Fcitx 5 | fcitx | ![fcitx5](static/fcitx.svg) | Next-generation input method framework |
| QTerminal | qterminal | ![utilities-terminal](static/utilities-terminal.svg) | Lightweight multi-tab terminal emulator |
| Drop-down QTerminal | qterminal | ![utilities-terminal](static/utilities-terminal.svg) | Quake-style drop-down terminal |
| qps | qps | ![qps](static/qps.svg) | Qt-based process viewer and manager |
| Disks | gnome-disk-utility | ![org.gnome.DiskUtility](static/org.gnome.DiskUtility.svg) | Disk and partition management utility |
| KCalc | kcalc | ![accessories-calculator](static/accessories-calculator.svg) | Scientific calculator |
| LXQt Archiver | lxqt-archiver | ![lxqt-archiver](static/lxqt-archiver.svg) | Archive compression and extraction tool |
| PCManFM-Qt | pcmanfm-qt | ![pcmanfm-qt](static/pcmanfm-qt.svg) | Default LXQt file manager |
| QtPass | qtpass | ![qtpass-icon](static/qtpass-icon.svg) | Password manager based on pass and GPG |
| Vim | vim-common | ![gvim](static/gvim.svg) | Highly configurable text editor |
| nobleNote | noblenote | ![noblenote](static/noblenote.svg) | Lightweight notes and checklist application |
| Keyboard Layout Viewer | fcitx5-config-qt | ![input-keyboard](static/input-keyboard.svg) | View and switch keyboard layouts |

## Package Management with APT

`APT` (Advanced Package Tool) is the default package manager used by the system. It is widely used in **Debian** and Debian-based distributions to manage software packages.

APT provides the following core functions:

- **Install software**  
  Download and install packages from official repositories.

- **Update the system**  
  Refresh package indexes and upgrade installed packages.

- **Remove software**  
  Uninstall packages and clean up unused dependencies.

- **Dependency management**  
  Automatically resolves package dependencies and prevents conflicts.

The following image shows the built-in help information for the `apt` command.  
For full command usage and options, refer to [the Debian official manual](https://manpages.debian.org/bookworm/apt/apt-get.8.en.html).

![APT help output](static/aptHelp.png)
