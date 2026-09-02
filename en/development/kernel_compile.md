---
sidebar_position: 1
---

# Kernel Compile

This guide describes how to build and package the Bianbu kernel and the other BSP components — U-Boot and OpenSBI — which are distributed as standard debian packages (`linux-riscv-spacemit-generic`, `u-boot-spacemit`, `opensbi-spacemit`). The sources are hosted at [gitee.com/spacemit-buildroot](https://gitee.com/spacemit-buildroot). This document describes how to build and package them, using **Bianbu 4.0 (K3)** as an example.

| Component | Source repository | K3 branch | K1 branch |
| --- | --- | --- | --- |
| Kernel | `linux-6.18` (K3) / `linux-6.6` (K1) | `k3-br-v1.0.y` (board config `k3`) | `k1-bl-v2.2.y` (board config `k1`) |
| U-Boot | `uboot-2022.10` | `k3-br-v1.0.y` | `k1-bl-v2.2.y` |
| OpenSBI | `opensbi` | `k3-br-v1.0.y` | `k1-bl-v2.2.y` |

## Environment Requirements

- Cross compilation (x86_64 host): use the official toolchain, download at <http://archive.spacemit.com/toolchain/>, e.g. `spacemit-toolchain-linux-glibc-x86_64-v1.0.0.tar.xz`:

  ```shell
  sudo tar -Jxf /path/to/spacemit-toolchain-linux-glibc-x86_64-v1.0.0.tar.xz -C /opt
  export PATH=/opt/spacemit-toolchain-linux-glibc-x86_64-v1.0.0/bin:$PATH
  ```

- Native compilation (on a K3 board or a riscv64 container/machine): build directly, no cross toolchain needed.
- Install build dependencies:

  ```shell
  sudo apt-get -y install git bison flex bc cpio rsync libncurses-dev \
      debhelper devscripts libssl-dev libssl3 openssl u-boot-tools xmlto asciidoc \
      libelf-dev libdw-dev systemtap-sdt-dev libaudit-dev libslang2-dev libiberty-dev \
      liblzma-dev libcap-dev libnuma-dev python3-dev libbabeltrace-dev libunwind-dev \
      libtraceevent-dev libpfm4-dev pkg-config kmod
  ```

## Quick Build (Script)

All three source repositories ship a cross-compilation packaging script (`scripts/build_kernel.sh` in `linux-6.18`; `scripts/build.sh` in `uboot-2022.10` and `opensbi`) that configures, cross-compiles, and packages the deb in one command:

```shell
cd ~/linux-6.18
./scripts/build_kernel.sh -c -d    # clean build producing deb packages

cd ~/uboot-2022.10
./scripts/build.sh -c -d           # same for U-Boot; identical usage in the opensbi repository
```

Notes:

- **By default the build runs inside a Docker container** (image `harbor.spacemit.com/bianbu/k3-bsp-builder:latest`, with the cross toolchain bundled), so the host needs no toolchain; set `DIRECT_BUILD=1` to build directly on the host instead (toolchain required as above);
- Common options: `-c` clean build, `-d` build deb packages, `-j N` parallel jobs (default `nproc`); commands: `build` (default) and `shell` (interactive container shell);
- The packaging details match the manual flow below: the kernel uses `k3_bianbu_defconfig` with `KERNELRELEASE` derived from the kernel Makefile plus the `-generic` suffix (`KDEB_SOURCENAME=linux-riscv-spacemit-generic`); U-Boot/OpenSBI generate the changelog automatically and run `dpkg-buildpackage -us -uc -b -a riscv64`;
- The deb packages land in the parent directory of the repository.

For custom kernel configurations or building modules/device trees separately, use the manual flow below.

## Kernel

Using K3's `linux-6.18` as an example (for K1 use `linux-6.6`, and replace the repository, branch, and defconfig with the K1 counterparts).

### Fetch the Source

```shell
git clone -b k3-br-v1.0.y https://gitee.com/spacemit-buildroot/linux-6.18 ~/linux-6.18
cd ~/linux-6.18
```

### Cross Compilation

Set up the build parameters:

```shell
export ARCH=riscv
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
```

Generate the default configuration:

```shell
make k3_bianbu_defconfig
```

Modify the configuration (optional):

```shell
make menuconfig
```

To save the modified configuration:

```shell
make savedefconfig
mv defconfig arch/riscv/configs/k3_bianbu_defconfig
```

Build the debian packages:

```shell
KERNELRELEASE=6.18.3 LOCALVERSION=-generic \
KDEB_SOURCENAME=linux-riscv-spacemit-generic \
KDEB_PKGVERSION=6.18.3-$(date +%y%m%d) \
KDEB_CHANGELOG_DIST=stable \
make -j$(nproc) bindeb-pkg
```

- `LOCALVERSION`: kernel version suffix; the official generic kernel uses `-generic`;
- `KDEB_SOURCENAME`/`KDEB_PKGVERSION`: package name and version (follow the Bianbu release naming rules);
- A build succeeds when `dpkg-buildpackage: info: binary-only upload (no source included)` is printed.

The packages land in the parent directory (`..`). Common packages:

- `linux-image-6.18.3-generic_*.deb`: kernel image
- `linux-headers-6.18.3-generic_*.deb`: kernel headers (for building modules)
- `linux-tools-*`: `perf` etc.

Copy to the device, install, and reboot:

```shell
sudo dpkg -i linux-image-6.18.3-generic_*.deb
sudo reboot
```

> Note: on UEFI-booted boards, the postinst runs `update-grub`, which generates entries with devicetree via `/etc/grub.d/09_bianbu_uefi` — no manual steps needed.

### Native Compilation

On a K3 board or in a riscv64 container:

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

### Modules and Device Trees

Build out-of-tree modules (rtl8852bs as an example):

```shell
make -j$(nproc) -C ~/linux-6.18 M=/path/to/rtl8852bs modules
```

- Replace `/path/to/rtl8852bs` with your path.

To build modules locally without the kernel source, install `linux-headers` first:

```shell
sudo apt-get install linux-headers-`uname -r`
cd /path/to/rtl8852bs
make -j$(nproc) -C /lib/modules/`uname -r`/build M=/path/to/rtl8852bs modules
```

Build device trees separately:

```shell
make -j$(nproc) dtbs
```

## U-Boot

### Fetch the Source

```shell
git clone -b k3-br-v1.0.y https://gitee.com/spacemit-buildroot/uboot-2022.10 ~/uboot-2022.10
cd ~/uboot-2022.10
```

### Build Directly

For cross compilation, set the environment variables first (skip for native compilation):

```shell
export ARCH=riscv
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
make k3_defconfig
make -j$(nproc)
```

### Debian Packaging (recommended, matches the official distribution)

The source repository ships a `debian/` packaging directory. Note that `apt-get build-dep` requires enabled `deb-src` repository lines (commented out by default on Ubuntu — uncomment them and run `apt-get update` first):

```shell
rm -f debian/changelog
VERSION=1~$(git rev-parse --short HEAD)
dch --create --package u-boot-spacemit -v ${VERSION} --distribution stable --force-distribution "Bianbu ${VERSION}"
apt-get -o DPkg::Lock::Timeout=600 build-dep -y .
apt-get -o DPkg::Lock::Timeout=600 install -y device-tree-compiler git
dpkg-buildpackage -us -uc -b -a riscv64
```

- **Cross compilation (x86_64 host) must pass `-a riscv64`**: `debian/rules` then sets `CROSS_COMPILE=riscv64-unknown-linux-gnu-` automatically (using the official toolchain installed above); without `-a`, the build uses the host gcc and fails.
- For native compilation (on a K3 board or a riscv64 machine), drop `-a riscv64`.

The resulting `u-boot-spacemit_${VERSION}_all.deb` is in the parent directory. After installation the firmware files live in `/usr/lib/u-boot/spacemit/` on the board system (`bootinfo_block.bin`, `bootinfo_spinand.bin`, `bootinfo_spinor.bin`, `FSBL.bin`, `u-boot.itb`, `env.bin`).

> Note: when installing `u-boot-spacemit`, the postinst script locates the target medium automatically (SD card `/dev/mmcblk0`, eMMC `/dev/mmcblk2`, UFS `/dev/sda`, NAND/NOR `/dev/mtdblock0`, etc., determined from the `boot_mode`/`root` device) and dd-writes `bootinfo_*.bin`, `FSBL.bin`, `env.bin`, `u-boot.itb` to the corresponding firmware partitions; it takes effect after a reboot — no image rebuild needed. Under UEFI boot mode `u-boot.itb` is not written. Make sure the boot parameters point to the correct medium.

## OpenSBI

### Fetch the Source

```shell
git clone -b k3-br-v1.0.y https://gitee.com/spacemit-buildroot/opensbi ~/opensbi
cd ~/opensbi
```

### Debian Packaging

```shell
rm -f debian/changelog
VERSION=1~$(git rev-parse --short HEAD)
dch --create --package opensbi-spacemit -v ${VERSION} --distribution stable --force-distribution "Bianbu ${VERSION}"
apt-get -o DPkg::Lock::Timeout=600 build-dep -y .
apt-get -o DPkg::Lock::Timeout=600 install -y git
dpkg-buildpackage -us -uc -b -a riscv64
```

The usage of `-a riscv64` is the same as for U-Boot (required for cross compilation, dropped for native builds).

The resulting `opensbi-spacemit_${VERSION}_all.deb` is in the parent directory. After installation the firmware lives in `/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb` on the board system.

The `opensbi-spacemit` postinst also writes `fw_dynamic.itb` to the firmware partition at install time; it takes effect after a reboot.

## K1 Platform (Bianbu 2.x)

K1 uses `linux-6.6`:

```shell
git clone -b k1-bl-v2.2.y https://gitee.com/spacemit-buildroot/linux-6.6 ~/linux-6.6
cd ~/linux-6.6
export ARCH=riscv
export CROSS_COMPILE=riscv64-unknown-linux-gnu-   # set for cross compilation; omit for native builds
make k1_defconfig
```

For the PREEMPT_RT real-time kernel, first update the source tree to the commit `3ac79a6dd update rt defconfig` or any later version, then apply the RT patches and generate the configuration:

```shell
patch -p1 < rt-linux/*.patch
make k1_rt_defconfig
```

The remaining build and packaging steps are the same as for K3, with the version-number differences: the K1 kernel version is `6.6.x` (e.g. `KERNELRELEASE=6.6.63`, `KDEB_PKGVERSION=6.6.63-...`), and the official kernel carries no `-generic` suffix (`LOCALVERSION=""`), so the packages are named like `linux-image-6.6.63_6.6.63-*.deb`. For U-Boot/OpenSBI, use the `k1-bl-v2.2.y` branch; everything else is identical.
