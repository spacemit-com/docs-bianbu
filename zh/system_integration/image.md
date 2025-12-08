---
sidebar_position: 5
---

# 固件制作指南

## Titan固件

1. 安装依赖

   ```shell
   apt-get -y install zip
   ```

2. 拷贝固件依赖的文件

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
   cp bootfs.ext4 $TMP
   cp rootfs.ext4 $TMP
   ```

3. 下载参考分区表

   ```shell
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/fastboot.yaml
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_2M.json
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_flash.json
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
   ```

4. 打包

   ```shell
   cd $TMP
   zip -r ../bianbu-custom.zip *
   ```

## SDCARD镜像

下面介绍如何使用genimage制作SDCARD镜像。

1. 安装依赖

   ```shell
   echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections
   echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections
   DEBIAN_FRONTEND=noninteractive apt-get -y install wget python3 genimage
   ```

1. 拷贝固件依赖的文件

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
   cp bootfs.ext4 $TMP
   cp rootfs.ext4 $TMP
   ```

2. 下载参考分区表

   ```shell
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
   ```

3. 下载生成genimage.cfg的脚本，并生成genimage.cfg

   ```shell
   wget -P $TMP https://gitee.com/spacemit-buildroot/scripts/raw/bl-v1.0.y/gen_imgcfg.py
   python3 $TMP/gen_imgcfg.py -i $TMP/partition_universal.json -n bianbu-custom.sdcard -o $TMP/genimage.cfg
   ```
   
4. 生成SDCARD镜像

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
