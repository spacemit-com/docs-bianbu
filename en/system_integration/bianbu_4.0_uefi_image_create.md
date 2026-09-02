---
sidebar_position: 9
---

# Bianbu 4.0 UEFI Image Creation

Bianbu 4.0 supports an EDK2-based UEFI boot mode: the UEFI firmware (`edk2.itb`) replaces the traditional U-Boot bootloader; the bootfs/rootfs layout stays the same, and the system is loaded by GRUB (located in the ESP partition) with the kernel and the per-board device tree. Based on [Bianbu 4.0 ROOTFS Creation](bianbu_4.0_rootfs_create.md), three differences apply when creating a UEFI image:

1. Install the UEFI software packages (GRUB + `edk2-spacemit`) in the rootfs and configure the GRUB entries with devicetree;
2. Add an ESP partition image (`esp.vfat`) containing GRUB;
3. In the partition tables, the `uboot` partition maps to `edk2.itb`, and an `ESP` partition is added.

> For UEFI firmware and image creation on Bianbu 2.x (K1, manually building EDK2), see the [UEFI Firmware and System Image Creation Guide](uefi_image.md).

## UEFI Boot Chain

```
FSBL → esos → OpenSBI → edk2.itb (UEFI firmware) → BOOTRISCV64.EFI (GRUB, ESP partition)
     → kernel + device tree (/boot/spacemit/<kernel version>/<product_name>.dtb)
```

- For each kernel version there is a `/boot/spacemit/<version>/` directory with the DTB for each board model;
- GRUB reads the product name via `efienv -g spacemit update product_name` and selects the matching DTB dynamically;
- The kernel and initramfs come from the boot entry; the DTB is the only part that varies per board.

## Stage 1: configure UEFI components in the ROOTFS

After completing [Bianbu 4.0 ROOTFS Creation](bianbu_4.0_rootfs_create.md) steps 1–4 (including `bianbu-minimal`), continue:

### 1. Install the UEFI packages

```shell
chroot rootfs /bin/bash -c "apt-get update"
chroot rootfs /bin/bash -c "DEBIAN_FRONTEND=noninteractive UCF_FORCE_CONFFNEW=1 apt-get -y --allow-downgrades install \
    grub-common grub2-common grub-efi-riscv64-bin grub-efi-riscv64-unsigned grub-efi-riscv64 \
    binutils edk2-spacemit"
```

`edk2-spacemit` provides the UEFI firmware `edk2.itb` (installed at `usr/lib/uefi/spacemit/edk2.itb`); the GRUB packages provide `grubriscv64.efi` (`usr/lib/grub/riscv64-efi/monolithic/`).

### 2. Configure the GRUB boot entries

Write `etc/default/grub.d/99-bianbu-uefi.cfg`:

```shell
cat > rootfs/etc/default/grub.d/99-bianbu-uefi.cfg <<EOF
GRUB_TIMEOUT=5
GRUB_TIMEOUT_STYLE=menu
GRUB_DISABLE_OS_PROBER=true
GRUB_DISABLE_RECOVERY=true
GRUB_DEFAULT=0
EOF
```

Write `etc/grub.d/09_bianbu_uefi`, which generates boot entries with devicetree (each kernel version gets its per-board DTB; older kernel entries double as recovery entries):

```shell
cat > rootfs/etc/grub.d/09_bianbu_uefi <<'EOF'
#!/bin/sh
set -e

# Bianbu UEFI boot entries with per-board devicetree selection (efienv product_name).
# Generates entries ahead of 10_linux (which is disabled) so the system
# loads the correct board DTB from /boot/spacemit/<kver>/.

DIST_LABEL='@DIST_LABEL@'       # Bianbu / Ubuntu
VARIANT_LABEL='@VARIANT_LABEL@' # e.g. LXQt / Minimal
BOARD='@BOARD@'                 # k3 / k1

# Detect a chroot: at build time there are no real block devices and grub-probe can
# misread the host, so emit LABEL placeholders (replaced with final UUIDs when
# packaging); a real-machine update-grub probes real UUIDs.
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

# Replace the placeholders
sed -i \
    -e "s|@DIST_LABEL@|Bianbu|g" \
    -e "s|@VARIANT_LABEL@|LXQt|g" \
    -e "s|@BOARD@|k3|g" \
    rootfs/etc/grub.d/09_bianbu_uefi

# Enable 09_bianbu_uefi, disable 10_linux (its entries carry no devicetree and cannot boot under UEFI)
chmod 0755 rootfs/etc/grub.d/09_bianbu_uefi
chmod 0644 rootfs/etc/grub.d/10_linux

# Generate grub.cfg (LABEL placeholders at build time; replaced with real UUIDs when packaging)
chroot rootfs /bin/bash -c "update-grub"
```

## Stage 2: Package the UEFI Image

On top of the "Create the Partition Images" stage of [Bianbu 4.0 ROOTFS Creation](bianbu_4.0_rootfs_create.md) and the packaging procedure of the [Image Creation Guide](image.md), add/replace the following steps. The commands below reuse the `$UUID_BOOTFS` / `$UUID_ROOTFS` variables exported in step 1 of that guide's partition-image stage (re-run the `export` if the shell was changed):

### 1. Firmware files and the ESP partition image

```shell
export TMP=pack_dir

# Use edk2.itb instead of u-boot.itb (copied to the pack dir for the partition tables)
cp rootfs/usr/lib/uefi/spacemit/edk2.itb $TMP

# Create the ESP partition image (256M) and install GRUB
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

### 2. Update /etc/fstab (add the ESP)

```shell
cat > rootfs/etc/fstab <<EOF
# <file system>     <dir>       <type>  <options>                          <dump> <pass>
UUID=$UUID_ROOTFS   /           ext4    defaults,noatime,errors=remount-ro 0      1
UUID=$UUID_BOOTFS   /boot       ext4    defaults                           0      2
UUID=$UUID_ESP      /boot/efi   vfat    umask=0077                         0      1
EOF
```

### 3. Replace LABEL with UUID in the bootfs grub.cfg

```shell
sed -i "s/root=LABEL=rootfs/root=UUID=$UUID_ROOTFS/g" bootfs/grub/grub.cfg
sed -i "s/--label bootfs/--fs-uuid $UUID_BOOTFS/g" bootfs/grub/grub.cfg
```

### 4. UEFI partition tables

Replace `partition_universal.json`, `partition_flash.json`, and `partition_4M.json` with the UEFI versions below (the `uboot` partition uses `edk2.itb`, an `ESP` partition is added, and the filesystem partitions are shifted accordingly). `partition_4M.json` targets SPI NOR flashes of 4M and above — the actual NOR on the boards is usually 8M, and the flashing tool matches the partition table downward by capacity:

```json
/* partition_universal.json — SD card / GPT general layout */
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
/* partition_4M.json — SPI NOR (mtd) layout */
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

### 5. Pack

Pack the same way as the 4.0 (K3) tar.gz variant in the [Image Creation Guide](image.md) (named after the UEFI variant): generate `md5sums.txt`, then `tar -cf - . | pigz > Bianbu-LXQt-UEFI-K3-<version>-<timestamp>.tar.gz`.

> Note: the official UEFI images ship only the `.tar.gz` (Titan firmware package), not an SD card `.img.gz`. For an SD card medium, generate it yourself with the "SDCARD Image" procedure of the [Image Creation Guide](image.md) using the UEFI `partition_universal.json` above.

## Flashing and Booting

1. Extract the `.tar.gz` and flash with the Titan tool using `partition_flash.json`;
2. Power on; the UEFI chain boots: UEFI firmware loads GRUB from the ESP, GRUB selects the DTB per board and loads the kernel.

## FAQ

- **Boot entries not updated after a kernel upgrade**: `/etc/grub.d/10_linux` must stay disabled (entries come from `09_bianbu_uefi`); after a kernel upgrade run `sudo update-grub`.
- **Device tree for the board not found**: make sure the kernel package installed `/boot/spacemit/<version>/`; the `efienv -g spacemit update product_name` command in the entry writes the product name.
- **Need an installable Live medium**: use the [ISO Image Creation Guide](iso_image.md) — the ISO is built from the UEFI image.
