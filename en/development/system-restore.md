---
sidebar_position: 9
---

# read-only-rootfs-config Usage Instructions

## Introduction

`read-only-rootfs-config` is a tool designed to switch the system root filesystem to read-only mode. When the read-only root filesystem is enabled, all modifications will be preserved but not stored on the root filesystem. Once the read-only root filesystem is disabled, all modifications will be deleted, and the system will revert to its previous state. The tool currently supports eMMC, SD card, and SSD storage devices.

## How to Use (Please Read the Precautions Before Use)

### 1. Pre-installation

This package can be pre-installed in the image but **will not automatically perform** any partition or read-only operations.

### 2. Manually Enable Read-Only Root Filesystem

1. Run as root user:

   ```sh
   sudo read-only-rootfs-config
   ```

2. The script will display the following warning:

   ```
   WARNING: !!!Enabling the read-only root filesystem will resize the root filesystem partition, Please operate with caution!!!
   Do you want to set the root directory to read-only? [Y/n]:
   ```

3. Type `yes` or `y`, the script will enable the read-only root filesystem. If it's already in read-only mode, it will exit.

**Note:** After enabling the read-only root filesystem on eMMC and SSD media, during reboot, you must manually type `yes` in the serial console to confirm the partition resize operation.

4. Type `no` or `n` to revert to a writable root filesystem. If it's already writable, the script will exit.

## Precautions

* **Be sure to back up important data in advance!** Partitioning and filesystem operations have risks.
* If there is insufficient disk space, the script will automatically exit and prompt.
* Do not power off during the process. Each time you enable or disable read-only or writable mode, a reboot is required to take effect. **Ensure that you reboot normally—do not turn off the power during reboot, or the filesystem may encounter issues.**

## Testing Method

You can use software installation/uninstallation or large files for testing:

* Test Steps:
  (1) First, without enabling the read-only root filesystem, boot into the system normally and install the glmark2-es2-wayland software.

  ```shell
  sudo apt-get install -y glmark2-es2-wayland
  ```

  After installation, place a large file A in the `/home/bianbu` directory (assuming the username is `bianbu`).

(2) Second, enable the read-only root filesystem as per the usage instructions. After rebooting, uninstall the glmark2-es2-wayland software.

```shell
sudo apt purge -y glmark2-es2-wayland
```

At this point, the glmark software should no longer exist. Next, delete file A from the `/home/bianbu` directory and place a large file B there.

You can use the `df -h -T` command to check if the operation was successful:

```shell
root@spacemit-k1-x-MUSE-Pi-Pro-board:~# df -h -T
Filesystem     Type    Size  Used  Avail Use% Mounted on
tmpfs          tmpfs   786M  1.9M  784M   1% /run
/dev/mmcblk0p6 ext4   8.9G  6.5G  1.9G  78% /media/root-ro
/dev/mmcblk0p7 ext4   20G   13M   19G   1% /media/root-rw
overlayroot    overlay 20G   13M   19G   1% /
tmpfs          tmpfs   3.9G  20K  3.9G   1% /dev/shm
tmpfs          tmpfs   5.0M  12K  5.0M   1% /run/lock
tmpfs          tmpfs   3.9G  8.0K  3.9G   1% /tmp
tmpfs          tmpfs   1.0M   0   1.0M   0% /run/credentials/systemd-journald.service
tmpfs          tmpfs   1.0M   0   1.0M   0% /run/credentials/systemd-resolved.service
/dev/mmcblk0p5 ext4   224M  55M  152M   27% /boot
tmpfs          tmpfs   786M  112K  785M   1% /run/user/117
tmpfs          tmpfs   1.0M   0   1.0M   0% /run/credentials/serial-getty@ttyS0.service
tmpfs          tmpfs   786M  96K  786M   1% /run/user/0
```

For SD card, you should see `/dev/mmcblk0p7`; for eMMC, it should be `/dev/mmcblk2p7`; for SSD, it should be `/dev/nvme0n1p7`, and successful mounting indicates that the setting was successful.

(3) Third, revert the root filesystem to writable as per the usage instructions, reboot, and enter the system.

* Expected Results:

The expected result is that glmark2-es2-wayland should work normally, file A should remain in `/home/bianbu`, and file B should not exist.

## Common Issues

### Q: Why does `df -h` show less space than expected after resizing the partition?

A: This is due to partition alignment, filesystem metadata, and ext4 reserved space, which is normal.

### Q: How can I revert to a writable root filesystem?

A: Run `read-only-rootfs-config` again and choose `no` to disable overlayroot.

---

For further questions, please contact the maintainer or refer to the source code.

