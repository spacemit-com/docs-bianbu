---
sidebar_position: 1
---

# Kernel Compile

This guide details how to compile the kernel for Buildroot (using `linux-6.6` as an example). Two compilation methods are supported:

- Cross-compilation – faster build time
- Native compilation – more convenient to operate directly on the target system

## Download Kernel Source Code

```shell
git clone https://gitee.com/spacemit-buildroot/linux-6.6.git ~/linux-6.6
```

## Cross-Compilation Method

### Prepare Cross-Development Environment

1. Refer to Buildroot's [Development Environment](https://sdk.spacemit.com/source#development-environment) to set up the cross-compiler.

2. Install required build dependencies with:

    ```shell
    sudo apt-get install debhelper libpfm4-dev libtraceevent-dev asciidoc libelf-dev devscripts
    ```

### Install Cross-Compiler Toolchain

Please visit Toolchain download link: [http://archive.spacemit.com/toolchain/](http://archive.spacemit.com/toolchain/)

1. Download a toolchain archive, e.g., `spacemit-toolchain-linux-glibc-x86_64-v1.0.0.tar.xz`:

2. Extract the toolchain with:

   ```shell
   sudo tar -Jxf /path/to/spacemit-toolchain-linux-glibc-x86_64-v1.0.0.tar.xz -C /opt
   ```

3. Set the cross-compiler environment variables with:

   ```shell
   export PATH=/opt/spacemit-toolchain-linux-glibc-x86_64-v1.0.0/bin:$PATH
   ```

### Cross-Compile the Kernel

Enter the kernel source directory:

```shell
cd ~/linux-6.6
```

Set kernel compilation parameters:

```shell
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
export ARCH=riscv
export LOCALVERSION=""
```

Generate the default configuration:

```shell
make k1_defconfig
```

If you need to build the **PREEMPT_RT real-time kernel** for the `bl-v2.0.y` branch, first update the source tree to commit `3ac79a6dd update rt defconfig` or any later version. Then apply the RT patch and generate the configuration. If not required, you can skip this step:

```shell
patch -p1 < rt-linux/*.patch
make k1_rt_defconfig
```

Modify the configuration if needed; otherwise, you may skip this step:

```shell
make menuconfig
```

Save the modified configuration:

```shell
make savedefconfig
mv defconfig arch/riscv/configs/k1_defconfig
```

Compile the Debian package:

```shell
make -j$(nproc) bindeb-pkg
```

If you see the following messages, the build has completed successfully.

```
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)
```

The resulting `.deb` packages are located in the parent directory. The key packages are:

- `linux-image-6.6.36_6.6.36-*.deb` - Kernel Image package.

- `linux-tools-6.6.36_6.6.36-*.deb` - `perf` and other tool packages.

Copy the packages to the device, install, and then reboot:

```shell
sudo dpkg -i linux-image-6.6.36_6.6.36-*.deb
sudo reboot
```

### Cross Compile Modules

To compile out-of-tree kernel modules, rtl8852bs is the example below, run the command as:

```shell
cd /path/to/rtl8852bs
make -j$(nproc) -C ~/linux-6.6 M=/path/to/rtl8852bs modules
```

- Replace `/path/to/rtl8852bs` with actual module path

Clean command:

```shell
make -j$(nproc) -C ~/linux-6.6 M=/path/to/rtl8852bs clean
```

### Cross Compile Device Tree

To compile the device tree separately:

```shell
make -j$(nproc) dtbs
```

## Native Compilation

You can directly compile the kernel on Bianbu, here is the guide.

### Local Development Environment

Install the dependencies:

```shell
sudo apt-get install flex bison libncurses-dev debhelper libssl-dev u-boot-tools libpfm4-dev libtraceevent-dev asciidoc bc rsync libelf-dev devscripts
```

### Native Compile Kernel

Navigate to  the kernel source directory:

```shell
cd ~/linux-6.6
```

Set kernel compilation parameters:

```shell
export ARCH=riscv
export LOCALVERSION=""
```

Generate default configuration:

```shell
make k1_defconfig
```

Modify the configuration if needed; otherwise, you may skip this step:

```shell
make menuconfig
```

To save the modified configuration:

```shell
make savedefconfig
mv defconfig arch/riscv/configs/k1_defconfig
```

Compile the Debian package:

```shell
make -j$(nproc) bindeb-pkg
```

If you see the following messages, the build has completed successfully.

```
dpkg-genchanges: info: binary-only upload (no source code included)
 dpkg-source --after-build .
dpkg-buildpackage: info: binary-only upload (no source included)
```

The resulting `.deb` packages are located in the parent directory. The key packages are:

- `linux-image-6.6.36_6.6.36-*.deb` - Kernel Image package.

- `linux-tools-6.6.36_6.6.36-*.deb` - `perf` and other tool packages.

Install and then reboot:

```shell
sudo dpkg -i linux-image-6.6.36_6.6.36-*.deb
sudo reboot
```

### Native Module Compilation

To compile out-of-tree kernel modules locally, you can do it without relying on the kernel source.

First, install `linux-headers`:

```shell
sudo apt-get install linux-headers-`uname -r`
```

Then compile the module, for example `rtl8852bs`:

```shell
cd /path/to/rtl8852bs
make -j$(nproc) -C /lib/modules/`uname -r`/build M=/path/to/rtl8852bs modules
```

- Replace `/path/to/rtl8852bs` with your actual module path

Clean command:

```shell
make -j$(nproc) -C /lib/modules/`uname -r`/build M=/path/to/rtl8852bs clean
```

## Build Other Components

### u-boot

Download the source code:

```shell
git clone https://gitee.com/spacemit-buildroot/uboot-2022.10 ~/uboot-2022.10
```

For cross-compilation, configure the following parameters first. Skip for native compilation:

```shell
export PATH=/opt/spacemit-toolchain-linux-glibc-x86_64-v1.0.0/bin:$PATH
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
export ARCH=riscv
```

Compile the Debian package:

```shell
cd ~/uboot-2022.10
VERSION=1~`git rev-parse --short HEAD`
dch --create --package u-boot-spacemit -v ${VERSION} --distribution mantic-porting --force-distribution 'Bianbu Test'
dpkg-buildpackage -us -uc -b
```

The generated Debian package is located in the parent directory. Install it with `dpkg` and reboot to take effect.

### opensbi

Download the source code:

```shell
git clone https://gitee.com/spacemit-buildroot/opensbi ~/opensbi
```

For cross-compilation, configure the following parameters first. Skip for native compilation:

```shell
export PATH=/opt/spacemit-toolchain-linux-glibc-x86_64-v1.0.0/bin:$PATH
export CROSS_COMPILE=riscv64-unknown-linux-gnu-
```

Compile the Debian package:

```shell
cd ~/opensbi
VERSION=1~`git rev-parse --short HEAD`
dch --create --package opensbi-spacemit -v ${VERSION} --distribution mantic-porting --force-distribution 'Bianbu Test'
dpkg-buildpackage -us -uc -b
```

The generated Debian package is located in the parent directory. Install it with `dpkg` and reboot to take effect.
