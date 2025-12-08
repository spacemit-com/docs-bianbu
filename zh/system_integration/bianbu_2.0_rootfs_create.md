---
sidebar_position: 2
---

# Bianbu 2.0 ROOTFS制作

建议使用 [bianbu 2.1/2.2 ROOTFS制作](./bianbu_2.1_rootfs_create.md)。

## 环境要求

宿主机推荐 Ubuntu 20.04/22.04，且安装了 docker ce 和 qemu-user-static（8.0.4，定制版，默认开启了 Vector 1.0 支持）。

### docker

docker ce 安装可参考 [https://docs.docker.com/engine/install/](https://docs.docker.com/engine/install/) 。

### qemu

1. 卸载 binfmt-support

   定制版的 qemu-user-static 与 binfmt-support 有冲突，因为 binfmt-support 提供的 `/etc/init.d/binfmt-support` 属于传统的 SysVinit 启动脚本，而定制版的 qemu-user-static 提供的 `/lib/systemd/system/systemd-binfmt.service` 是 systemd 服务单元文件。`/etc/init.d/binfmt-support` 会晚于 `/lib/systemd/system/systemd-binfmt.service` 执行，导致覆盖 systemd 的设置。

   ```shell
   sudo apt-get purge binfmt-support
   ```

2. 下载定制版的 qemu

   ```shell
   wget https://archive.spacemit.com/qemu/qemu-user-static_8.0.4%2Bdfsg-1ubuntu3.23.10.1_amd64.deb
   ```

3. 安装定制版的 qemu

   ```shell
   sudo dpkg -i qemu-user-static_8.0.4+dfsg-1ubuntu3.23.10.1_amd64.deb
   ```

4. 注册 qemu-user-static 到内核，这样整个系统范围（含容器）均可以直接执行 riscv 的二进制文件

   ```shell
   sudo systemctl restart systemd-binfmt.service
   ```

5. 验证 qemu-user-static 是否注册成功

   ```shell
   wget https://archive.spacemit.com/qemu/rvv
   chmod a+x rvv
   ./rvv
   ```

   出现以下信息表示注册成功。

   ```
   helloworld
   spacemit
   ```

## 准备基础 rootfs

1. 创建工作目录

   ```shell
   mkdir ~/bianbu-workspace
   ```

2. 创建并启动容器

   ```shell
   docker run --privileged -itd -v ~/bianbu-workspace:/mnt --name build-bianbu-rootfs harbor.spacemit.com/bianbu/bianbu:latest
   ```

3. 进入容器

   ```shell
   docker exec -it -w /mnt build-bianbu-rootfs bash
   ```

4. 安装基本工具

   ```shell
   apt-get update
   apt-get -y install wget uuid-runtime
   ```

5. 配置环境变量，方便后续命令使用

   ```shell
   export BASE_ROOTFS_URL=https://archive.spacemit.com/bianbu-base/bianbu-base-24.04-base-riscv64.tar.gz
   export BASE_ROOTFS=$(basename "$BASE_ROOTFS_URL")
   export TARGET_ROOTFS=rootfs
   ```

6. 下载

   ```shell
   wget $BASE_ROOTFS_URL
   ```

7. 解压到指定目录

   ```shell
   mkdir -p $TARGET_ROOTFS && tar -zxpf $BASE_ROOTFS -C $TARGET_ROOTFS
   ```

8. 挂载一些系统资源到 rootfs 中

   ```shell
   mount -t proc /proc $TARGET_ROOTFS/proc
   mount -t sysfs /sys $TARGET_ROOTFS/sys
   mount -o bind /dev $TARGET_ROOTFS/dev
   mount -o bind /dev/pts $TARGET_ROOTFS/dev/pts
   ```

## 必要配置

### 配置源

1. 首先配置环境变量，方便后续命令使用

   ```shell
   export REPO="archive.spacemit.com/bianbu"
   export VERSION="v2.0.4"
   ```

   [点击查看最新VERSION](../release_notes/bianbu_2.0.md)

2. 配置 bianbu.sources

   ```shell
   cat <<EOF | tee $TARGET_ROOTFS/etc/apt/sources.list.d/bianbu.sources
   Types: deb
   URIs: https://$REPO/
   Suites: noble/snapshots/$VERSION noble-security/snapshots/$VERSION noble-porting/snapshots/$VERSION noble-customization/snapshots/$VERSION
   Components: main universe restricted multiverse
   Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
   EOF
   ```

### 配置 DNS

```shell
echo "nameserver 8.8.8.8" >$TARGET_ROOTFS/etc/resolv.conf
```

### 安装硬件相关的包

```shell
chroot $TARGET_ROOTFS /bin/bash -c "apt-get update"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades upgrade"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install initramfs-tools"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-esos img-gpu-powervr k1x-vpu-firmware k1x-cam spacemit-uart-bt spacemit-modules-usrload opensbi-spacemit u-boot-spacemit linux-image-6.6.36"
```

### 安装元包

不同变体有不同的元包，

- Minimal：bianbu-minimal
- Dekstop：bianbu-desktop bianbu-desktop-zh bianbu-desktop-en bianbu-desktop-minimal-en bianbu-standard bianbu-development
- NAS：bianbu-nas

Dekstop和NAS都是基于Minimal的，建议先安装Mnimal元包再安装Dekstop元包。

这里以制作最小的 minimal 变体为例：

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-minimal"
```

提示：完成全部包的安装后可以执行如下命令清理一下缓存，以减少最终固件的大小

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
```

## 通用配置

#### 配置地区

```shell
chroot $TARGET_ROOTFS /bin/bash -c "apt-get -y install locales"
chroot $TARGET_ROOTFS /bin/bash -c "echo \"locales locales/locales_to_be_generated multiselect en_US.UTF-8 UTF-8, zh_CN.UTF-8 UTF-8\" | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "echo \"locales locales/default_environment_locale select zh_CN.UTF-8\" | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "sed -i 's/^# zh_CN.UTF-8 UTF-8/zh_CN.UTF-8 UTF-8/' /etc/locale.gen"
chroot $TARGET_ROOTFS /bin/bash -c "dpkg-reconfigure --frontend=noninteractive locales"
```

#### 配置时区

```shell
chroot $TARGET_ROOTFS /bin/bash -c "echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections"
chroot $TARGET_ROOTFS /bin/bash -c "rm /etc/timezone"
chroot $TARGET_ROOTFS /bin/bash -c "rm /etc/localtime"
chroot $TARGET_ROOTFS /bin/bash -c "dpkg-reconfigure --frontend=noninteractive tzdata"
```

#### 配置时间服务器

```shell
sed -i 's/^#NTP=.*/NTP=ntp.aliyun.com/' $TARGET_ROOTFS/etc/systemd/timesyncd.conf
```

#### 配置密码

```shell
chroot $TARGET_ROOTFS /bin/bash -c "echo root:bianbu | chpasswd"
```

#### 配置网络

- minimal

```shell
cat <<EOF | tee $TARGET_ROOTFS/etc/netplan/01-netcfg.yaml
network:
    version: 2
    renderer: networkd
    ethernets:
        end0:
            dhcp4: true
        end1:
            dhcp4: true
EOF
chroot $TARGET_ROOTFS /bin/bash -c "chmod 600 /etc/netplan/01-netcfg.yaml"
```

- desktop

```shell
cat <<EOF | tee $TARGET_ROOTFS/etc/netplan/01-network-manager-all.yaml
# Let NetworkManager manage all devices on this system
network:
version: 2
renderer: NetworkManager
EOF
chroot $TARGET_ROOTFS /bin/bash -c "chmod 600 /etc/netplan/01-network-manager-all.yaml"
```

注意：不同变体只需要配置各自的文件即可。

## 生成分区镜像

注意安装配置完后，先取消挂载！

```shell
mount | grep "$TARGET_ROOTFS/proc" > /dev/null && umount -l $TARGET_ROOTFS/proc
mount | grep "$TARGET_ROOTFS/sys" > /dev/null && umount -l $TARGET_ROOTFS/sys
mount | grep "$TARGET_ROOTFS/dev/pts" > /dev/null && umount -l $TARGET_ROOTFS/dev/pts
mount | grep "$TARGET_ROOTFS/dev" > /dev/null && umount -l $TARGET_ROOTFS/dev
```

生成 UUID,并写入/etc/fstab

```shell
UUID_BOOTFS=$(uuidgen)
UUID_ROOTFS=$(uuidgen)
cat >$TARGET_ROOTFS/etc/fstab <<EOF
# <file system>     <dir>    <type>  <options>                          <dump> <pass>
UUID=$UUID_ROOTFS   /        ext4    defaults,noatime,errors=remount-ro 0      1
UUID=$UUID_BOOTFS   /boot    ext4    defaults                           0      2
EOF
```

移动 boot 到其他目录，以便分别制作 bootfs 和 rootfs 分区，

```shell
mkdir -p bootfs
mv $TARGET_ROOTFS/boot/* bootfs
```

生成 bootfs.ext4 和 rootfs.ext4，

```shell
mke2fs -d bootfs -L bootfs -t ext4 -U $UUID_BOOTFS bootfs.ext4 "256M"
mke2fs -d $TARGET_ROOTFS -L rootfs -t ext4 -N 524288 -U $UUID_ROOTFS rootfs.ext4 "2048M"
```

此时，在当前目录可以看到两个分区镜像，bootfs.ext4 和 rootfs.ext4，可使用 fastboot 烧写到板子中。

## 制作固件

见 [固件制作指南](./image.md)。
