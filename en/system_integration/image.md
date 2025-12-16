---
sidebar_position: 5
---

# Image Creation Guide

## Titan Image

1. Install dependencies

   ```shell
   apt-get -y install zip
   ```

2. Copy required files for the image

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

3. Download reference partition tables

   ```shell
   wget -P $TMP https://gitee.com/bianbu/firmware-config/raw/main/fastboot.yaml
   wget -P $TMP https://gitee.com/bianbu/firmware-config/raw/main/partition_2M.json
   wget -P $TMP https://gitee.com/bianbu/firmware-config/raw/main/partition_flash.json
   wget -P $TMP https://gitee.com/bianbu/firmware-config/raw/main/partition_universal.json
   ```

4. Package the image

   ```shell
   cd $TMP
   zip -r ../bianbu-custom.zip *
   ```

## SDCARD Image

Here’s how to create an SDCARD image using `genimage`.

1. Install dependencies

   ```shell
   echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections
   echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections
   DEBIAN_FRONTEND=noninteractive apt-get -y install wget python3 genimage
   ```

2. Copy required files for the image

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

3. Download reference partition table

   ```shell
   wget -P $TMP https://gitee.com/bianbu/firmware-config/raw/main/partition_universal.json
   ```

4. Download script to generate `genimage.cfg` and create `genimage.cfg`

   ```shell
   wget -P $TMP https://gitee.com/spacemit-buildroot/scripts/raw/bl-v1.0.y/gen_imgcfg.py
   python3 $TMP/gen_imgcfg.py -i $TMP/partition_universal.json -n bianbu-custom.sdcard -o $TMP/genimage.cfg
   ```

5. Generate SDCARD image

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

   If you see the following messages, the packaging was successful:

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
