---
sidebar_position: 9
---

# Bianbu 4.0 UEFI 镜像制作

Bianbu 4.0 提供基于 EDK2 的 UEFI 启动方式：UEFI 固件（`edk2.itb`）替代传统 U-Boot 引导，bootfs/rootfs 布局不变，系统由 GRUB（位于 ESP 分区）加载内核与按板卡选择的设备树。在 [Bianbu 4.0 ROOTFS 制作](bianbu_4.0_rootfs_create.md) 的基础上，制作 UEFI 镜像主要有三处差异：

1. rootfs 中安装 UEFI 相关软件包（GRUB + `edk2-spacemit`），并配置带 devicetree 的 GRUB 启动项；
2. 增加 ESP 分区镜像（`esp.vfat`），装入 GRUB；
3. 分区表中 `uboot` 分区映射为 `edk2.itb`，并新增 `ESP` 分区。

> Bianbu 2.x（K1）的 UEFI 固件与镜像制作（手工编译 EDK2）请参考 [UEFI固件与系统镜像制作指南](uefi_image.md)。

## UEFI 启动链路

```
FSBL → esos → OpenSBI → edk2.itb (UEFI 固件) → BOOTRISCV64.EFI (GRUB, ESP 分区)
     → 内核 + 设备树 (/boot/spacemit/<内核版本>/<product_name>.dtb)
```

- 每个内核版本对应 `/boot/spacemit/<版本>/` 目录，包含各板卡型号的 DTB；
- GRUB 通过 `efienv -g spacemit update product_name` 读取板卡名（product_name），动态选择对应 DTB；
- 内核与 initramfs 由启动项指定，DTB 是唯一按板卡变化的部分。

## 一、在 ROOTFS 中配置 UEFI 组件

按 [Bianbu 4.0 ROOTFS 制作](bianbu_4.0_rootfs_create.md) 完成基础 rootfs 后（第 1~4 步，含 `bianbu-minimal`），继续执行：

### 1. 安装 UEFI 软件包

```shell
chroot rootfs /bin/bash -c "apt-get update"
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive UCF_FORCE_CONFFNEW=1 apt-get -y --allow-downgrades install \
    grub-common grub2-common grub-efi-riscv64-bin grub-efi-riscv64-unsigned grub-efi-riscv64 \
    binutils edk2-spacemit"
```

`edk2-spacemit` 提供 UEFI 固件 `edk2.itb`（安装到 `usr/lib/uefi/spacemit/edk2.itb`），GRUB 包提供 `grubriscv64.efi`（`usr/lib/grub/riscv64-efi/monolithic/`）。

### 2. 配置 GRUB 启动项

写入 `etc/default/grub.d/99-bianbu-uefi.cfg`：

```shell
cat > rootfs/etc/default/grub.d/99-bianbu-uefi.cfg <<EOF
GRUB_TIMEOUT=5
GRUB_TIMEOUT_STYLE=menu
GRUB_DISABLE_OS_PROBER=true
GRUB_DISABLE_RECOVERY=true
GRUB_DEFAULT=0
EOF
```

写入 `etc/grub.d/09_bianbu_uefi`，为其生成包含 devicetree 的启动项（每个内核版本都选择配套的板卡 DTB，旧内核条目兼作救援入口）：

```shell
cat > rootfs/etc/grub.d/09_bianbu_uefi <<'EOF'
#!/bin/sh
set -e

# Bianbu UEFI boot entries with per-board devicetree selection (efienv product_name).
# Generates entries ahead of 10_linux (which is disabled) so the system
# loads the correct board DTB from /boot/spacemit/<kver>/.

DIST_LABEL='@DIST_LABEL@'      # Bianbu / Ubuntu
VARIANT_LABEL='@VARIANT_LABEL@' # e.g. LXQt / Minimal
BOARD='@BOARD@'                # k3 / k1

# 判断是否运行在 chroot 中：构建期 chroot 无真实块设备，grub-probe 会误读宿主，
# 故写死 LABEL（打包阶段替换为最终 UUID）；真机 update-grub 用真实 UUID。
running_in_chroot() {
    if [ "${SYSTEMD_IGNORE_CHROOT:-0}" = "1" ]; then return 1; fi
    if [ -e "/proc/1/root" ]; then
        root_dev_ino=$(stat -c '%d:%i' / 2>/dev/null) || return 0
        proc1_root_dev_ino=$(stat -L -c '%d:%i' /proc/1/root 2>/dev/null) || return 0
        [ "$root_dev_ino" = "$proc1_root_dev_ino" ] && return 1 || return 0
    fi
    [ "$$" = "1" ] && return 1 || return 0
}

if running_in_chroot; then
    ROOT_PARAM="LABEL=rootfs"
    ROOT_SEARCH="--label bootfs --set=root"
else
    GRUB_PROBE=$(command -v grub-probe 2>/dev/null) || GRUB_PROBE=""
    rootuuid=""; bootuuid=""
    if [ -n "$GRUB_PROBE" ]; then
        rootuuid=$("$GRUB_PROBE" --target=fs_uuid / 2>/dev/null) || rootuuid=""
        bootuuid=$("$GRUB_PROBE" --target=fs_uuid /boot 2>/dev/null) || bootuuid=""
    fi
    if [ -n "$rootuuid" ] && [ -n "$bootuuid" ]; then
        ROOT_PARAM="UUID=$rootuuid"
        ROOT_SEARCH="--fs-uuid $bootuuid --set=root"
    else
        ROOT_PARAM="LABEL=rootfs"
        ROOT_SEARCH="--label bootfs --set=root"
    fi
fi

first=1
for kernel in $(ls /boot/vmlinuz-* 2>/dev/null | sort -rV); do
    version="${kernel#/boot/vmlinuz-}"
    dtbdir="/boot/spacemit/$version"
    initrd="/boot/initrd.img-$version"
    [ -d "$dtbdir" ] || continue
    ls "$dtbdir"/*.dtb >/dev/null 2>&1 || continue
    [ -f "$initrd" ] || continue

    if [ "$first" = 1 ]; then
        title="$DIST_LABEL $VARIANT_LABEL UEFI"
        first=0
    else
        title="$DIST_LABEL $VARIANT_LABEL UEFI (Linux $version)"
    fi

    cat <<GRUBEOF
menuentry '$title' --class gnu-linux --class gnu --class os 'bianbu-uefi-$BOARD-$version' {
  insmod part_gpt
  insmod ext2
  search --no-floppy $ROOT_SEARCH
  echo 'Loading Linux $version ...'
  linux /vmlinuz-$version root=$ROOT_PARAM rootwait rw plymouth.prefer-fbcon plymouth.ignore-serial-consoles splash
  echo 'Loading initramfs ...'
  initrd /initrd.img-$version
  efienv -g spacemit update product_name
  echo 'Loading device tree blob ...'
  devicetree /spacemit/$version/\${product_name}.dtb
}
GRUBEOF
done

if [ "$first" = 1 ]; then
    echo "09_bianbu_uefi: no kernel with dtb/initrd found" >&2
    exit 1
fi
EOF

# 替换标签
sed -i \
    -e "s|@DIST_LABEL@|Bianbu|g" \
    -e "s|@VARIANT_LABEL@|LXQt|g" \
    -e "s|@BOARD@|k3|g" \
    rootfs/etc/grub.d/09_bianbu_uefi

# 启用 09_bianbu_uefi，禁用 10_linux（其条目不含 devicetree，UEFI 下无法引导）
chmod 0755 rootfs/etc/grub.d/09_bianbu_uefi
chmod 0644 rootfs/etc/grub.d/10_linux

# 生成 grub.cfg（构建期输出 LABEL 占位符，打包阶段替换为真实 UUID）
chroot rootfs /bin/bash -c "update-grub"
```

## 二、打包 UEFI 镜像

在 [Bianbu 4.0 ROOTFS 制作](bianbu_4.0_rootfs_create.md) 的「二、生成分区镜像」与 [固件制作指南](image.md) 的打包流程基础上，增加/替换以下步骤。以下命令沿用「二」第 1 步导出的 `$UUID_BOOTFS` / `$UUID_ROOTFS` 环境变量（如更换过 shell，请重新执行该步的 `export`）：

### 1. 固件文件与 ESP 分区镜像

```shell
export TMP=pack_dir

# 用 edk2.itb 替代 u-boot.itb（拷贝到打包目录，供分区表引用）
cp rootfs/usr/lib/uefi/spacemit/edk2.itb $TMP

# 创建 ESP 分区镜像（256M）并装入 GRUB
dd if=/dev/zero of=$TMP/esp.vfat bs=1M count=256
mkfs.vfat -S 4096 -F 16 -n "ESP" $TMP/esp.vfat
export UUID_ESP=$(blkid -s UUID -o value $TMP/esp.vfat)

ESP_MOUNT=$(mktemp -d)
mount -o loop $TMP/esp.vfat $ESP_MOUNT
mkdir -p $ESP_MOUNT/EFI/BOOT $ESP_MOUNT/EFI/bianbu
cp rootfs/usr/lib/grub/riscv64-efi/monolithic/grubriscv64.efi $ESP_MOUNT/EFI/BOOT/BOOTRISCV64.EFI
cat > $ESP_MOUNT/EFI/BOOT/grub.cfg <<EOF
search --no-floppy --fs-uuid --set=root $UUID_BOOTFS
set prefix=(\$root)/grub
configfile \$prefix/grub.cfg
EOF
sync
umount $ESP_MOUNT
```

### 2. 更新 /etc/fstab（增加 ESP）

```shell
cat > rootfs/etc/fstab <<EOF
# <file system>     <dir>       <type>  <options>                          <dump> <pass>
UUID=$UUID_ROOTFS   /           ext4    defaults,noatime,errors=remount-ro 0      1
UUID=$UUID_BOOTFS   /boot       ext4    defaults                           0      2
UUID=$UUID_ESP      /boot/efi   vfat    umask=0077                         0      1
EOF
```

### 3. 替换 bootfs 中 grub.cfg 的 LABEL 为 UUID

```shell
sed -i "s/root=LABEL=rootfs/root=UUID=$UUID_ROOTFS/g" bootfs/grub/grub.cfg
sed -i "s/--label bootfs/--fs-uuid $UUID_BOOTFS/g" bootfs/grub/grub.cfg
```

### 4. UEFI 分区表

将 `partition_universal.json`、`partition_flash.json`、`partition_4M.json` 替换为以下 UEFI 版本（`uboot` 分区使用 `edk2.itb`，并新增 `ESP` 分区；文件系统分区相应后移）。`partition_4M.json` 适用于 ≥4M 的 SPI NOR——板卡实际 NOR 一般为 8M，刷机工具会按容量向下匹配分区表：

```json
/* partition_universal.json — SD 卡 / GPT 通用布局 */
{
    "version": "1.0",
    "format": "gpt",
    "partitions": [
        { "name": "env",      "hidden": true, "offset": "640K",  "size": "64K",  "image": "env.bin" },
        { "name": "bootinfo", "hidden": true, "offset": "1M",    "size": "128K", "image": "factory/bootinfo_block.bin" },
        { "name": "fsbl",     "hidden": true, "offset": "1536K", "size": "512K", "image": "factory/FSBL.bin" },
        { "name": "esos",     "hidden": true, "offset": "4M",    "size": "3M",   "image": "esos.itb" },
        { "name": "opensbi",  "hidden": true, "offset": "7M",    "size": "1M",   "image": "fw_dynamic.itb" },
        { "name": "uboot",    "hidden": true, "offset": "8M",    "size": "4M",   "image": "edk2.itb" },
        { "name": "ESP",      "offset": "12M", "size": "256M", "image": "esp.vfat", "compress": "gzip-5" },
        { "name": "bootfs",   "offset": "268M", "size": "256M", "image": "bootfs.ext4", "compress": "gzip-5" },
        { "name": "rootfs",   "offset": "524M", "size": "-",   "image": "rootfs.ext4", "compress": "gzip-5" }
    ]
}
```

```json
/* partition_4M.json — SPI NOR (mtd) 布局 */
{
    "version": "1.0",
    "format": "mtd",
    "partitions": [
        { "name": "bootinfo", "offset": "0",     "size": "128K", "image": "factory/bootinfo_spinor.bin" },
        { "name": "fsbl",     "offset": "128K",  "size": "512K", "image": "factory/FSBL.bin" },
        { "name": "env",      "offset": "640K",  "size": "64K",  "image": "env.bin" },
        { "name": "esos",     "offset": "704K",  "size": "1M",   "image": "esos.itb" },
        { "name": "opensbi",  "offset": "1728K", "size": "384K", "image": "fw_dynamic.itb" },
        { "name": "uboot",    "offset": "2112K", "size": "3M",   "image": "edk2.itb" }
    ]
}
```

### 5. 打包

同 [固件制作指南](image.md) 中 4.0（K3）的 tar.gz 方式打包（文件名用 UEFI 变体命名）：生成 `md5sums.txt` 后 `tar -cf - . | pigz > Bianbu-LXQt-UEFI-K3-<版本>-<时间戳>.tar.gz`。

> 注意：UEFI 镜像官方只发布 `.tar.gz`（Titan 固件包），不发布 SD 卡 `.img.gz`。如需 SD 卡介质，可直接用上面的 UEFI 版 `partition_universal.json` 按 [固件制作指南](image.md) 的「SDCARD 镜像」流程自行生成。

## 三、刷写与启动

1. 解压 `.tar.gz`，使用 Titan 工具选择 `partition_flash.json` 刷写；
2. 上电后按 UEFI 链路启动：UEFI 固件加载 ESP 中的 GRUB，GRUB 按板卡选择 DTB，加载内核进入系统。

## 常见问题

- **内核升级后启动条目未更新**：`/etc/grub.d/10_linux` 必须保持禁用状态（由 `09_bianbu_uefi` 生成条目）；升级内核后执行 `sudo update-grub` 即可。
- **找不到板卡 DTB**：确认内核包安装了 `/boot/spacemit/<版本>/` 目录；启动项中的 `efienv -g spacemit update product_name` 会自动写入板卡名。
- **想要可安装的 Live 介质**：使用 [ISO 镜像制作指南](iso_image.md)，ISO 基于 UEFI 镜像制作。
