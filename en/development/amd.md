---
sidebar_position: 9
---

# AMD Graphics Card Usage Guide

## Supported Platforms

| Platform | System | Mesa | R600 | Radeonsi |
| --- | --- | --- | --- | --- |
| K1 | Bianbu 2.x | 24.01-bb3 | Supported | Supported |
| K3 | Bianbu 4.x | 26.0.3 | Supported | Supported |

Notes:

- R600 refers to TeraScale cards driven by the radeon kernel driver and the Mesa r600 userspace driver (Radeon HD 2000 ~ 6000 series).
- Radeonsi refers to modern cards driven by the amdgpu kernel driver and the Mesa radeonsi userspace driver (e.g. Polaris, Vega, and RDNA series), a completely different driver stack from R600.
- K1's shipped Mesa (24.01-bb3) is built without radeonsi; using Radeonsi cards on K1 requires installing upstream Mesa.

## K1 (Bianbu 2.0)

### Modify Environment Variables

Comment out `MESA_LOADER_DRIVER_OVERRIDE=pvr` so that Mesa automatically selects the driver for the installed card:

```shell
sudo sed -i '/^[[:space:]]*MESA_LOADER_DRIVER_OVERRIDE=pvr/s/^/#/' /etc/environment
```

### Install the Firmware Package

Bianbu 2.x repositories do not include the linux-firmware-amd-graphics split package. Install the full linux-firmware package (~500 MB), which contains both radeon/ (R600) and amdgpu/ (Radeonsi) firmware. The firmware is loaded by the kernel at boot.

```shell
sudo apt install linux-firmware
```

## K3 (Bianbu 4.0)

### Modify Environment Variables

Comment out `MESA_LOADER_DRIVER_OVERRIDE=pvr` in the same way:

```shell
sudo sed -i '/^[[:space:]]*MESA_LOADER_DRIVER_OVERRIDE=pvr/s/^/#/' /etc/environment
```

### Install the Firmware Package

linux-firmware-amd-graphics contains the AMD GPU firmware: R600 firmware in `/lib/firmware/radeon/`, Radeonsi firmware in `/lib/firmware/amdgpu/` (polaris12_*.bin). The firmware is loaded by the kernel at boot.

```shell
sudo apt install linux-firmware-amd-graphics
```

### Modify the libglvnd Vendor Configuration

Bianbu 4.0 ships two vendor files under `/usr/share/glvnd/egl_vendor.d/`: `40_pvr.json` and `50_mesa.json`. `40_pvr.json` points to `libEGL_pvr.so`, the EGL implementation for the on-chip PVR GPU. `libglvnd` tries vendors in ascending filename order, so `40_pvr.json` has higher priority than the system Mesa's `50_mesa.json`; the PVR vendor initializes before Mesa, which breaks AMD cards.

Lower the priority of `40_pvr.json` so that `50_mesa.json` is tried first:

```shell
sudo mv /usr/share/glvnd/egl_vendor.d/40_pvr.json /usr/share/glvnd/egl_vendor.d/60_pvr.json
```

The vendor order becomes `50_mesa.json` → `60_pvr.json`:

- AMD cards are driven by system Mesa's radeonsi;
- The PVR GPU is taken over by the PVR vendor after system Mesa fails to initialize;
- The two do not interfere with each other.

Note: `40_pvr.json` belongs to the libegl-pvr0 package. Upgrading that package may restore the file; if so, move it again.

## Reboot

Reboot the host to apply the environment variables and let the kernel load the newly installed firmware.

```shell
sudo reboot
```

Connect the graphics card to the PCIe slot, then power on.

## Verification

Confirm the graphics card is detected:

```shell
lspci | grep -i amd
```

After logging into the desktop, run `eglinfo` to check the active graphics driver:

```shell
eglinfo | grep -i renderer
```
