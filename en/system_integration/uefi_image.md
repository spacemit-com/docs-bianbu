---
sidebar_position: 6
---

# UEFI Firmware and System Image Creation Guide

As of the latest document modification date of July 1, 2025, only products using **SPI NOR** for booting support UEFI firmware. This document uses the MUSE Pi Pro as an example and be based on Bianbu Minimal 2.2 to introduce how to create RISC-V UEFI firmware and system images based on EDK2.

## Document Content Overview

1. **Environment Requirements** - Workspace configuration and environment preparation
2. **UEFI Firmware Creation** - Compiling and generating the `edk2.itb` firmware file using EDK2
3. **GRUB Environment Setup** - Creating the ESP partition and configuring the GRUB bootloader
4. **Recreating File Systems** - Generating bootfs and rootfs partition images
5. **Titan Firmware** - Packaging the firmware for Titan Flasher Tool
6. **SDCARD Image Creation** - Creating an image file that can be directly written to an SD card


## Environment Requirements

The host machine's environment preparation and the creation of the Bianbu Minimal 2.2 ROOTFS can be found in the [Bianbu 2.1/2.2 ROOTFS Creation Guide](https://bianbu.spacemit.com/en/system_integration/bianbu_2.1_rootfs_create/). After preparing the container environment and ROOTFS, it is possible to proceed with the creation of the UEFI firmware and system image.

First, set up the workspace environment variables to manage the UEFI firmware compilation and image creation:

```shell
export UEFI_WORKSPACE=/mnt/uefi-workspace
export ROOTFS_WORKSPACE=/mnt/image-workspace
mkdir -p $UEFI_WORKSPACE
mkdir -p $ROOTFS_WORKSPACE
```

Then, clean up the previously created partition image files `rootfs.ext4` and `bootfs.ext4`, and integrate the contents of the rootfs and bootfs directories into the working directory. The file system will be recreated later:

```shell
# Delete the previously generated partition image files

rm -f rootfs.ext4 bootfs.ext4

# Merge the content of bootfs back into the boot directory of rootfs
mv bootfs/* rootfs/boot/ 

# Move the integrated rootfs to a dedicated workspace
mv rootfs $ROOTFS_WORKSPACE/rootfs
```

## UEFI Firmware Creation

Before creating the UEFI system image, it is necessary to create the UEFI firmware first. This is because the UEFI firmware is a fundamental component of the system boot process, responsible for hardware initialization and the startup of the bootloader. Traditional U-Boot boot methods require specific U-Boot images, while UEFI boot methods require firmware images that comply with the UEFI standard. By first creating the UEFI firmware (`edk2.itb`), it is possible to replace the traditional U-Boot image to achieve the UEFI standard boot process.

1. Install compilation dependencies
   ```shell
   apt install -y git sudo make gcc g++ uuid-dev python3 u-boot-tools 2to3 brotli
   ```

2. Clone the edk2 project code

   edk2 is the core repository of the TianoCore project, providing the basic framework and common code for UEFI firmware development. edk2-platforms is a platform-specific code repository, containing support code and configuration files for various hardware platforms. Both are used together to build UEFI firmware for specific hardware platforms.

   ```shell
   cd $UEFI_WORKSPACE
   git clone https://gitee.com/spacemit-buildroot/edk2.git
   git -C edk2 submodule update --init
   git clone https://gitee.com/spacemit-buildroot/edk2-platforms.git
   git -C edk2-platforms submodule update --init
   ```

3. Compile the UEFI firmware
   ```shell
   cd $UEFI_WORKSPACE
   export GCC5_RISCV64_PREFIX=riscv64-linux-gnu-
   export PACKAGES_PATH=$UEFI_WORKSPACE/edk2:$UEFI_WORKSPACE/edk2-platforms
   export PYTHON_COMMAND=python3
   export WORKSPACE=$UEFI_WORKSPACE
   
   # Initialize the EDK2 build environment
   cd edk2/
   source edksetup.sh
   cd $UEFI_WORKSPACE
   ln /usr/bin/python3 /usr/bin/python
   # Compile BaseTools
   make -C $UEFI_WORKSPACE/edk2/BaseTools
   
   # Compile the UEFI firmware
   build -a RISCV64 -p Platform/Spacemit/K1/MUSE-Pi-Pro/MUSE-Pi-Pro.dsc -t GCC5 -b DEBUG
   
   # Copy the generated ITB firmware file to the ROOTFS file system workspace and rename it
   cp $WORKSPACE/fitimage/MUSE-Pi-Pro/MUSE-Pi-Pro.itb $ROOTFS_WORKSPACE/edk2.itb
   ```

4. Verify the generated firmware file

   After compilation, the generated UEFI firmware file can be found in the ROOTFS file system workspace.

   ```shell
   ls -la $ROOTFS_WORKSPACE/edk2.itb
   ```

   This file is the UEFI firmware image for the SpacemiT K1 platform.

## GRUB Installation and Configuration

After the UEFI firmware creation is completed, it is necessary to install and configure the GRUB bootloader. GRUB is a multi-boot program that takes over the system boot process after the UEFI firmware initializes the hardware. The UEFI firmware will look for and load the GRUB bootloader stored in the ESP (EFI System Partition). Then, GRUB will load the Linux kernel, device tree, and initramfs according to the configuration file. Therefore, it is necessary to create the ESP partition, install GRUB to the ESP partition, and configure GRUB to correctly identify and boot Bianbu system.

1. Environment Preparation

   Download and flash the Bianbu 2.2 image to the MUSE Pi Pro. It is recommended to use the [Bianbu GNOME Desktop 2.2 firmware version](https://archive.spacemit.com/image/k1/version/bianbu/v2.2/bianbu-24.04-desktop-k1-v2.2-release-20250430190125.zip)，the method for obtaining the image and the usage of the Titan Flasher tool can be found at [Image](https://bianbu.spacemit.com/en/image/).
   
   After flashing the system, ensure that the system can connect to the network and that the network connection is normal, as the process of creating partition files requires downloading software packages.   

2. Create ESP Partition File

   After entering the Bianbu 2.2 system, execute the following commands to create the ESP partition file:

   ```shell
   sudo apt update
   sudo apt install -y dosfstools
   sudo dd if=/dev/zero of=~/efi.img bs=1M count=200
   sudo mkfs.fat -F32 ~/efi.img
   sudo mkdir -p /boot/efi
   sudo mount ~/efi.img /boot/efi
   sudo apt install -y grub-efi
   sudo grub-install riscv64-efi
   sudo update-grub
   sudo umount /boot/efi
   sudo fsck.vfat -a ~/efi.img
   ```
   The created `efi.img` file is the ESP partition file required. Please transfer this file to the ROOTFS file system workspace, for example using scp for remote transfer.

   Here is a sample transfer command:
   
   >First, execute the following commands on the Muse Pi Pro to obtain the username used to create the ESP partition and the system's IP address.
   >
   > ```shell
   >whoami
   ># Assume the output is: bianbu
   >ip a | grep 'inet ' | grep -v '127.0.0.1' | awk '{print $2}' | cut -d'/' -f1
   ># Assume the output is: 192.168.10.156
   >```
   >
   >Then switch to the host environment (note that this is the host environment, not inside the virtual machine) and use the scp command to remotely transfer the efi.img file to the working directory.
   >
   >```shell
   >scp username@ip_addr:~/efi.img ~/bianbu-workspace
   ># For example: scp bianbu@192.168.10.156:~/efi.img ~/bianbu-workspace
   >docker cp ~/bianbu-workspace/efi.img build-bianbu-rootfs:/mnt/image-workspace/
   ># This way, the efi.img file has been transferred to the working directory where the system image is being created
   >```

   In addition, record the UUIDs of the bootfs and rootfs disk partitions from the Bianbu 2.2 system. When the UEFI system image is built later, it is a must to assign the same UUIDs to rootfs and bootfs. This is necessary because the ESP partition file and system configuration were extracted from the previously flashed Bianbu system. If the new UEFI image uses different UUIDs, the copied configuration files will not correctly identify the new partitions, causing the system to fail to boot.

   ```shell
   # Check the UUID of the rootfs partition:
   blkid -s UUID -o value $(findmnt -n -o SOURCE /)
   # Assume the output is: 964235f6-e9c0-4b6d-9b4d-2567f6630358

   # Check the UUID of the bootfs partition
   blkid -s UUID -o value $(findmnt -n -o SOURCE /boot)
   # Assume the output is: 26d39279-a9ae-485e-8467-dda899d9c3bf
   ```

3. GRUB Configuration

   First, synchronize the obtained UUIDs into environment variables: the UUID for rootfs corresponds to UUID_ROOTFS, and the UUID for bootfs corresponds to UUID_BOOTFS.
   
   ```shell
   # Replace 'xxxxx' with the corresponding UUID.
   export UUID_ROOTFS=xxxxx
   export UUID_BOOTFS=xxxxx
   # For example:
   # export UUID_ROOTFS=964235f6-e9c0-4b6d-9b4d-2567f6630358
   # export UUID_BOOTFS=26d39279-a9ae-485e-8467-dda899d9c3bf
   ```

   After the ESP partition file is ready, install and configure GRUB inside rootfs.

   ```shell
   cd $ROOTFS_WORKSPACE
   # Mount system resources.
   mount -t proc /proc rootfs/proc
   mount -t sysfs /sys rootfs/sys
   mount -o bind /dev rootfs/dev
   mount -o bind /dev/pts rootfs/dev/pts
   mkdir -p rootfs/boot/efi
   # Create a temporary ESP partition image file for the grub-install operation.
   # This temporary file is only used to satisfy grub-install’s installation requirements, allowing it to correctly install GRUB-related files into rootfs within the chroot environment.
   # The actual ESP partition file used is efi.img extracted from the real Bianbu 2.2 system.
   dd if=/dev/zero of=tmp.img bs=1M count=200
   apt update
   apt install -y dosfstools
   mkfs.fat -F32 tmp.img
   mount tmp.img rootfs/boot/efi/
   chroot rootfs /bin/bash -c "apt update"
   chroot rootfs /bin/bash -c "apt install -y grub-efi"
   chroot rootfs /bin/bash -c "grub-install riscv-efi"
   chroot rootfs /bin/bash -c "update-grub"
   chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
   # Unmount and delete the temporary ESP partition file, since the actual ESP partition uses efi.img extracted from the real system
   umount tmp.img
   rm tmp.img
   # Unmount system resources.
   mount | grep "rootfs/proc" > /dev/null && umount -l rootfs/proc
   mount | grep "rootfs/sys" > /dev/null && umount -l rootfs/sys
   mount | grep "rootfs/dev/pts" > /dev/null && umount -l rootfs/dev/pts
   mount | grep "rootfs/dev" > /dev/null && umount -l rootfs/dev
   echo 'usb_start=usb start' >> rootfs/boot/env_k1-x.txt
   export UUID_ESP=$(blkid efi.img | grep -o 'UUID="[^"]*"' | cut -d '"' -f 2)
   cat > rootfs/boot/grub/grub.cfg <<EOF
   #
   # DO NOT EDIT THIS FILE
   #
   # It is automatically generated by grub-mkconfig using templates
   # from /etc/grub.d and settings from /etc/default/grub
   #

   ### BEGIN /etc/grub.d/00_header ###
   if [ -s \$prefix/grubenv ]; then
   set have_grubenv=true
   load_env
   fi
   if [ "\${initrdfail}" = 2 ]; then
      set initrdfail=
   elif [ "\${initrdfail}" = 1 ]; then
      set next_entry="\${prev_entry}"
      set prev_entry=
      save_env prev_entry
      if [ "\${next_entry}" ]; then
         set initrdfail=2
      fi
   fi
   if [ "\${next_entry}" ] ; then
      set default="\${next_entry}"
      set next_entry=
      save_env next_entry
      set boot_once=true
   else
      set default="0"
   fi

   if [ x"\${feature_menuentry_id}" = xy ]; then
   menuentry_id_option="--id"
   else
   menuentry_id_option=""
   fi

   export menuentry_id_option

   if [ "\${prev_saved_entry}" ]; then
   set saved_entry="\${prev_saved_entry}"
   save_env saved_entry
   set prev_saved_entry=
   save_env prev_saved_entry
   set boot_once=true
   fi

   function savedefault {
   if [ -z "\${boot_once}" ]; then
      saved_entry="\${chosen}"
      save_env saved_entry
   fi
   }
   function initrdfail {
      if [ -n "\${have_grubenv}" ]; then if [ -n "\${partuuid}" ]; then
         if [ -z "\${initrdfail}" ]; then
         set initrdfail=1
         if [ -n "\${boot_once}" ]; then
            set prev_entry="\${default}"
            save_env prev_entry
         fi
         fi
         save_env initrdfail
      fi; fi
   }
   function load_video {
   if [ x\$feature_all_video_module = xy ]; then
      insmod all_video
   else
      insmod efi_gop
      insmod efi_uga
      insmod ieee1275_fb
      insmod vbe
      insmod vga
      insmod video_bochs
      insmod video_cirrus
   fi
   }

   if [ x\$feature_default_font_path = xy ] ; then
      font=unicode
   else
   insmod part_gpt
   insmod ext2
   search --no-floppy --fs-uuid --set=root $UUID_ROOTFS
      font="/usr/share/grub/unicode.pf2"
   fi

   if loadfont \$font ; then
   set gfxmode=auto
   load_video
   insmod gfxterm
   set locale_dir=\$prefix/locale
   set lang=zh_CN
   insmod gettext
   fi
   terminal_output gfxterm
   if [ "\${recordfail}" = 1 ] ; then
   set timeout=30
   else
   if [ x\$feature_timeout_style = xy ] ; then
      set timeout_style=menu
      set timeout=10
   # Fallback normal timeout code in case the timeout_style feature is
   # unavailable.
   else
      set timeout=10
   fi
   fi
   ### END /etc/grub.d/00_header ###

   ### BEGIN /etc/grub.d/05_debian_theme ###
   set menu_color_normal=cyan/blue
   set menu_color_highlight=white/blue
   ### END /etc/grub.d/05_debian_theme ###

   ### BEGIN /etc/grub.d/10_linux ###
   function gfxmode {
      set gfxpayload="\${1}"
   }
   set linux_gfx_mode=
   export linux_gfx_mode
   menuentry 'Bianbu GNU/Linux' --class bianbu --class gnu-linux --class gnu --class os \$menuentry_id_option 'gnulinux-simple-$UUID_ROOTFS' {
      load_video
      insmod gzio
      if [ x\$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
      insmod part_gpt
      insmod ext2
      search --no-floppy --fs-uuid --set=root $UUID_BOOTFS
      echo	'Loading Linux 6.6.63 ...'
      linux	/vmlinuz-6.6.63 root=UUID=$UUID_ROOTFS ro  earlycon=sbi earlyprintk quiet splash plymouth.ignore-serial-consoles plymouth.prefer-fbcon console=ttyS0,115200 loglevel=8 clk_ignore_unused swiotlb=65536 rdinit=\/init workqueue.default_affinity_scope=system rootwait rootfstype=ext4
      echo	'Loading initial ramdisk ...'
      initrd	/initrd.img-6.6.63
      echo	'Loading device tree blob...'
      devicetree	/spacemit/6.6.63/k1-x_MUSE-Pi-Pro.dtb
   }

   ### END /etc/grub.d/10_linux ###

   ### BEGIN /etc/grub.d/10_linux_zfs ###
   ### END /etc/grub.d/10_linux_zfs ###

   ### BEGIN /etc/grub.d/20_linux_xen ###
   ### END /etc/grub.d/20_linux_xen ###

   ### BEGIN /etc/grub.d/25_bli ###
   if [ "\$grub_platform" = "efi" ]; then
   insmod bli
   fi
   ### END /etc/grub.d/25_bli ###

   ### BEGIN /etc/grub.d/30_os-prober ###
   ### END /etc/grub.d/30_os-prober ###

   ### BEGIN /etc/grub.d/30_uefi-firmware ###
   if [ "\$grub_platform" = "efi" ]; then
      fwsetup --is-supported
      if [ "\$?" = 0 ]; then
         menuentry 'UEFI Firmware Settings' \$menuentry_id_option 'uefi-firmware' {
            fwsetup
         }
      fi
   fi
   ### END /etc/grub.d/30_uefi-firmware ###

   ### BEGIN /etc/grub.d/35_fwupd ###
   ### END /etc/grub.d/35_fwupd ###

   ### BEGIN /etc/grub.d/40_custom ###
   # This file provides an easy way to add custom menu entries.  Simply type the
   # menu entries you want to add after this comment.  Be careful not to change
   # the 'exec tail' line above.
   ### END /etc/grub.d/40_custom ###

   ### BEGIN /etc/grub.d/41_custom ###
   if [ -f  \${config_directory}/custom.cfg ]; then
   source \${config_directory}/custom.cfg
   elif [ -z "\${config_directory}" -a -f  \$prefix/custom.cfg ]; then
   source \$prefix/custom.cfg
   fi
   ### END /etc/grub.d/41_custom ###
   EOF
   ```

   Edit the fstab configuration file to enable automatic mounting of the ESP partition.

   ```shell
   cat >rootfs/etc/fstab <<EOF
   # <file system>     <dir>    <type>  <options>                          <dump> <pass>
   UUID=$UUID_ROOTFS   /        ext4    defaults,noatime,errors=remount-ro 0      1
   UUID=$UUID_BOOTFS   /boot    ext4    defaults                           0      2
   UUID=$UUID_ESP    /boot/efi    vfat    umask=0077    0    1
   EOF
   ```

## Recreate the File System

```shell
cd $ROOTFS_WORKSPACE

# Create the bootfs directory and move the boot content into it.
mkdir -p bootfs
mv rootfs/boot/* bootfs/

# Generate the new file system.
mke2fs -d bootfs -L bootfs -t ext4 -U $UUID_BOOTFS bootfs.ext4 "256M"
mke2fs -d rootfs -L rootfs -t ext4 -N 524288 -U $UUID_ROOTFS rootfs.ext4 "2048M"
```

**Note:** Due to the relatively small file system in **Bianbu Minimal** builds, the `rootfs.ext4` file-system size is set to 2048 MB by default. For **Bianbu GNOME Desktop Version** or **Bianbu LXQt Desktop Version** builds, file-system size adjustment according to actual requirements is recommended (e.g., 10240 MB).

## Titan Firmware

1. Install dependencies

   ```shell
   apt -y install zip
   ```

2. Copy the firmware dependency files

   ```shell
   export TMP=$ROOTFS_WORKSPACE/pack_dir
   mkdir -p $TMP/factory/
   
   # Copy the base firmware files
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_emmc.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_sd.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinand.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinor.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/FSBL.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/u-boot.itb $TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/env.bin $TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb $TMP
   
   # Create the UEFI firmware file
   cp $ROOTFS_WORKSPACE/edk2.itb $TMP
   
   # Copy ESP partition files
   cp $ROOTFS_WORKSPACE/efi.img $TMP
   
   # Copy the file system image
   cp $ROOTFS_WORKSPACE/bootfs.ext4 $TMP
   cp $ROOTFS_WORKSPACE/rootfs.ext4 $TMP
   ```

3. Load reference partition table

   ```shell
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/fastboot.yaml
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_2M.json
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_flash.json
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
   ```

4. Modify partition table

   Modify the `partition_2M.json` partition table by replacing the `u-boot.itb` file with the `edk2.itb` file:

   ```shell
   # Replace u-boot.itb with edk2.itb
   sed -i 's/"image": "u-boot.itb"/"image": "edk2.itb"/g' $TMP/partition_2M.json
   ```

   Modify the `partition_universal.json` partition table by adding the ESP partition and adjusting the partition order:

   ```shell
   # Use the jq tool to add the ESP partition and adjust the offsets of subsequent partitions
   apt install -y jq
   
   # Backup the original file
   cp $TMP/partition_universal.json $TMP/partition_universal.json.bak
   
   # Step 1: Adjust the offsets of existing partitions
   jq '.partitions = (.partitions | map(
     if .name == "bootfs" then
       .offset = "260M"
     elif .name == "rootfs" then
       .offset = "516M"
     else
       .
     end
   ))' $TMP/partition_universal.json > $TMP/partition_universal_temp.json
   
   # Step 2: Insert the ESP partition after the uboot partition
   jq '.partitions = (
     (.partitions | map(select(.name == "bootinfo" or .name == "fsbl" or .name == "env" or .name == "opensbi" or .name == "uboot"))) +
     [{
       "name": "ESP",
       "offset": "4M",
       "size": "256M",
       "image": "efi.img"
     }] +
     (.partitions | map(select(.name == "bootfs" or .name == "rootfs")))
   )' $TMP/partition_universal_temp.json > $TMP/partition_universal_new.json
   
   mv $TMP/partition_universal_new.json $TMP/partition_universal.json
   rm $TMP/partition_universal_temp.json
   
   # Verify the modification results
   echo "=== Modified Partition Configuration ==="
   jq '.partitions[] | select(.name == "uboot" or .name == "ESP" or .name == "bootfs" or .name == "rootfs") | {name, offset, size}' $TMP/partition_universal.json

   # Delete redundant files
   rm $TMP/partition_universal.json.bak
   ```

   Modify `partition_flash.json` by adding `edk2.itb` and `efi.img` files to the image array of the bootfs partition:

   ```shell
   # Backup the original file
   cp $TMP/partition_flash.json $TMP/partition_flash.json.bak
   
   # Use jq tool to add edk2.itb and efi.img to the image array of the bootfs partition
   jq '
   # Add edk2.itb and efi.img to the image array in the bootfs partition
   .partitions = (.partitions | map(
     if .name == "bootfs" then
       .image += ["edk2.itb", "efi.img"]
     else
       .
     end
   ))
   ' $TMP/partition_flash.json > $TMP/partition_flash_new.json
   
   mv $TMP/partition_flash_new.json $TMP/partition_flash.json
   
   # Verify the modification results
   echo "=== Image configuration of the bootfs partition ==="
   jq '.partitions[] | select(.name == "bootfs") | .image' $TMP/partition_flash.json

   # Delete redundant files
   rm $TMP/partition_flash.json.bak
   ```


5. Shell

   ```shell
   cd $TMP
   # Create the Titan firmware package
   zip -r ../bianbu-custom.zip *
   ```

## Creating SDCARD Image

For creating an image file directly writable to an SD card, the genimage tool can be used:

1. Installing genimage tool

   ```shell
   echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections
   echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections
   DEBIAN_FRONTEND=noninteractive apt-get -y install wget python3 genimage
   ```

2. Preparing files for SDCARD image creation

   ```shell
   export SDCARD_TMP=$ROOTFS_WORKSPACE/sdcard_pack_dir
   mkdir -p $SDCARD_TMP/factory/
   
   # Copying firmware files
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_emmc.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_sd.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinand.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinor.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/FSBL.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/u-boot.itb $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/env.bin $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb $SDCARD_TMP
   
   # Copying UEFI-related files
   cp $ROOTFS_WORKSPACE/edk2.itb $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/efi.img $SDCARD_TMP
   
   # Creating the filesystem image
   cp $ROOTFS_WORKSPACE/bootfs.ext4 $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/rootfs.ext4 $SDCARD_TMP
   ```

3. Loading and modifying partition table configuration

   ```shell
   # Loading the base partition table
   wget -P $SDCARD_TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
   
   # Modification of the partition table to support UEFI boot
   # Step 1: Adjustment of existing partition offsets
   jq '.partitions = (.partitions | map(
     if .name == "bootfs" then
       .offset = "260M"
     elif .name == "rootfs" then
       .offset = "516M"
     else
       .
     end
   ))' $SDCARD_TMP/partition_universal.json > $SDCARD_TMP/partition_universal_temp.json
   
   # Step 2: Insertion of the ESP partition after the uboot partition
   jq '.partitions = (
     (.partitions | map(select(.name == "bootinfo" or .name == "fsbl" or .name == "env" or .name == "opensbi" or .name == "uboot"))) +
     [{
       "name": "ESP",
       "offset": "4M",
       "size": "256M",
       "image": "efi.img"
     }] +
     (.partitions | map(select(.name == "bootfs" or .name == "rootfs")))
   )' $SDCARD_TMP/partition_universal_temp.json > $SDCARD_TMP/partition_universal_new.json
   
   mv $SDCARD_TMP/partition_universal_new.json $SDCARD_TMP/partition_universal.json
   rm $SDCARD_TMP/partition_universal_temp.json
   ```

4. Generation of genimage configuration file

   ```shell
   # Download of genimage configuration generation script
   wget -P $SDCARD_TMP https://gitee.com/spacemit-buildroot/scripts/raw/bl-v1.0.y/gen_imgcfg.py
   
   # Generation of genimage configuration
   python3 $SDCARD_TMP/gen_imgcfg.py -i $SDCARD_TMP/partition_universal.json -n bianbu-uefi.sdcard -o $SDCARD_TMP/genimage.cfg
   ```

5. Creation of the SDCARD image

   ```shell
   # Creation of temporary directory
   ROOTPATH_TMP="$(mktemp -d)"
   GENIMAGE_TMP="$(mktemp -d)"
   
   # Generation of the SDCARD image
   genimage \
       --config "$SDCARD_TMP/genimage.cfg" \
       --rootpath "$ROOTPATH_TMP" \
       --tmppath "$GENIMAGE_TMP" \
       --inputpath "$SDCARD_TMP" \
       --outputpath "."
   
   # Cleanup of temporary directory
   rm -rf "$ROOTPATH_TMP" "$GENIMAGE_TMP"
   
   echo "=== SDCARD image creation completed ==="
   ls -la bianbu-uefi.sdcard
   ```

   Indication of successful packaging upon seeing the following messages:

   ```shell
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'bootinfo' from 'factory/bootinfo_sd.bin' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'fsbl' (in MBR) from 'factory/FSBL.bin' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'env' (in MBR) from 'env.bin' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'opensbi' (in MBR) from 'fw_dynamic.itb' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'uboot' (in MBR) from 'u-boot.itb' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'ESP' (in MBR) from 'efi.img' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'bootfs' (in MBR) from 'bootfs.ext4' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition 'rootfs' (in MBR) from 'rootfs.ext4' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition '[MBR]' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition '[GPT header]' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition '[GPT array]' ...
   INFO: hdimage(bianbu-uefi.sdcard): adding partition '[GPT backup]' ...
   INFO: hdimage(bianbu-uefi.sdcard): writing GPT
   INFO: hdimage(bianbu-uefi.sdcard): writing protective MBR
   INFO: hdimage(bianbu-uefi.sdcard): writing MBR
   ```
