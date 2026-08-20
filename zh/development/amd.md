---
sidebar_position: 9
---

# AMD 显卡使用指南

## 支持平台

| 平台 | 系统版本 | Mesa 版本 | R600 显卡 | Radeonsi 显卡 |
| --- | --- | --- | --- | --- |
| K1 | Bianbu 2.x | 24.01-bb3 | 支持 | 支持 |
| K3 | Bianbu 4.x | 26.0.3 | 支持 | 支持 |

说明：

- R600 指的是使用 radeon 内核驱动和 Mesa r600 用户态驱动的 TeraScale 架构显卡（Radeon HD 2000 ~ 6000 系列）。
- Radeonsi 是使用 amdgpu 内核驱动和 Mesa radeonsi 用户态驱动的现代显卡架构（如 Polaris、Vega 及 RDNA 系列），与 R600 采用的是完全不同的驱动栈。
- K1 的 Mesa（24.01-bb3）默认没有编译 radeonsi 驱动，在 K1 上使用 Radeonsi 显卡则需安装上游原版Mesa。

## K1（Bianbu 2.0）

### 修改环境变量

注释掉 `MESA_LOADER_DRIVER_OVERRIDE=pvr` 让 Mesa 按实际显卡自动选择对应驱动：

```shell
sudo sed -i '/^[[:space:]]*MESA_LOADER_DRIVER_OVERRIDE=pvr/s/^/#/' /etc/environment
```

### 安装固件包

Bianbu 2.x 的软件源中没有 linux-firmware-amd-graphics 这个拆分的 AMD 显卡固件包，需安装完整的 linux-firmware 包（约 500MB），其中已包含 radeon/ 与 amdgpu/ 目录下的固件文件。固件由内核启动的时候加载。

```shell
sudo apt install linux-firmware
```

## K3（Bianbu 4.0）

### 修改环境变量

同样将 `MESA_LOADER_DRIVER_OVERRIDE=pvr` 注释掉：

```shell
sudo sed -i '/^[[:space:]]*MESA_LOADER_DRIVER_OVERRIDE=pvr/s/^/#/' /etc/environment
```

### 安装固件包

linux-firmware-amd-graphics 是包含 AMD 显卡固件的软件包。安装后 R600 的固件位于 `/lib/firmware/radeon/` 目录，Radeonsi 的固件位于 `/lib/firmware/amdgpu/` 目录（polaris12_*.bin）。固件由内核启动的时候加载。

```shell
sudo apt install linux-firmware-amd-graphics
```

### 修改 libglvnd vendor 配置

Bianbu 4.0 出厂时 `/usr/share/glvnd/egl_vendor.d/` 目录下同时存在 `40_pvr.json`、`50_mesa.json` 两个 vendor 配置文件，其中 `40_pvr.json` 指向 `libEGL_pvr.so`（芯片内置 PVR 集显的 EGL 实现）。`libglvnd` 按文件名的数字升序依次尝试 vendor，`40_pvr.json` 的优先级高于系统 Mesa 的 `50_mesa.json`，PVR vendor 会先于系统 Mesa 尝试初始化，影响 AMD 显卡的正常使用。

需要修改 `40_pvr.json` 的优先级，确保先加载 `50_mesa.json`：

```shell
sudo mv /usr/share/glvnd/egl_vendor.d/40_pvr.json /usr/share/glvnd/egl_vendor.d/60_pvr.json
```

修改后 libglvnd 的 vendor 尝试顺序为 `50_mesa.json` → `60_pvr.json`：

- AMD 显卡由系统 Mesa 的 radeonsi 驱动；
- PVR 集显在系统 Mesa 初始化失败后由 PVR vendor 接管；
- 两者互不干扰。

注意：`40_pvr.json` 属于 libegl-pvr0 软件包，升级该包后此文件可能被恢复，如果恢复需再次执行上述改名操作。

## 重启

为了使环境变量的配置生效，并且使 linux 内核可以加载新安装的固件，需要重启主机。

```shell
sudo reboot
```

将显卡接到 pcie 接口上，然后开机，即可使用。

## 验证

确认显卡已被系统识别：

```shell
lspci | grep -i amd
```

登录桌面后，运行 `eglinfo` 查看当前使用的显卡驱动：

```shell
eglinfo | grep -i renderer
```
