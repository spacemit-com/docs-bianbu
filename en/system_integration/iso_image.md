---
sidebar_position: 10
---

# ISO Image Creation Guide

Creates a bootable Live ISO from the official UEFI image (`.tar.gz`): boot into the Live desktop ("Try Bianbu") or install to the board with the Calamares installer ("Install Bianbu"). Currently only the `lxqt-uefi` variant yields a meaningful ISO.

> **Note**: the workflow described in this guide is **still under development, not yet mature, and provided for reference only**.

## Environment Requirements

- Root privileges (the `rootfs.ext4`/`bootfs.ext4` images are mounted and chroot operations run inside; with a container, use `--privileged`);
- **Recommended to run on a K3 board or a riscv64 machine**; on an x86_64 host, configure `qemu-user` (10.2+) binfmt first, as described in [Bianbu 4.0 ROOTFS Creation](bianbu_4.0_rootfs_create.md);
- Install the dependencies in the build environment:

  ```shell
  sudo apt-get -y install wget squashfs-tools xorriso mtools dosfstools apt-utils
  ```

- Disk space: reserve 20G+ (image, extracted directories, and squashfs intermediate artifacts);
- Input: the official UEFI variant image `.tar.gz` (containing `rootfs.ext4` / `bootfs.ext4`).

## Steps

### 1. Extract and mount

```shell
export TARBALL=Bianbu-LXQt-UEFI-K3-v4.0.1-20260529101530.tar.gz
export WORKDIR=k3-iso-work
mkdir -p $WORKDIR && tar xzf $TARBALL -C $WORKDIR
export R=$WORKDIR/rootfs   # $R: the mount point of rootfs.ext4; every later $R refers to this directory
mkdir -p $R
mount $WORKDIR/rootfs.ext4 $R
mount $WORKDIR/bootfs.ext4 $R/boot
mount -t proc /proc $R/proc
mount -t sysfs /sys $R/sys
mount -o bind /dev $R/dev
mount -o bind /dev/pts $R/dev/pts
```

### 2. Install the Live components (casper + Calamares)

```shell
chroot $R bash -c "
  apt-get remove -y -qq calamares-settings-bianbu 2>/dev/null || true
  apt-get update -qq
  apt-get install -y -qq casper calamares-settings-lubuntu
  grep -qx isofs /etc/initramfs-tools/modules || echo isofs >> /etc/initramfs-tools/modules
"
```

### 3. Adapt Calamares (riscv64/bianbu)

Calamares configuration changes with upstream versions. Key adaptation points (exact sed rules depend on the actual version; the official ISO installer configuration can also be used as a reference):

- `calamares/bootloader/main.py`: add a `riscv64-efi` bitness branch (`return "riscv64-efi", "grubriscv64.efi", "bootriscv64.efi"`);
- `bootloader.conf`: `efiBootloaderId: "bianbu"`;
- `settings.conf`: enable `unpackfs`, `initramfs`, `shellprocess` (clean up lubuntu-calamares shortcuts), remove inapplicable modules such as pkgselect/automirror; point `branding` at bianbu;
- `users.conf`: allow simple passwords (minLength 0, allowWeakPasswords true);
- `partition.conf`: partition name `bianbu_boot`, `name: "Bianbu <version>"`;
- `displaymanager.conf`: `executable: startlxqtwayland`, `desktopFile: lxqt-wayland`;
- `unpackfs.conf`: unpack source `/cdrom/casper/filesystem.squashfs`;
- `branding.desc` and the desktop entry: replace the version/name with Bianbu (Wayland compatibility tweaks such as `XDG_RUNTIME_DIR` may be needed);
- Add the local repository: `cdrom.sources` (the `CODENAME` variable is reused in later steps):

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

- Optional: install the kpmcore patch deb (e.g. `libkpmcore13_*k3fix*.deb`) into the rootfs, otherwise manual partitioning in the installer may crash.

### 4. Configure the Live GRUB entries (with devicetree)

Write `$R/etc/grub.d/09_bianbu_uefi` (same content as step 2 of [Bianbu 4.0 UEFI Image Creation](bianbu_4.0_uefi_image_create.md)), then:

```shell
chmod 0755 $R/etc/grub.d/09_bianbu_uefi
chmod 0644 $R/etc/grub.d/10_linux
chroot $R update-initramfs -u
chroot $R apt-get clean
```

### 5. Create the ESP boot image (BOOTRISCV64.EFI)

```shell
export KVER=$(basename "$(ls $R/boot/vmlinuz-* | sort -V | tail -1)")
export KVER=${KVER#vmlinuz-}    # full version string, including suffixes such as -generic
export I=iso-tree
rm -rf $I && mkdir -p $I/casper $I/boot/grub $I/EFI/boot $I/.disk \
    $I/dists/$CODENAME/main/binary-riscv64 $I/pool/main
touch $I/bianbu
cp $R/boot/vmlinuz-$KVER $I/casper/vmlinuz
cp $R/boot/initrd.img-$KVER $I/casper/initrd
cp -r $R/boot/spacemit $I/casper/spacemit
cp $R/boot/config-$KVER $R/boot/System.map-$KVER $I/casper/ 2>/dev/null || true

# ISO GRUB menu (Try / Install)
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

# Build BOOTRISCV64.EFI with grub-mkstandalone and put it into a 64M FAT efi.img
# Note: the grub.cfg must be copied into the rootfs first (host paths are not visible inside the chroot)
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

Note: `grub-mkstandalone` and `mksquashfs` must run serially, and `$R/proc`, `$R/sys`, `$R/dev` must be unmounted before `mksquashfs` (otherwise walking the rootfs hangs).

### 6. Create the Live filesystem (squashfs)

```shell
# Optional: download language packs into the apt cache; they are shipped in the ISO pool for offline installs
chroot $R bash -c 'DEBIAN_FRONTEND=noninteractive apt-get install --download-only --reinstall -y -qq \
    language-pack-zh-hans language-pack-gnome-zh-hans \
    language-pack-en language-pack-gnome-en 2>/dev/null || true'
for deb in "$R"/var/cache/apt/archives/*.deb; do
    [ -f "$deb" ] || continue
    name=$(basename "$deb"); src=${name%%_*}
    letter=${src:0:1}; case "$src" in lib*) letter=${src:0:4};; esac
    mkdir -p "$I/pool/main/$letter/$src"; cp "$deb" "$I/pool/main/$letter/$src/"
done

# Unmount the system resources (mandatory before mksquashfs walks the rootfs, otherwise it hangs)
umount $R/proc $R/sys $R/dev/pts $R/dev 2>/dev/null || true

mksquashfs $R $I/casper/filesystem.squashfs \
  -noappend -no-duplicates -no-recovery -wildcards \
  -comp zstd -Xcompression-level 19 -b 1M \
  -processors "$(nproc)" \
  -e "var/cache/apt/archives/*" -e "root/*" -e "root/.*" \
  -e "tmp/*" -e "tmp/.*" -e "swapfile" -e "boot/lost+found" -e "lost+found"
du -sx --block-size=1 "$I/casper/filesystem.squashfs" | cut -f1 > "$I/casper/filesystem.size"
```

### 7. Pack the ISO

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

## Usage

1. Burn or mount the ISO and boot via UEFI (the board must support UEFI firmware, i.e. it can boot the official 4.0 UEFI image);
2. Choose **Try Bianbu FS without installing** (Live desktop) or **Install Bianbu FS** (Calamares installer) in GRUB;
3. The installation works offline (the ISO bundles the local `cdrom.sources` repository and base language packs).

## FAQ

- **No desktop in Live / installer does not start**: make sure the Calamares `branding` and `displaymanager` settings from step 3 are applied, and that the initramfs contains the `isofs` module (step 2).
- **squashfs hangs**: make sure `$R/proc`, `$R/sys`, `$R/dev` are unmounted before running `mksquashfs`.
