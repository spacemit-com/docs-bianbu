---
sidebar_position: 6
---

# 固件制作指南

本文介绍如何将制作好的 `bootfs.ext4` / `rootfs.ext4` 分区镜像连同启动固件打包成可刷写的固件，适用于 **Bianbu 2.x（K1）** 与 **Bianbu 4.0（K3）**，产出两种形态：

- **Titan 固件包**：配合 Titan 刷机工具使用；
- **SD 卡镜像**：可直接写入 SD 卡（genimage 制作）。

两种形态的前置步骤相同，版本差异（固件文件、分区表分支、打包格式）在各步骤中分别标注。ROOTFS 与分区镜像的制作见各版本的 ROOTFS 制作文档（如 [Bianbu 3.0 ROOTFS 制作](bianbu_3.0_rootfs_create.md)、[Bianbu 4.0 ROOTFS 制作](bianbu_4.0_rootfs_create.md)）。

## 准备打包目录

1. 安装依赖

   ```shell
   apt-get -y install zip wget python3 genimage pigz
   ```

2. 拷贝固件依赖的文件

   以 ROOTFS 制作文档中导出的 `$TARGET_ROOTFS`（rootfs 目录）及其同目录下的 `bootfs.ext4`、`rootfs.ext4` 为准。

   - 2.x（K1）：

     ```shell
     export TMP=pack_dir
     mkdir -p $TMP/factory/
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/bootinfo_emmc.bin $TMP/factory
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/bootinfo_sd.bin $TMP/factory
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/bootinfo_spinand.bin $TMP/factory
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/bootinfo_spinor.bin $TMP/factory
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/FSBL.bin $TMP/factory
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/u-boot.itb $TMP
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/env.bin $TMP
     cp $TARGET_ROOTFS/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb $TMP
     cp bootfs.ext4 rootfs.ext4 $TMP
     ```

   - 4.0（K3）：bootinfo 改为通配拷贝（4.0 的 deb 提供 `bootinfo_block/spinand/spinor.bin`，不再提供 `bootinfo_emmc/sd.bin`），并额外拷贝 `esos.itb` 与 K3 板卡的 `ec.bin`：

     ```shell
     export TMP=pack_dir
     mkdir -p $TMP/factory/
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/bootinfo_*.bin $TMP/factory/
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/FSBL.bin $TMP/factory/
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/u-boot.itb $TMP
     cp $TARGET_ROOTFS/usr/lib/u-boot/spacemit/env.bin $TMP
     cp $TARGET_ROOTFS/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb $TMP
     cp $TARGET_ROOTFS/usr/lib/riscv64-linux-gnu/esos/esos.itb $TMP
     cp $TARGET_ROOTFS/usr/lib/firmware/k3-pico-itx/ec.bin $TMP
     cp bootfs.ext4 rootfs.ext4 $TMP
     ```

3. 下载参考分区表与 fastboot 配置（按版本选择分支）

   - 2.x（K1，`main` 分支）：

     ```shell
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/fastboot.yaml
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_2M.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_flash.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
     ```

   - 4.0（K3，`k3-br-v1.0.y` 分支；分区表包含 `esos` 分区，`bootinfo` 引用 `bootinfo_block.bin`）：

     ```shell
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/fastboot.yaml
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/partition_universal.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/partition_flash.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/partition_4M.json
     ```

     `partition_4M.json` 适用于 ≥4M 的 SPI NOR（板卡实际 NOR 一般为 8M，刷机工具会按容量向下匹配分区表）。

## Titan 固件包

- 2.x（K1）：打包为 zip

  ```shell
  cd $TMP
  zip -r ../bianbu-custom.zip *
  ```

- 4.0（K3）：打包为 tar.gz（含 `md5sums.txt`，Titan 工具直接使用包内的 `fastboot.yaml` / `partition_flash.json` 刷写）

  ```shell
  cd $TMP
  find . -type f ! -name md5sums.txt -exec md5sum {} + | sort -k2 > md5sums.txt
  tar -cf - . | pigz > ../Bianbu-LXQt-K3-$(date +%Y%m%d%H%M%S).tar.gz
  ```

## SDCARD 镜像

下面介绍如何使用 genimage 制作 SDCARD 镜像，打包目录复用上文「准备打包目录」的结果。

1. 安装依赖

   ```shell
   echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections
   echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections
   DEBIAN_FRONTEND=noninteractive apt-get -y install wget python3 genimage
   ```

2. 下载生成 genimage.cfg 的脚本（按版本选择分支），并生成 genimage.cfg

   - 2.x（K1，`bl-v1.0.y` 分支）：

     ```shell
     wget -P $TMP https://gitee.com/spacemit-buildroot/scripts/raw/bl-v1.0.y/gen_imgcfg.py
     python3 $TMP/gen_imgcfg.py -i $TMP/partition_universal.json -n bianbu-custom.sdcard -o $TMP/genimage.cfg
     ```

   - 4.0（K3，`k3-br-v1.0.y` 分支）：

     ```shell
     wget -P $TMP https://gitee.com/spacemit-buildroot/scripts/raw/k3-br-v1.0.y/gen_imgcfg.py
     python3 $TMP/gen_imgcfg.py -i $TMP/partition_universal.json -n sdcard.img -o $TMP/genimage.cfg
     ```

3. 生成 SDCARD 镜像

   ```shell
   ROOTPATH_TMP="$(mktemp -d)"
   GENIMAGE_TMP="$(mktemp -d)"
   genimage \
       --config "$TMP/genimage.cfg" \
       --rootpath "$ROOTPATH_TMP" \
       --tmppath "$GENIMAGE_TMP" \
       --inputpath "$TMP" \
       --outputpath "."
   ```

   当看到以下信息时，说明打包成功。

   ```shell
   INFO: hdimage(bianbu-custom): adding partition 'bootinfo' from 'factory/bootinfo_sd.bin' ...
   INFO: hdimage(bianbu-custom): adding partition 'fsbl' (in MBR) from 'factory/FSBL.bin' ...
   INFO: hdimage(bianbu-custom): adding partition 'env' (in MBR) from 'env.bin' ...
   INFO: hdimage(bianbu-custom): adding partition 'opensbi' (in MBR) from 'fw_dynamic.itb' ...
   INFO: hdimage(bianbu-custom): adding partition 'uboot' (in MBR) from 'u-boot.itb' ...
   INFO: hdimage(bianbu-custom): adding partition 'bootfs' (in MBR) from 'bootfs.ext4' ...
   INFO: hdimage(bianbu-custom): adding partition 'rootfs' (in MBR) from 'rootfs.ext4' ...
   INFO: hdimage(bianbu-custom): adding partition '[MBR]' ...
   INFO: hdimage(bianbu-custom): adding partition '[GPT header]' ...
   INFO: hdimage(bianbu-custom): adding partition '[GPT array]' ...
   INFO: hdimage(bianbu-custom): adding partition '[GPT backup]' ...
   INFO: hdimage(bianbu-custom): writing GPT
   INFO: hdimage(bianbu-custom): writing protective MBR
   INFO: hdimage(bianbu-custom): writing MBR
   INFO: cmd: "rm -rf "/tmp/tmp.rX4fZ39DKG"/*" (stderr):
   ```
