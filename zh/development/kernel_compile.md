---
sidebar_position: 1
---

# 内核编译

本指南介绍 Bianbu 内核以及 U-Boot、OpenSBI 等 BSP 组件的编译打包方法，它们均以标准 debian 软件包的形式分发（`linux-riscv-spacemit-generic`、`u-boot-spacemit`、`opensbi-spacemit`）。源码托管在 [gitee.com/spacemit-buildroot](https://gitee.com/spacemit-buildroot)，本文以 **Bianbu 4.0（K3）** 为例介绍编译打包方法。

| 组件 | 源码仓库 | K3 分支 | K1 分支 |
| --- | --- | --- | --- |
| 内核 | `linux-6.18`（K3）/ `linux-6.6`（K1） | `k3-br-v1.0.y`（板型 `k3`） | `k1-bl-v2.2.y`（板型 `k1`） |
| U-Boot | `uboot-2022.10` | `k3-br-v1.0.y` | `k1-bl-v2.2.y` |
| OpenSBI | `opensbi` | `k3-br-v1.0.y` | `k1-bl-v2.2.y` |

## 环境要求

- 交叉编译（x86_64 宿主机）：使用官方工具链，下载地址 <http://archive.spacemit.com/toolchain/>，例如 `spacemit-toolchain-linux-glibc-x86_64-v1.0.0.tar.xz`：

  ```shell
  sudo tar -Jxf /path/to/spacemit-toolchain-linux-glibc-x86_64-v1.0.0.tar.xz -C /opt
  export PATH=/opt/spacemit-toolchain-linux-glibc-x86_64-v1.0.0/bin:$PATH
  ```

- 本地编译（在 K3 板卡或 riscv64 容器/机器上）：直接编译，无需交叉工具链。
- 安装编译依赖：

  ```shell
  sudo apt-get -y install git bison flex bc cpio rsync libncurses-dev \
      debhelper devscripts libssl-dev libssl3 openssl u-boot-tools xmlto asciidoc \
      libelf-dev libdw-dev systemtap-sdt-dev libaudit-dev libslang2-dev libiberty-dev \
      liblzma-dev libcap-dev libnuma-dev python3-dev libbabeltrace-dev libunwind-dev \
      libtraceevent-dev libpfm4-dev pkg-config kmod
  ```

## 快速编译（脚本）

三个源码仓库均自带交叉编译打包脚本（`linux-6.18` 为 `scripts/build_kernel.sh`，`uboot-2022.10` 与 `opensbi` 为 `scripts/build.sh`），一条命令即可完成配置、交叉编译与 deb 打包：

```shell
cd ~/linux-6.18
./scripts/build_kernel.sh -c -d    # 清理后编译并生成 deb 包

cd ~/uboot-2022.10
./scripts/build.sh -c -d           # U-Boot 同理；opensbi 仓库中脚本用法相同
```

说明：

- **默认在 Docker 容器中编译**（镜像 `harbor.spacemit.com/bianbu/k3-bsp-builder:latest`，已内置交叉工具链），宿主机无需安装工具链；设置 `DIRECT_BUILD=1` 则直接在本机编译，需按上文安装工具链；
- 常用参数：`-c` 清理后编译，`-d` 生成 deb 包，`-j N` 并行任务数（默认 `nproc`）；子命令 `build`（默认）编译，`shell` 进入容器交互环境；
- 打包细节与下文手工流程一致：内核使用 `k3_bianbu_defconfig`，`KERNELRELEASE` 自动取自内核 Makefile 并附加 `-generic` 后缀（`KDEB_SOURCENAME=linux-riscv-spacemit-generic`）；U-Boot/OpenSBI 自动生成 changelog 并执行 `dpkg-buildpackage -us -uc -b -a riscv64`；
- 生成的 deb 包位于仓库上一层目录。

如需自定义内核配置或单独编译模块/设备树，参考下文的手工流程。

## 内核编译

以 K3 的 `linux-6.18` 为例（K1 使用 `linux-6.6`，把仓库、分支与 defconfig 换成 K1 对应值即可）。

### 下载源码

```shell
git clone -b k3-br-v1.0.y https://gitee.com/spacemit-buildroot/linux-6.18 ~/linux-6.18
cd ~/linux-6.18
```

### 交叉编译

设置编译参数：

```shell
export ARCH=riscv
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
```

生成默认配置：

```shell
make k3_bianbu_defconfig
```

修改配置（可跳过）：

```shell
make menuconfig
```

如需保存修改后的配置：

```shell
make savedefconfig
mv defconfig arch/riscv/configs/k3_bianbu_defconfig
```

编译 debian 软件包：

```shell
KERNELRELEASE=6.18.3 LOCALVERSION=-generic \
KDEB_SOURCENAME=linux-riscv-spacemit-generic \
KDEB_PKGVERSION=6.18.3-$(date +%y%m%d) \
KDEB_CHANGELOG_DIST=stable \
make -j$(nproc) bindeb-pkg
```

- `LOCALVERSION`：内核版本后缀，官方 generic 内核为 `-generic`；
- `KDEB_SOURCENAME`/`KDEB_PKGVERSION`：软件包名与版本（软件包版本号请按 Bianbu 发布的规则填写）；
- 当看到 `dpkg-buildpackage: info: binary-only upload (no source included)` 说明编译成功。

软件包位于上一层目录（`..`），常用包：

- `linux-image-6.18.3-generic_*.deb`：内核 Image 软件包
- `linux-headers-6.18.3-generic_*.deb`：内核头文件（编译模块使用）
- `linux-tools-*`：`perf` 等工具

拷贝到设备安装，然后重启：

```shell
sudo dpkg -i linux-image-6.18.3-generic_*.deb
sudo reboot
```

> 提示：UEFI 启动的板卡升级内核后，`update-grub`（postinst 自动执行）会通过 `/etc/grub.d/09_bianbu_uefi` 生成带 devicetree 的启动条目，无需手动操作。

### 本地编译

在 K3 板卡或 riscv64 容器中：

```shell
cd ~/linux-6.18
export ARCH=riscv
make k3_bianbu_defconfig
KERNELRELEASE=6.18.3 LOCALVERSION=-generic \
KDEB_SOURCENAME=linux-riscv-spacemit-generic \
KDEB_PKGVERSION=6.18.3-$(date +%y%m%d) \
KDEB_CHANGELOG_DIST=stable \
make -j$(nproc) bindeb-pkg
```

### 编译模块与设备树

编译内核源码树外的模块（以 rtl8852bs 为例）：

```shell
make -j$(nproc) -C ~/linux-6.18 M=/path/to/rtl8852bs modules
```

- `/path/to/rtl8852bs` 请替换为您的路径

本地编译模块也可以不依赖内核源码，先安装 `linux-headers`：

```shell
sudo apt-get install linux-headers-`uname -r`
cd /path/to/rtl8852bs
make -j$(nproc) -C /lib/modules/`uname -r`/build M=/path/to/rtl8852bs modules
```

单独编译设备树：

```shell
make -j$(nproc) dtbs
```

## U-Boot 编译

### 下载源码

```shell
git clone -b k3-br-v1.0.y https://gitee.com/spacemit-buildroot/uboot-2022.10 ~/uboot-2022.10
cd ~/uboot-2022.10
```

### 直接编译

交叉编译请先设置环境变量（本地编译可跳过）：

```shell
export ARCH=riscv
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
make k3_defconfig
make -j$(nproc)
```

### Debian 打包（推荐，与官方分发一致）

源码仓库自带 `debian/` 打包配置。注意 `apt-get build-dep` 需要启用 `deb-src` 源（Ubuntu 默认注释掉了，需先放开注释并 `apt-get update`）：

```shell
rm -f debian/changelog
VERSION=1~$(git rev-parse --short HEAD)
dch --create --package u-boot-spacemit -v ${VERSION} --distribution stable --force-distribution "Bianbu ${VERSION}"
apt-get -o DPkg::Lock::Timeout=600 build-dep -y .
apt-get -o DPkg::Lock::Timeout=600 install -y device-tree-compiler git
dpkg-buildpackage -us -uc -b -a riscv64
```

- **交叉编译（x86_64 宿主机）必须带 `-a riscv64`**：此时 `debian/rules` 会自动设置 `CROSS_COMPILE=riscv64-unknown-linux-gnu-`（使用上面安装的官方工具链）；不带 `-a` 时会用宿主机 gcc 编译而失败。
- 本地编译（K3 板卡或 riscv64 环境）去掉 `-a riscv64`。

生成的 `u-boot-spacemit_${VERSION}_all.deb` 位于上一层目录，安装后固件文件位于板载系统的 `/usr/lib/u-boot/spacemit/`（`bootinfo_block.bin`、`bootinfo_spinand.bin`、`bootinfo_spinor.bin`、`FSBL.bin`、`u-boot.itb`、`env.bin`）。

> 提示：安装 `u-boot-spacemit` 时，postinst 脚本会根据启动方式自动定位目标介质（SD 卡 `/dev/mmcblk0`、eMMC `/dev/mmcblk2`、UFS `/dev/sda`、NAND/NOR `/dev/mtdblock0` 等，由 `boot_mode`/`root` 设备确定），将 `bootinfo_*.bin`、`FSBL.bin`、`env.bin`、`u-boot.itb` 直接 dd 写入相应固件分区，安装后重启生效，无需重制镜像；UEFI 启动模式下不写入 `u-boot.itb`。请确认启动参数指向正确的介质。

## OpenSBI 编译

### 下载源码

```shell
git clone -b k3-br-v1.0.y https://gitee.com/spacemit-buildroot/opensbi ~/opensbi
cd ~/opensbi
```

### Debian 打包

```shell
rm -f debian/changelog
VERSION=1~$(git rev-parse --short HEAD)
dch --create --package opensbi-spacemit -v ${VERSION} --distribution stable --force-distribution "Bianbu ${VERSION}"
apt-get -o DPkg::Lock::Timeout=600 build-dep -y .
apt-get -o DPkg::Lock::Timeout=600 install -y git
dpkg-buildpackage -us -uc -b -a riscv64
```

`-a riscv64` 的用法与 U-Boot 一致（交叉编译必带，本地编译去掉）。

生成的 `opensbi-spacemit_${VERSION}_all.deb` 位于上一层目录，安装后固件位于板载系统的 `/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb`。

`opensbi-spacemit` 的 postinst 同样会在安装时将 `fw_dynamic.itb` 写入固件分区，安装后重启生效。

## K1 平台（Bianbu 2.x）

K1 使用 `linux-6.6`：

```shell
git clone -b k1-bl-v2.2.y https://gitee.com/spacemit-buildroot/linux-6.6 ~/linux-6.6
cd ~/linux-6.6
export ARCH=riscv
export CROSS_COMPILE=riscv64-unknown-linux-gnu-   # 交叉编译时设置，本地编译省略
make k1_defconfig
```

如果需要编译 PREEMPT_RT 实时内核，请先将源码更新到提交 `3ac79a6dd update rt defconfig` 或之后的版本，然后打 RT 补丁再生成配置：

```shell
patch -p1 < rt-linux/*.patch
make k1_rt_defconfig
```

其余编译、打包流程与 K3 相同，注意版本号的差异：K1 内核版本号为 `6.6.x`（如 `KERNELRELEASE=6.6.63`、`KDEB_PKGVERSION=6.6.63-...`），且官方内核不带 `-generic` 后缀（`LOCALVERSION=""`），生成的软件包形如 `linux-image-6.6.63_6.6.63-*.deb`。U-Boot/OpenSBI 换用 `k1-bl-v2.2.y` 分支，其余一致。
