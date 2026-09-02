---
sidebar_position: 8
---

# Bianbu 4.0 ROOTFS 制作

本文介绍基于 **Bianbu 4.0（bianbu-26.04，基于 Ubuntu 26.04，K3 板卡）** 制作自定义系统镜像的流程，共分两个阶段：

1. **制作 ROOTFS**：从官方基础 rootfs 开始，配置软件源、安装 BSP 固件包与变体元包，得到完整的 rootfs 目录；
2. **生成分区镜像**：配置 fstab/UUID，将 rootfs 拆分并生成为 bootfs 与 rootfs 分区镜像。

之后按 [固件制作指南](image.md) 的统一流程打包成 Titan 固件包（`.tar.gz`）与 SD 卡镜像（`.img.gz`），版本差异处均选择 **4.0（K3）** 的说明。

## 环境要求

1. **宿主机**：Ubuntu 26.04（x86_64），建议在 Docker 容器内执行（需 root；容器以 `--privileged` 启动，过程中会挂载分区镜像）。
2. **riscv64 执行环境**：需要 chroot 进入 riscv64 rootfs 安装软件包。**推荐直接在可以联网的 K3 板卡上制作**；如在 x86_64 宿主机上制作，需安装 `qemu-user` 10.2+（建议在 Ubuntu 26.04 中通过 apt 安装，自带 qemu 10.x）并注册 binfmt：

   ```shell
   sudo apt-get -y install qemu-user qemu-user-binfmt
   ```
3. **构建容器**：`harbor.spacemit.com/bianbu/bianbu:latest`，已内置网络、工具链等；如不用容器，请安装 `wget uuid-runtime e2fsprogs dosfstools mtools jq pigz genimage zip python3`。

```shell
mkdir ~/bianbu-workspace
docker run --privileged -itd -v ~/bianbu-workspace:/mnt \
    --name build-bianbu-rootfs harbor.spacemit.com/bianbu/bianbu:latest
docker exec -it -w /mnt build-bianbu-rootfs bash
```

## 一、制作 ROOTFS

### 1. 下载并解压官方基础 rootfs

```shell
export BASE_ROOTFS_URL=https://archive.spacemit.com/bianbu-base/bianbu-base-26.04-base-riscv64.tar.gz
wget $BASE_ROOTFS_URL
mkdir -p rootfs && tar -zxpf ${BASE_ROOTFS_URL##*/} -C rootfs
```

### 2. 挂载系统资源

```shell
mkdir -p rootfs/proc rootfs/sys rootfs/dev/pts
mount -t proc /proc rootfs/proc
mount -t sysfs /sys rootfs/sys
mount -o bind /dev rootfs/dev
mount -o bind /dev/pts rootfs/dev/pts
```

### 3. 配置软件源与基础文件

```shell
# 移除 Ubuntu 默认源（K3 使用 bianbu 源）
rm -f rootfs/etc/apt/sources.list.d/ubuntu.sources

# 配置 Bianbu 4.0 软件源
cat > rootfs/etc/apt/sources.list.d/bianbu.sources <<EOF
Types: deb
URIs: https://archive.spacemit.com/bianbu4/
Suites: resolute resolute-security resolute-updates resolute-backports resolute-porting resolute-customization
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
EOF

# 网络与主机名
echo "nameserver 8.8.8.8" > rootfs/etc/resolv.conf
echo bianbu > rootfs/etc/hostname
```

### 4. 安装基础包与 BSP 固件包

```shell
chroot rootfs /bin/bash -c "apt-get update"
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades full-upgrade"
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install \
    initramfs-tools base-files fdisk u-boot-spacemit opensbi-spacemit bianbu-esos \
    linux-generic bianbu-minimal ssh"
```

- `u-boot-spacemit` / `opensbi-spacemit`：将 `u-boot.itb`、`FSBL.bin`、`fw_dynamic.itb` 等固件安装到 rootfs，供后续打包使用；
- `linux-generic`：安装内核（`/boot/vmlinuz-*` 与 `/boot/spacemit/<版本>/` 设备树）；
- `bianbu-minimal`：Minimal 变体元包。

### 5. 安装变体元包

Bianbu 4.0 官方提供 Minimal 与 LXQt 两种变体（LXQt 另提供 `lxqt-uefi` UEFI 版本）。按目标变体安装：

```shell
# Minimal 变体：第 4 步已完成

# LXQt 桌面变体
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y install bianbu-lxqt"
chroot rootfs /bin/bash -c "apt-get purge -y zutty terminator"
```

### 6. 通用配置

```shell
# 配置 locale（zh_CN + en_US）
chroot rootfs /bin/bash -c "apt-get -y install locales"
chroot rootfs /bin/bash -c "echo 'locales locales/locales_to_be_generated multiselect en_US.UTF-8 UTF-8, zh_CN.UTF-8 UTF-8' | debconf-set-selections"
chroot rootfs /bin/bash -c "echo 'locales locales/default_environment_locale select zh_CN.UTF-8' | debconf-set-selections"
chroot rootfs /bin/bash -c "sed -i 's/^# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen"
chroot rootfs /bin/bash -c "dpkg-reconfigure --frontend=noninteractive locales"

# 配置时区
chroot rootfs /bin/bash -c "echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections"
chroot rootfs /bin/bash -c "echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections"
chroot rootfs /bin/bash -c "rm -f /etc/timezone && rm -f /etc/localtime"
chroot rootfs /bin/bash -c "dpkg-reconfigure --frontend=noninteractive tzdata"

# 配置密码
chroot rootfs /bin/bash -c "echo root:bianbu | chpasswd"

# 配置时间服务器
sed -i 's/^#NTP=.*/NTP=ntp.aliyun.com ntp1.aliyun.com ntp2.aliyun.com/' rootfs/etc/systemd/timesyncd.conf

# 网络配置（按变体二选一）
# - Minimal 变体（networkd）：
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

# - LXQt 桌面变体（NetworkManager，并移除上面的 networkd 配置）：
cat > rootfs/etc/netplan/01-network-manager-all.yaml <<EOF
network:
   version: 2
   renderer: NetworkManager
EOF
chroot rootfs /bin/bash -c "chmod 600 /etc/netplan/01-network-manager-all.yaml"
rm -f rootfs/etc/netplan/01-netcfg.yaml
```

### 7. 收尾

```shell
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
mount | grep "rootfs/proc" > /dev/null && umount -l rootfs/proc
mount | grep "rootfs/sys" > /dev/null && umount -l rootfs/sys
mount | grep "rootfs/dev/pts" > /dev/null && umount -l rootfs/dev/pts
mount | grep "rootfs/dev" > /dev/null && umount -l rootfs/dev
```

此时 `rootfs/` 目录即为可打包的 ROOTFS。

## 二、生成分区镜像

### 1. 生成 UUID 并配置 /etc/fstab

```shell
export UUID_BOOTFS=$(uuidgen)
export UUID_ROOTFS=$(uuidgen)

cat > rootfs/etc/fstab <<EOF
# <file system>     <dir>    <type>  <options>                          <dump> <pass>
UUID=$UUID_ROOTFS   /        ext4    defaults,noatime,errors=remount-ro 0      1
UUID=$UUID_BOOTFS   /boot    ext4    defaults                           0      2
EOF
```

### 2. 拆分 bootfs 并生成分区镜像

```shell
mkdir -p bootfs
mv rootfs/boot/* bootfs/

mke2fs -d bootfs -L bootfs -t ext4 -U $UUID_BOOTFS bootfs.ext4 "256M"
mke2fs -d rootfs -L rootfs -t ext4 -N 524288 -U $UUID_ROOTFS rootfs.ext4 "6144M"
```

rootfs 大小建议：Minimal `2048M`、LXQt `6144M`；inode 数可用 `524288`。

## 三、打包固件

执行 `export TARGET_ROOTFS=rootfs` 后，按 [固件制作指南](image.md) 的统一流程打包即可，版本差异处均选择 **4.0（K3）** 的说明（bootinfo 通配拷贝、额外拷贝 `esos.itb` / `ec.bin`、分区表使用 `k3-br-v1.0.y` 分支、tar.gz 打包格式）。

## 烧写与使用

- **SD 卡镜像**：解压后写入 SD 卡

  ```shell
  sudo zcat Bianbu-LXQt-K3-sdcard-*.img.gz | sudo dd of=/dev/sdX bs=4M conv=fsync status=progress
  ```

- **Titan 固件包**：解压 `.tar.gz`，使用 Titan 工具选择 `partition_flash.json` 刷写（刷写前请备份数据）。

## 更新 BSP 包后重新制作

内核、U-Boot、OpenSBI 以 deb 包形式发布在 Bianbu 软件源中，编译方法见 [内核编译](../development/kernel_compile.md)。更新后：

1. 将新 deb 包放入 Bianbu 源，或先在本地 `dpkg -i` 到 rootfs 中；
2. 重新生成 `bootfs.ext4` / `rootfs.ext4`（步骤二），再按 [固件制作指南](image.md) 重新打包。

也可以在已启动的板卡上直接 `dpkg -i` 安装新 deb 包，无需重制镜像。
