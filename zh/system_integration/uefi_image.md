---
sidebar_position: 6
---

# UEFI固件与系统镜像制作指南

截至2025年07月01日, 当前只支持 **使用 spinor 启动的产品** 搭配 UEFI 固件。  
本文基于 **Bianbu Minimal 2.2** 环境，适用于 MUSE Pi Pro 板卡。  
教你如何一步一步制作基于EDK2的 RISC-V UEFI 固件，并制作可以刷机的系统镜像（支持 Titan 固件包 + SD 卡镜像）。

## 文档内容概览

本指南共包含以下部分：

1. **环境要求** - 工作空间配置和环境准备
2. **UEFI 固件制作** - 使用EDK2编译生成 `edk2.itb` 固件文件
3. **GRUB环境搭建 + ESP 分区** - 创建 ES P分区和配置 GRUB 启动器
4. **重新制作文件系统** - 生成 bootfs 和 rootfs 分区镜像
5. **制作 Titan 固件包** - 打包用于 Titan 工具烧录的固件包
6. **SD卡镜像制作** - 制作可直接写入SD卡的镜像文件

## 环境要求

1. **准备宿主机的环境**, 制作  的 ROOTFS 请参考 [Bianbu 2.1/2.2 ROOTFS制作](https://bianbu.spacemit.com/system_integration/bianbu_2.1_rootfs_create) **完成 Bianbu Minimal 2.2 ROOTFS 制作**。

2. **设置工作目录**
   设置工作空间环境变量，分别管理UEFI固件编译和镜像制作：

   ```shell
   export UEFI_WORKSPACE=/mnt/uefi-workspace
   export ROOTFS_WORKSPACE=/mnt/image-workspace
   mkdir -p $UEFI_WORKSPACE
   mkdir -p $ROOTFS_WORKSPACE
   ```

3. **准备 rootfs 和 boot 内容**
   - 清理先前制作的分区镜像文件 `rootfs.ext4` 和 `bootfs.ext4`
   - 将 rootfs 和 bootfs 目录的内容整合到工作目录，后续会重新制作文件系统：

     ```shell
     # 删除之前生成的分区镜像文件
     rm -f rootfs.ext4 bootfs.ext4

     # 将bootfs内容合并回rootfs的/boot目录
     mv bootfs/* rootfs/boot/ 

     # 将整合后的rootfs移动到专用工作空间
     mv rootfs $ROOTFS_WORKSPACE/rootfs
     ```

## UEFI 固件制作

在制作UEFI系统镜像之前，我们需要先制作UEFI固件。这是因为UEFI固件是系统启动的基础组件，它负责硬件初始化和引导加载程序的启动。传统的U-Boot启动方式需要特定的U-Boot镜像，而UEFI启动方式则需要符合UEFI标准的固件镜像。通过先制作UEFI固件（edk2.itb），我们才能替换掉传统的U-Boot镜像，从而实现UEFI标准的启动流程。

1. **安装编译依赖**

   ```shell
   apt install -y git sudo make gcc g++ uuid-dev python3 u-boot-tools 2to3 brotli
   ```

2. **获取 edk2 项目代码**

   edk2 是 TianoCore 项目的核心仓库，提供UEFI固件开发的基础框架和通用代码；edk2-platforms 是平台特定的代码仓库，包含各种硬件平台的支持代码和配置文件。两者配合使用来构建针对特定硬件平台的UEFI固件。

   ```shell
   cd $UEFI_WORKSPACE
   git clone https://gitee.com/spacemit-buildroot/edk2.git
   git -C edk2 submodule update --init
   git clone https://gitee.com/spacemit-buildroot/edk2-platforms.git
   git -C edk2-platforms submodule update --init
   ```

3. **编译UEFI固件**

   ```shell
   cd $UEFI_WORKSPACE
   export GCC5_RISCV64_PREFIX=riscv64-linux-gnu-
   export PACKAGES_PATH=$UEFI_WORKSPACE/edk2:$UEFI_WORKSPACE/edk2-platforms
   export PYTHON_COMMAND=python3
   export WORKSPACE=$UEFI_WORKSPACE
   
   # 初始化EDK2构建环境
   cd edk2/
   source edksetup.sh
   cd $UEFI_WORKSPACE
   ln /usr/bin/python3 /usr/bin/python
   # 编译BaseTools
   make -C $UEFI_WORKSPACE/edk2/BaseTools
   
   # 编译UEFI固件
   build -a RISCV64 -p Platform/Spacemit/K1/MUSE-Pi-Pro/MUSE-Pi-Pro.dsc -t GCC5 -b DEBUG
   
   # 将生成的ITB固件文件复制到ROOTFS文件系统工作空间并重命名
   cp $WORKSPACE/fitimage/MUSE-Pi-Pro/MUSE-Pi-Pro.itb $ROOTFS_WORKSPACE/edk2.itb
   ```

4. **验证生成的固件文件**

   编译完成后，可以在 ROOTFS 文件系统工作空间中找到生成的 **UEFI固件文件**：

   ```shell
   ls -la $ROOTFS_WORKSPACE/edk2.itb
   ```

   该文件即为用于SpacemiT K1平台的 **UEFI 固件镜像**。

## GRUB 安装与配置

在UEFI固件制作完成后，我们需要安装和配置GRUB引导器。GRUB是一个多重引导程序，它在UEFI固件初始化硬件后接管系统启动流程。UEFI固件会查找并加载存储在 ESP（EFI System Partition）分区中的GRUB引导器，然后由GRUB根据配置文件加载Linux内核、设备树和初始化镜像。因此，我们需要创建ESP分区、安装GRUB到ESP分区，并配置GRUB来正确识别和启动我们的Bianbu系统。

1. **环境准备**

   下载并烧录 Bianbu 2.2 镜像到 MUSE Pi Pro,建议使用 [Bianbu GNOME 2.2版本的固件](https://archive.spacemit.com/image/k1/version/bianbu/v2.2/bianbu-24.04-desktop-k1-v2.2-release-20250430190125.zip)，镜像获取方式及 Titan 烧录工具使用方法可参见: [镜像](https://bianbu.spacemit.com/image)

   烧录系统后，请**确保系统能够正常连接网络且网络**，因为制作分区文件的过程中需要下载软件包。

2. **创建ESP分区文件（给 GRUB 引导用）**

   在 MUSE Pi Pro 设备上，进入 Bianbu 2.2 系统后，执行以下命令制作ESP分区文件。

   ```shell
   sudo apt update
   sudo apt install -y dosfstools
   sudo dd if=/dev/zero of="$(echo ~)/efi.img" bs=1M count=200
   sudo mkfs.fat -F32 ~/efi.img
   sudo mkdir -p /boot/efi
   sudo mount ~/efi.img /boot/efi
   sudo apt install -y grub-efi
   sudo grub-install riscv64-efi
   sudo update-grub
   sudo umount /boot/efi
   sudo fsck.vfat -a ~/efi.img
   ```

   创建的 `efi.img` 文件就是我们需要的ESP分区文件，请设法将这个文件转移到我们的ROOTFS文件系统工作空间中，例如采用scp远程传输。

   以下是一个传输示例：

   - 首先，在Muse Pi Pro 上执行以下命令获取制作ESP分区的用户名以及系统的IP地址。

     ```shell
     whoami
     # 假设输出为: bianbu
     ip a | grep 'inet ' | grep -v '127.0.0.1' | awk '{print $2}' | cut -d'/' -f1
     # 假设输出为： 192.168.10.156
     ```

   - 然后,切换到**宿主机环境**，注意这里是宿主机环境，不在虚拟机中，使用 `scp` 命令将 `efi.img` 文件远程传输到工作目录。

     ```shell
     scp username@ip_addr:~/efi.img ~/bianbu-workspace
     # 例如：scp bianbu@192.168.10.156:~/efi.img ~/bianbu-workspace
     docker cp ~/bianbu-workspace/efi.img build-bianbu-rootfs:/mnt/image-workspace/
     # 这样efi.img文件就被我们传输到了制作系统镜像的工作目录下
     ```

3. **获取 bootfs 和 rootfs 的 UUID**

   记录 Bianbu 2.2 系统的 bootfs 和 rootfs 对应磁盘分区的 UUID，稍后制作 UEFI 的系统镜像时，要为 rootfs 和 bootfs 设置相同的 UUID。这样做是因为我们的 ESP 分区文件和系统配置是从先前烧录的 Bianbu 系统中提取出来的，如果制作的 UEFI 镜像使用不同的 UUID，那么从原系统复制的配置文件就无法正确识别新的分区，导致系统无法正常启动。

   ```shell
   # 查看rootfs分区的UUID
   blkid -s UUID -o value $(findmnt -n -o SOURCE /)
   # 假设输出为: 964235f6-e9c0-4b6d-9b4d-2567f6630358

   # 查看bootfs分区的UUID
   blkid -s UUID -o value $(findmnt -n -o SOURCE /boot)
   # 假设输出为: 26d39279-a9ae-485e-8467-dda899d9c3bf
   ```

4. **GRUB 配置**

   - 将更改获取的UUID同步到环境变量中，rootfs的UUID对应`UUID_ROOTFS`, bootfs的UUID对应`UUID_BOOTFS`.

   ```shell
   # 请将‘xxxxx’替换为对应的UUID
   export UUID_ROOTFS=xxxxx
   export UUID_BOOTFS=xxxxx
   # 例如：
   # export UUID_ROOTFS=964235f6-e9c0-4b6d-9b4d-2567f6630358
   # export UUID_BOOTFS=26d39279-a9ae-485e-8467-dda899d9c3bf
   ```

   - 准备好ESP分区文件之后，需要在rootfs中安装并配置grub.

   ```shell
   cd $ROOTFS_WORKSPACE
   # 挂载系统资源
   mount -t proc /proc rootfs/proc
   mount -t sysfs /sys rootfs/sys
   mount -o bind /dev rootfs/dev
   mount -o bind /dev/pts rootfs/dev/pts
   mkdir -p rootfs/boot/efi
   # 创建临时ESP分区镜像文件，用于grub-install操作
   # 这个临时文件仅用于满足grub-install的安装要求，让它能够在chroot环境中正确安装GRUB相关文件到rootfs中
   # 实际使用的ESP分区文件是从真实Bianbu 2.2系统中提取的efi.img
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
   # 卸载并删除临时ESP分区文件，因为实际的ESP分区使用的是从真实系统提取的efi.img
   umount tmp.img
   rm tmp.img
   # 卸载系统资源
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
      echo 'Loading Linux 6.6.63 ...'
      linux /vmlinuz-6.6.63 root=UUID=$UUID_ROOTFS ro  earlycon=sbi earlyprintk quiet splash plymouth.ignore-serial-consoles plymouth.prefer-fbcon console=ttyS0,115200 loglevel=8 clk_ignore_unused swiotlb=65536 rdinit=\/init workqueue.default_affinity_scope=system rootwait rootfstype=ext4
      echo 'Loading initial ramdisk ...'
      initrd /initrd.img-6.6.63
      echo 'Loading device tree blob...'
      devicetree /spacemit/6.6.63/k1-x_MUSE-Pi-Pro.dtb
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

   - 修改`fstab`配置文件，让ESP分区自动挂载。

   ```shell
   cat >rootfs/etc/fstab <<EOF
   # <file system>     <dir>    <type>  <options>                          <dump> <pass>
   UUID=$UUID_ROOTFS   /        ext4    defaults,noatime,errors=remount-ro 0      1
   UUID=$UUID_BOOTFS   /boot    ext4    defaults                           0      2
   UUID=$UUID_ESP    /boot/efi    vfat    umask=0077    0    1
   EOF
   ```

## 重新制作文件系统

```shell
cd $ROOTFS_WORKSPACE

# 创建bootfs目录并移动boot内容
mkdir -p bootfs
mv rootfs/boot/* bootfs/

# 生成新的文件系统
mke2fs -d bootfs -L bootfs -t ext4 -U $UUID_BOOTFS bootfs.ext4 "256M"
mke2fs -d rootfs -L rootfs -t ext4 -N 524288 -U $UUID_ROOTFS rootfs.ext4 "2048M"
```

需要注意的是，因为这里制作的是 Bianbu Minimal, 文件系统体积会比较小，所以设置 `rootfs.ext4` 文件系统大小为 2048M。如果是制作 Bianbu GNOME桌面版本 或 Bianbu LXQt桌面版本，请根据实际情况调整文件系统大小，例如 10240M。

## 制作 Titan 固件包

1. **安装依赖**

   ```shell
   apt -y install zip
   ```

2. **拷贝固件依赖的文件**

   ```shell
   export TMP=$ROOTFS_WORKSPACE/pack_dir
   mkdir -p $TMP/factory/
   
   # 复制基础固件文件
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_emmc.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_sd.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinand.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinor.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/FSBL.bin $TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/u-boot.itb $TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/env.bin $TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb $TMP
   
   # 复制UEFI固件文件
   cp $ROOTFS_WORKSPACE/edk2.itb $TMP
   
   # 复制ESP分区文件
   cp $ROOTFS_WORKSPACE/efi.img $TMP
   
   # 复制文件系统镜像
   cp $ROOTFS_WORKSPACE/bootfs.ext4 $TMP
   cp $ROOTFS_WORKSPACE/rootfs.ext4 $TMP
   ```

3. **下载参考分区表**

   ```shell
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/fastboot.yaml
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_2M.json
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_flash.json
   wget -P $TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
   ```

4. **修改分区表**

   - 修改 `partition_2M.json` 分区表，将 `u-boot.itb` 文件替换为 `edk2.itb` 文件：

   ```shell
   # 将u-boot.itb替换为edk2.itb
   sed -i 's/"image": "u-boot.itb"/"image": "edk2.itb"/g' $TMP/partition_2M.json
   ```

   - 修改 `partition_universal.json` 分区表，添加ESP分区并调整分区顺序：

   ```shell
   # 使用jq工具添加ESP分区并调整后续分区偏移量
   apt install -y jq
   
   # 备份原文件
   cp $TMP/partition_universal.json $TMP/partition_universal.json.bak
   
   # 第一步：调整现有分区的偏移量
   jq '.partitions = (.partitions | map(
     if .name == "bootfs" then
       .offset = "260M"
     elif .name == "rootfs" then
       .offset = "516M"
     else
       .
     end
   ))' $TMP/partition_universal.json > $TMP/partition_universal_temp.json
   
   # 第二步：在uboot分区后插入ESP分区
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
   
   # 验证修改结果
   echo "=== 修改后的分区配置 ==="
   jq '.partitions[] | select(.name == "uboot" or .name == "ESP" or .name == "bootfs" or .name == "rootfs") | {name, offset, size}' $TMP/partition_universal.json

   # 删除多余文件
   rm $TMP/partition_universal.json.bak
   ```

   - 修改 `partition_flash.json`，在 bootfs 分区的 image 数组中添加 `edk2.itb` 和 `efi.img` 文件：

   ```shell
   # 备份原文件
   cp $TMP/partition_flash.json $TMP/partition_flash.json.bak
   
   # 使用jq工具在bootfs分区的image数组中添加edk2.itb和efi.img
   jq '
   # 在bootfs分区的image数组中添加edk2.itb和efi.img
   .partitions = (.partitions | map(
     if .name == "bootfs" then
       .image += ["edk2.itb", "efi.img"]
     else
       .
     end
   ))
   ' $TMP/partition_flash.json > $TMP/partition_flash_new.json
   
   mv $TMP/partition_flash_new.json $TMP/partition_flash.json
   
   # 验证修改结果
   echo "=== bootfs分区的image配置 ==="
   jq '.partitions[] | select(.name == "bootfs") | .image' $TMP/partition_flash.json

   # 删除多余文件
   rm $TMP/partition_flash.json.bak
   ```

5. **打包 - 生成 zip 文件**

   ```shell
   cd $TMP
   # 创建Titan固件包
   zip -r ../bianbu-custom.zip *
   ```

## SD卡镜像制作

如果需要制作可直接写入SD卡的镜像文件，可以使用 **genimage** 工具：

1. **安装 genimage 工具**

   ```shell
   echo 'tzdata tzdata/Areas select Asia' | debconf-set-selections
   echo 'tzdata tzdata/Zones/Asia select Shanghai' | debconf-set-selections
   DEBIAN_FRONTEND=noninteractive apt-get -y install wget python3 genimage
   ```

2. **准备 SD 卡镜像内容**

   ```shell
   export SDCARD_TMP=$ROOTFS_WORKSPACE/sdcard_pack_dir
   mkdir -p $SDCARD_TMP/factory/
   
   # 复制固件文件
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_emmc.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_sd.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinand.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/bootinfo_spinor.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/FSBL.bin $SDCARD_TMP/factory
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/u-boot.itb $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/u-boot/spacemit/env.bin $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/rootfs/usr/lib/riscv64-linux-gnu/opensbi/generic/fw_dynamic.itb $SDCARD_TMP
   
   # 复制UEFI相关文件
   cp $ROOTFS_WORKSPACE/edk2.itb $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/efi.img $SDCARD_TMP
   
   # 复制文件系统镜像
   cp $ROOTFS_WORKSPACE/bootfs.ext4 $SDCARD_TMP
   cp $ROOTFS_WORKSPACE/rootfs.ext4 $SDCARD_TMP
   ```

3. **下载并修改分区表配置**

   ```shell
   # 下载基础分区表
   wget -P $SDCARD_TMP https://gitee.com/bianbu/image-config/raw/main/partition_universal.json
   
   # 修改分区表以支持UEFI启动
   # 第一步：调整现有分区的偏移量
   jq '.partitions = (.partitions | map(
     if .name == "bootfs" then
       .offset = "260M"
     elif .name == "rootfs" then
       .offset = "516M"
     else
       .
     end
   ))' $SDCARD_TMP/partition_universal.json > $SDCARD_TMP/partition_universal_temp.json
   
   # 第二步：在uboot分区后插入ESP分区
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

4. **生成 genimage 配置文件**

   ```shell
   # 下载genimage配置生成脚本
   wget -P $SDCARD_TMP https://gitee.com/spacemit-buildroot/scripts/raw/bl-v1.0.y/gen_imgcfg.py
   
   # 生成genimage配置
   python3 $SDCARD_TMP/gen_imgcfg.py -i $SDCARD_TMP/partition_universal.json -n bianbu-uefi.sdcard -o $SDCARD_TMP/genimage.cfg
   ```

5. **制作SD卡镜像**

   ```shell
   # 创建临时目录
   ROOTPATH_TMP="$(mktemp -d)"
   GENIMAGE_TMP="$(mktemp -d)"
   
   # 生成SDCARD镜像
   genimage \
       --config "$SDCARD_TMP/genimage.cfg" \
       --rootpath "$ROOTPATH_TMP" \
       --tmppath "$GENIMAGE_TMP" \
       --inputpath "$SDCARD_TMP" \
       --outputpath "."
   
   # 清理临时目录
   rm -rf "$ROOTPATH_TMP" "$GENIMAGE_TMP"
   
   echo "=== SDCARD镜像制作完成 ==="
   ls -la bianbu-uefi.sdcard
   ```

   当看到以下信息时，说明**打包成功**。

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
