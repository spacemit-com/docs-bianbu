---
sidebar_position: 9
---

# read-only-rootfs-config 使用说明

## 简介

`read-only-rootfs-config` 是一个用于将系统根文件系统切换为只读模式，当设置只读根文件系统后，所有的修改会被保留但不会被保留在根文件系统上，当取消只读根文件系统后，所有的修改将被删除，系统将恢复至设置前的状态。工具目前支持eMMC、SD卡、SSD三种存储介质

## 使用方法（使用前请阅读注意事项）

### 1. 预装

本软件包可预装在镜像中，**不会自动执行**任何分区或只读操作。

### 2. 手动执行配置只读根文件系统

1. 以 root 用户运行：
   
   ```sh
   sudo read-only-rootfs-config
   ```

2. 脚本运行会显示如下警告：
   
   ```
   WARNING: !!!Enabling the read-only root filesystem will resize the root filesystem partition, Please operate with caution!!!
   Do you want to set the root directory to read-only? [Y/n]:
   ```

3. 输入 `yes` 或 `y`，脚本将使能只读根文件系统，如果已经是只读根文件系统的话会退出

**注意：eMMC和SSD介质每次使能只读根文件系统后在重启过程中，必须在串口手动输入yes，确认resize分区操作**

4. 输入 `no` 或 `n`，则重新设置为可写根文件系统，如果当前已经是可写读根文件系统的话会退出

## 注意事项

- **请务必提前备份重要数据！** 分区和文件系统操作有风险。
- 若磁盘空间不足，脚本会自动退出并提示。
- 设置过程中不要断电，每次设置只读或者可写，必须重启后生效，**注意重启时必须正常reboot重启，不要断电重启，否则文件系统可能出问题**

## 测试方法

可用安装/卸载软件、大文件来做测试：

* 测试步骤
  (1) 第一步：在没有使能只读根文件系统之前，正常进入系统，并安装glmark2-es2-wayland软件
  
  ```shell
  sudo apt-get install -y glmark2-es2-wayland
  ```
  
  安装完成后，在/home/bianbu(假设这里用户名是bianbu)，目录下放一个大文件A

(2) 第二步：按照使用方法，使能只读根文件系统，重启进入系统之后，卸载glmark2-es2-wayland软件

```shell
sudo apt purge -y glmark2-es2-wayland
```

此时glmark软件应该不存在了，接着删除/home/bianbu目录下的文件A，并在/home/bianbu目录下放一个大文件B。

可用`df -h -T`指令查看是否成功
  
  ```shell
  root@spacemit-k1-x-MUSE-Pi-Pro-board:~# df -h -T
文件系统       类型     大小  已用  可用 已用% 挂载点
tmpfs          tmpfs    786M  1.9M  784M    1% /run
/dev/mmcblk0p6 ext4     8.9G  6.5G  1.9G   78% /media/root-ro
/dev/mmcblk0p7 ext4      20G   13M   19G    1% /media/root-rw
overlayroot    overlay   20G   13M   19G    1% /
tmpfs          tmpfs    3.9G   20K  3.9G    1% /dev/shm
tmpfs          tmpfs    5.0M   12K  5.0M    1% /run/lock
tmpfs          tmpfs    3.9G  8.0K  3.9G    1% /tmp
tmpfs          tmpfs    1.0M     0  1.0M    0% /run/credentials/systemd-journald.service
tmpfs          tmpfs    1.0M     0  1.0M    0% /run/credentials/systemd-resolved.service
/dev/mmcblk0p5 ext4     224M   55M  152M   27% /boot
tmpfs          tmpfs    786M  112K  785M    1% /run/user/117
tmpfs          tmpfs    1.0M     0  1.0M    0% /run/credentials/serial-getty@ttyS0.service
tmpfs          tmpfs    786M   96K  786M    1% /run/user/0

  ```

SD卡看到/dev/mmcblk0p7，eMMC能看到/dev/mmcblk2p7，SSD能看到/dev/nvme0n1p7，并且成功挂载就算设置成功

(3)第三步：按照使用方法，恢复根文件系统为可写，重启并进入系统

* 测试结果

期望结果为glmark2-es2-wayland可以正常运行，/home/bianbu目录下文件A仍然存在，文件B不存在。

## 常见问题

### Q: 为什么分区扩容后 `df -h` 显示的空间比预期小？

A: 这是因为分区对齐、文件系统元数据和 ext4 的保留空间等原因导致的，属于正常现象。

### Q: 如何恢复为可写根文件系统？

A: 重新运行 `read-only-rootfs-config`，选择 `no`，即可关闭 overlayroot。

---

如有更多问题，请联系维护者或查阅源码。
