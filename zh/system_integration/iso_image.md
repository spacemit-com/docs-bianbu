---
sidebar_position: 10
---

# ISO 镜像制作指南

基于官方 UEFI 镜像（`.tar.gz`）制作可启动的 Live ISO：进入 Live 桌面（"Try Bianbu"）或通过 Calamares 安装器安装到板卡（"Install Bianbu"）。目前仅 `lxqt-uefi` 变体可以产出有意义的 ISO。

> **注意**：本指南所述流程**仍在开发中，暂不成熟，仅供参考**。

## 环境要求

- root 权限（需要挂载 `rootfs.ext4`/`bootfs.ext4` 并在其中 chroot 执行），容器内执行需 `--privileged`；
- **推荐在 K3 板卡或 riscv64 环境执行**；x86_64 宿主机执行需先按 [Bianbu 4.0 ROOTFS 制作](bianbu_4.0_rootfs_create.md) 的环境要求配置 `qemu-user`（10.2+）binfmt；
- 构建环境安装依赖：

  ```shell
  sudo apt-get -y install wget squashfs-tools xorriso mtools dosfstools apt-utils
  ```

- 磁盘空间：建议预留 20G 以上（含镜像、解压目录与 squashfs 中间产物）；
- 输入：UEFI 变体的官方镜像 `.tar.gz`（包含 `rootfs.ext4` / `bootfs.ext4`）。

## 制作步骤

### 1. 解包并挂载

```shell
export TARBALL=Bianbu-LXQt-UEFI-K3-v4.0.1-20260529101530.tar.gz
export WORKDIR=k3-iso-work
mkdir -p $WORKDIR && tar xzf $TARBALL -C $WORKDIR
export R=$WORKDIR/rootfs   # $R：rootfs.ext4 的挂载点，后续所有步骤中的 $R 均指此目录
mkdir -p $R
mount $WORKDIR/rootfs.ext4 $R
mount $WORKDIR/bootfs.ext4 $R/boot
mount -t proc /proc $R/proc
mount -t sysfs /sys $R/sys
mount -o bind /dev $R/dev
mount -o bind /dev/pts $R/dev/pts
```

### 2. 安装 Live 组件（casper + Calamares）

```shell
chroot $R bash -c "
  apt-get remove -y -qq calamares-settings-bianbu 2>/dev/null || true
  apt-get update -qq
  apt-get install -y -qq casper calamares-settings-lubuntu
  grep -qx isofs /etc/initramfs-tools/modules || echo isofs >> /etc/initramfs-tools/modules
"
```

### 3. 适配 Calamares（riscv64/bianbu）

Calamares 配置随上游版本变化，以下为关键适配点（具体 sed 规则以实际版本为准，也可参考官方 ISO 中的安装器配置）：

- `calamares/bootloader/main.py`：增加 `riscv64-efi` bitness 分支（`return "riscv64-efi", "grubriscv64.efi", "bootriscv64.efi"`）；
- `bootloader.conf`：`efiBootloaderId: "bianbu"`；
- `settings.conf`：启用 `unpackfs`、`initramfs`、`shellprocess`（清理 lubuntu-calamares 快捷方式）等模块，移除 pkgselect/automirror 等不适用模块；`branding` 指向 bianbu；
- `users.conf`：允许简单密码（minLength 0、allowWeakPasswords true）；
- `partition.conf`：分区名 `bianbu_boot`，名称 `Bianbu <版本>`；
- `displaymanager.conf`：`executable: startlxqtwayland`，`desktopFile: lxqt-wayland`；
- `unpackfs.conf`：解包源为 `/cdrom/casper/filesystem.squashfs`；
- `branding.desc` 与桌面入口：将版本/名称替换为 Bianbu（并考虑 `XDG_RUNTIME_DIR` 等 Wayland 兼容调整）；
- 增加本地源 `cdrom.sources`（`CODENAME` 变量后续步骤还会用到）：

  ```shell
  export CODENAME=$(grep -m1 '^VERSION_CODENAME=' $R/etc/os-release | cut -d= -f2)
  cat > $R/etc/apt/sources.list.d/cdrom.sources <<EOF
  Types: deb
  URIs: file:/cdrom
  Suites: $CODENAME
  Components: main
  Trusted: yes
  EOF
  ```

- 可选：安装 kpmcore 补丁 deb（如 `libkpmcore13_*k3fix*.deb`）到 rootfs，否则安装器手动分区可能存在闪退风险。

### 4. 配置 Live GRUB 启动项（含 devicetree）

写入 `$R/etc/grub.d/09_bianbu_uefi`（内容同 [Bianbu 4.0 UEFI 镜像制作](bianbu_4.0_uefi_image_create.md) 第 2 步），并：

```shell
chmod 0755 $R/etc/grub.d/09_bianbu_uefi
chmod 0644 $R/etc/grub.d/10_linux
chroot $R update-initramfs -u
chroot $R apt-get clean
```

### 5. 生成 ESP 引导镜像（BOOTRISCV64.EFI）

```shell
export KVER=$(basename "$(ls $R/boot/vmlinuz-* | sort -V | tail -1)")
export KVER=${KVER#vmlinuz-}    # 完整版本号，含 -generic 等后缀
export I=iso-tree
rm -rf $I && mkdir -p $I/casper $I/boot/grub $I/EFI/boot $I/.disk \
    $I/dists/$CODENAME/main/binary-riscv64 $I/pool/main
touch $I/bianbu
cp $R/boot/vmlinuz-$KVER $I/casper/vmlinuz
cp $R/boot/initrd.img-$KVER $I/casper/initrd
cp -r $R/boot/spacemit $I/casper/spacemit
cp $R/boot/config-$KVER $R/boot/System.map-$KVER $I/casper/ 2>/dev/null || true

# ISO GRUB 菜单（Try / Install）
cat > $I/boot/grub/grub.cfg <<EOF
efienv update product_name
efienv -g spacemit update product_name
search --set=root --file /bianbu
insmod all_video
set default="0"
set timeout=30
menuentry "Try Bianbu FS without installing" {
   linux /casper/vmlinuz boot=casper nopersistent toram plymouth.prefer-fbcon plymouth.ignore-serial-consoles quiet splash ---
   initrd /casper/initrd
   devicetree /casper/spacemit/${KVER}/\${product_name}.dtb
}
menuentry "Install Bianbu FS" {
   linux /casper/vmlinuz boot=casper bianbu.installer nopersistent toram plymouth.prefer-fbcon plymouth.ignore-serial-consoles quiet splash ---
   initrd /casper/initrd
   devicetree /casper/spacemit/${KVER}/\${product_name}.dtb
}
EOF

# grub-mkstandalone 生成 BOOTRISCV64.EFI，装入 64M FAT 的 efi.img
# 注意：grub.cfg 需先拷入 rootfs（chroot 内无法访问宿主机路径）
cp $I/boot/grub/grub.cfg $R/tmp/iso-grub.cfg
chroot $R grub-mkstandalone --format=riscv64-efi --output=/tmp/BOOTRISCV64.EFI \
    --install-modules="linux normal iso9660 memdisk search tar ls all_video fdt efivariable gzio xzio zstd" \
    --modules="linux normal iso9660 search efivariable efifwsetup fdt gzio" \
    --locales="" --fonts="" "boot/grub/grub.cfg=/tmp/iso-grub.cfg"
dd if=/dev/zero of=$I/EFI/boot/efi.img bs=1M count=64 status=none
mkfs.vfat -F 32 $I/EFI/boot/efi.img >/dev/null
LC_CTYPE=C mmd -i $I/EFI/boot/efi.img EFI EFI/BOOT
LC_CTYPE=C mcopy -i $I/EFI/boot/efi.img $R/tmp/BOOTRISCV64.EFI ::EFI/BOOT/BOOTRISCV64.EFI
rm -f $R/tmp/BOOTRISCV64.EFI $R/tmp/iso-grub.cfg
```

注意：`grub-mkstandalone` 与 `mksquashfs` 需串行执行，且在 `mksquashfs` 前必须卸载 `$R/proc`、`$R/sys`、`$R/dev`（否则遍历 rootfs 会卡死）。

### 6. 生成 Live 文件系统（squashfs）

```shell
# 可选：下载语言包到 apt 缓存，随 ISO pool 提供离线安装
chroot $R bash -c 'DEBIAN_FRONTEND=noninteractive apt-get install --download-only --reinstall -y -qq \
    language-pack-zh-hans language-pack-gnome-zh-hans \
    language-pack-en language-pack-gnome-en 2>/dev/null || true'
for deb in "$R"/var/cache/apt/archives/*.deb; do
    [ -f "$deb" ] || continue
    name=$(basename "$deb"); src=${name%%_*}
    letter=${src:0:1}; case "$src" in lib*) letter=${src:0:4};; esac
    mkdir -p "$I/pool/main/$letter/$src"; cp "$deb" "$I/pool/main/$letter/$src/"
done

# 卸载系统资源（mksquashfs 遍历 rootfs 前必须卸载，否则会卡死）
umount $R/proc $R/sys $R/dev/pts $R/dev 2>/dev/null || true

mksquashfs $R $I/casper/filesystem.squashfs \
  -noappend -no-duplicates -no-recovery -wildcards \
  -comp zstd -Xcompression-level 19 -b 1M \
  -processors "$(nproc)" \
  -e "var/cache/apt/archives/*" -e "root/*" -e "root/.*" \
  -e "tmp/*" -e "tmp/.*" -e "swapfile" -e "boot/lost+found" -e "lost+found"
du -sx --block-size=1 "$I/casper/filesystem.squashfs" | cut -f1 > "$I/casper/filesystem.size"
```

### 7. 打包 ISO

```shell
( cd $I
  apt-ftparchive packages pool/main > "dists/$CODENAME/main/binary-riscv64/Packages"
  find . -type f -print0 | xargs -0 md5sum | grep -v md5sum.txt > md5sum.txt
  xorriso -as mkisofs -iso-level 3 -full-iso9660-filenames -J -joliet-long \
    -no-emul-boot -partition_cyl_align off -partition_offset 16 \
    -volid "BIANBU_26.04_K3_ISO" -output ../Bianbu-LXQT-UEFI-K3-$(date +%Y%m%d%H%M%S).iso \
    -e EFI/boot/efi.img \
    -append_partition 2 28732ac11ff8d211ba4b00a0c93ec93b EFI/boot/efi.img \
    -appended_part_as_gpt -isohybrid-gpt-basdat -graft-points . )

umount $R/boot $R
```

## 使用

1. 将 ISO 烧录/挂载，通过 UEFI 引导启动（板卡需已支持 UEFI 固件，即官方 4.0 UEFI 镜像可启动的板卡）；
2. 在 GRUB 中选择 **Try Bianbu FS without installing**（Live 桌面）或 **Install Bianbu FS**（Calamares 安装器）；
3. 安装过程可离线完成（ISO 内置本地源 `cdrom.sources` 与基础语言包）。

## 常见问题

- **Live 没有桌面 / 安装器未启动**：确认 Calamares `branding`、`displaymanager` 配置已按第 3 步适配，且 initramfs 中包含 `isofs` 模块（第 2 步）。
- **squashfs 卡死**：执行 `mksquashfs` 前必须确认 `$R/proc`、`$R/sys`、`$R/dev` 已卸载。
