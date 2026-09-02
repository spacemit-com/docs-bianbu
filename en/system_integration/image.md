---
sidebar_position: 6
---

# Image Creation Guide

This guide describes how to package prepared `bootfs.ext4` / `rootfs.ext4` partition images together with the boot firmware into flashable artifacts, for both **Bianbu 2.x (K1)** and **Bianbu 4.0 (K3)**, in two forms:

- **Titan firmware package**: used with the Titan flashing tool;
- **SD card image**: written directly to an SD card (created with genimage).

Both share the same preparation steps; the version differences (firmware files, partition table branch, package format) are marked in each step. The creation of the ROOTFS and the partition images is covered by the per-version ROOTFS guides (e.g. [Bianbu 3.0 ROOTFS Creation](bianbu_3.0_rootfs_create.md), [Bianbu 4.0 ROOTFS Creation](bianbu_4.0_rootfs_create.md)).

## Prepare the Packaging Directory

1. Install the dependencies

   ```shell
   apt-get -y install zip wget python3 genimage pigz
   ```

2. Copy the files required by the firmware

   Based on the `$TARGET_ROOTFS` (the rootfs directory) exported in the ROOTFS creation guide and the `bootfs.ext4`, `rootfs.ext4` next to it.

   - 2.x (K1):

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

   - 4.0 (K3): the bootinfo files are copied with a glob (the 4.0 debs provide `bootinfo_block/spinand/spinor.bin`, no longer `bootinfo_emmc/sd.bin`), and `esos.itb` plus the `ec.bin` of K3 boards are added:

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

3. Download the reference partition tables and the fastboot config (choose the branch per version)

   - 2.x (K1, `main` branch):

     ```shell
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/fastboot.yaml
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_2M.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_flash.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
     ```

   - 4.0 (K3, `k3-br-v1.0.y` branch; the tables include an `esos` partition, and `bootinfo` references `bootinfo_block.bin`):

     ```shell
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/fastboot.yaml
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/partition_universal.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/partition_flash.json
     wget -P $TMP https://gitee.com/bianbu/image-config/raw/k3-br-v1.0.y/partition_4M.json
     ```

     `partition_4M.json` targets SPI NOR flashes of 4M and above (the actual NOR on the boards is usually 8M — the flashing tool matches the partition table downward by capacity).

## Titan Firmware Package

- 2.x (K1): packaged as zip

  ```shell
  cd $TMP
  zip -r ../bianbu-custom.zip *
  ```

- 4.0 (K3): packaged as tar.gz (with `md5sums.txt`; the Titan tool flashes using the `fastboot.yaml` / `partition_flash.json` inside the package)

  ```shell
  cd $TMP
  find . -type f ! -name md5sums.txt -exec md5sum {} + | sort -k2 > md5sums.txt
  tar -cf - . | pigz > ../Bianbu-LXQt-K3-$(date +%Y%m%d%H%M%S).tar.gz
  ```

## SDCARD Image

The following describes how to create an SDCARD image with genimage, reusing the packaging directory prepared above.

1. Install the dependencies

   ```shell
   echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections
   echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections
   DEBIAN_FRONTEND=noninteractive apt-get -y install wget python3 genimage
   ```

2. Download the script that generates `genimage.cfg` (choose the branch per version) and create `genimage.cfg`

   - 2.x (K1, `bl-v1.0.y` branch):

     ```shell
     wget -P $TMP https://gitee.com/spacemit-buildroot/scripts/raw/bl-v1.0.y/gen_imgcfg.py
     python3 $TMP/gen_imgcfg.py -i $TMP/partition_universal.json -n bianbu-custom.sdcard -o $TMP/genimage.cfg
     ```

   - 4.0 (K3, `k3-br-v1.0.y` branch):

     ```shell
     wget -P $TMP https://gitee.com/spacemit-buildroot/scripts/raw/k3-br-v1.0.y/gen_imgcfg.py
     python3 $TMP/gen_imgcfg.py -i $TMP/partition_universal.json -n sdcard.img -o $TMP/genimage.cfg
     ```

3. Generate the SDCARD image

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
