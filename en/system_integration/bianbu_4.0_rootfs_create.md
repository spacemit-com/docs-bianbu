---
sidebar_position: 8
---

# Bianbu 4.0 ROOTFS Creation

This document describes the workflow of creating a custom system image based on **Bianbu 4.0 (bianbu-26.04, built from Ubuntu 26.04, K3 board)**, in two stages:

1. **Create a ROOTFS**: start from the official base rootfs, configure the software repository, install the BSP firmware packages and the variant meta packages, to obtain a complete rootfs directory;
2. **Create the partition images**: configure fstab/UUIDs, split the rootfs, and create the bootfs and rootfs partition images.

Finally, package everything with the unified procedure of the [Image Creation Guide](image.md) into a Titan firmware package (`.tar.gz`) and an SD card image (`.img.gz`), choosing the **4.0 (K3)** notes wherever version differences apply.

## Environment Requirements

1. **Host**: Ubuntu 26.04 (x86_64). It is recommended to run inside a Docker container (root required; start with `--privileged` since the images are mounted as loop devices).
2. **riscv64 execution environment**: packages are installed inside the riscv64 rootfs via chroot. **It is recommended to build directly on a networked K3 board**; on an x86_64 host, install `qemu-user` 10.2+ (Ubuntu 26.04's apt ships qemu 10.x) and register binfmt:

   ```shell
   sudo apt-get -y install qemu-user qemu-user-binfmt
   ```
3. **Build container**: `harbor.spacemit.com/bianbu/bianbu:latest`. Without a container, install `wget uuid-runtime e2fsprogs dosfstools mtools jq pigz genimage zip python3`.

```shell
mkdir ~/bianbu-workspace
docker run --privileged -itd -v ~/bianbu-workspace:/mnt \
    --name build-bianbu-rootfs harbor.spacemit.com/bianbu/bianbu:latest
docker exec -it -w /mnt build-bianbu-rootfs bash
```

## Stage 1: Create a ROOTFS

### 1. Download and extract the official base rootfs

```shell
export BASE_ROOTFS_URL=https://archive.spacemit.com/bianbu-base/bianbu-base-26.04-base-riscv64.tar.gz
wget $BASE_ROOTFS_URL
mkdir -p rootfs && tar -zxpf ${BASE_ROOTFS_URL##*/} -C rootfs
```

### 2. Mount system resources

```shell
mkdir -p rootfs/proc rootfs/sys rootfs/dev/pts
mount -t proc /proc rootfs/proc
mount -t sysfs /sys rootfs/sys
mount -o bind /dev rootfs/dev
mount -o bind /dev/pts rootfs/dev/pts
```

### 3. Configure the repository and basic files

```shell
# Remove the default Ubuntu sources (K3 uses the bianbu repository)
rm -f rootfs/etc/apt/sources.list.d/ubuntu.sources

# Configure the Bianbu 4.0 repository
cat > rootfs/etc/apt/sources.list.d/bianbu.sources <<EOF
Types: deb
URIs: https://archive.spacemit.com/bianbu4/
Suites: resolute resolute-security resolute-updates resolute-backports resolute-porting resolute-customization
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
EOF

# Network and hostname
echo "nameserver 8.8.8.8" > rootfs/etc/resolv.conf
echo bianbu > rootfs/etc/hostname
```

### 4. Install base packages and BSP firmware packages

```shell
chroot rootfs /bin/bash -c "apt-get update"
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades full-upgrade"
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install \
    initramfs-tools base-files fdisk u-boot-spacemit opensbi-spacemit bianbu-esos \
    linux-generic bianbu-minimal ssh"
```

- `u-boot-spacemit` / `opensbi-spacemit`: install the firmware (`u-boot.itb`, `FSBL.bin`, `fw_dynamic.itb`, etc.) into the rootfs for later packaging;
- `linux-generic`: installs the kernel (`/boot/vmlinuz-*` and the device trees in `/boot/spacemit/<version>/`);
- `bianbu-minimal`: the Minimal variant meta package.

### 5. Install variant meta packages

Bianbu 4.0 officially provides two variants: Minimal and LXQt (LXQt also has a `lxqt-uefi` UEFI version). Install according to the target variant:

```shell
# Minimal variant: already done in step 4

# LXQt desktop variant
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y install bianbu-lxqt"
chroot rootfs /bin/bash -c "apt-get purge -y zutty terminator"
```

### 6. Common configuration

```shell
# Locale (zh_CN + en_US)
chroot rootfs /bin/bash -c "apt-get -y install locales"
chroot rootfs /bin/bash -c "echo 'locales locales/locales_to_be_generated multiselect en_US.UTF-8 UTF-8, zh_CN.UTF-8 UTF-8' | debconf-set-selections"
chroot rootfs /bin/bash -c "echo 'locales locales/default_environment_locale select zh_CN.UTF-8' | debconf-set-selections"
chroot rootfs /bin/bash -c "sed -i 's/^# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen"
chroot rootfs /bin/bash -c "dpkg-reconfigure --frontend=noninteractive locales"

# Timezone
chroot rootfs /bin/bash -c "echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections"
chroot rootfs /bin/bash -c "echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections"
chroot rootfs /bin/bash -c "rm -f /etc/timezone && rm -f /etc/localtime"
chroot rootfs /bin/bash -c "dpkg-reconfigure --frontend=noninteractive tzdata"

# Password
chroot rootfs /bin/bash -c "echo root:bianbu | chpasswd"

# Time server
sed -i 's/^#NTP=.*/NTP=ntp.aliyun.com ntp1.aliyun.com ntp2.aliyun.com/' rootfs/etc/systemd/timesyncd.conf

# Network configuration (choose one per variant)
# - Minimal variant (networkd):
cat > rootfs/etc/netplan/01-netcfg.yaml <<EOF
network:
    version: 2
    renderer: networkd
    ethernets:
        end0:
            dhcp4: true
            dhcp-identifier: mac
            critical: true
        end1:
            dhcp4: true
            dhcp-identifier: mac
            critical: true
EOF
chroot rootfs /bin/bash -c "chmod 600 /etc/netplan/01-netcfg.yaml"

# - LXQt desktop variant (NetworkManager; also remove the networkd config above):
cat > rootfs/etc/netplan/01-network-manager-all.yaml <<EOF
network:
   version: 2
   renderer: NetworkManager
EOF
chroot rootfs /bin/bash -c "chmod 600 /etc/netplan/01-network-manager-all.yaml"
rm -f rootfs/etc/netplan/01-netcfg.yaml
```

### 7. Cleanup

```shell
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
mount | grep "rootfs/proc" > /dev/null && umount -l rootfs/proc
mount | grep "rootfs/sys" > /dev/null && umount -l rootfs/sys
mount | grep "rootfs/dev/pts" > /dev/null && umount -l rootfs/dev/pts
mount | grep "rootfs/dev" > /dev/null && umount -l rootfs/dev
```

The `rootfs/` directory is now a ready-to-package ROOTFS.

## Stage 2: Create the Partition Images

### 1. Generate UUIDs and configure /etc/fstab

```shell
export UUID_BOOTFS=$(uuidgen)
export UUID_ROOTFS=$(uuidgen)

cat > rootfs/etc/fstab <<EOF
# <file system>     <dir>    <type>  <options>                          <dump> <pass>
UUID=$UUID_ROOTFS   /        ext4    defaults,noatime,errors=remount-ro 0      1
UUID=$UUID_BOOTFS   /boot    ext4    defaults                           0      2
EOF
```

### 2. Split the bootfs and create the partition images

```shell
mkdir -p bootfs
mv rootfs/boot/* bootfs/

mke2fs -d bootfs -L bootfs -t ext4 -U $UUID_BOOTFS bootfs.ext4 "256M"
mke2fs -d rootfs -L rootfs -t ext4 -N 524288 -U $UUID_ROOTFS rootfs.ext4 "6144M"
```

Recommended rootfs sizes: Minimal `2048M`, LXQt `6144M`; use `524288` inodes.

## Stage 3: Package the Firmware

Run `export TARGET_ROOTFS=rootfs`, then package with the unified procedure of the [Image Creation Guide](image.md), choosing the **4.0 (K3)** notes wherever version differences apply (bootinfo glob copy, additionally copying `esos.itb` / `ec.bin`, partition tables from the `k3-br-v1.0.y` branch, tar.gz package format).

## Flashing and Use

- **SD card image**: decompress and write to an SD card

  ```shell
  sudo zcat Bianbu-LXQt-K3-sdcard-*.img.gz | sudo dd of=/dev/sdX bs=4M conv=fsync status=progress
  ```

- **Titan firmware package**: extract the `.tar.gz` and flash with the Titan tool using `partition_flash.json` (back up your data before flashing).

## Rebuilding After a BSP Update

The kernel, U-Boot, and OpenSBI are distributed as deb packages in the Bianbu repository; see [Kernel Compile](../development/kernel_compile.md). After an update:

1. Put the new deb packages in the Bianbu repository, or install them into the rootfs locally with `dpkg -i`;
2. Re-create `bootfs.ext4` / `rootfs.ext4` (Stage 2), then repackage with the [Image Creation Guide](image.md).

You can also install the new deb packages directly on a running board with `dpkg -i`, without rebuilding the image.
