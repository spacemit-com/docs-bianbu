---
sidebar_position: 4
---

# 单应用 ROOTFS 制作

本文档介绍如何在 Bianbu minimal 基础上定制一个基于 labwc 显示服务器运行单应用的 rootfs。

## 环境要求和基础 ROOTFS 制作

宿主机的环境准备和基础的 ROOTFS 制作见 [bianbu 2.1/2.2 ROOTFS制作](./bianbu_2.1_rootfs_create.md)。完成 ROOTFS 的基本配置（必要配置和通用配置）之后，即可进行下面的 Labwc 配置。

## Labwc 配置

当不需要桌面登录管理器时，可以直接启动 labwc 以提供 wayland 显示环境。具体流程如下：

 1. 安装 labwc；
 2. 配置 getty@tty1.service 在系统启动时自动登陆指定用户；
 3. 配置用户目录下 .bashrc / .zshrc 在用户登陆后启动 labwc；
 4. 配置 labwc config 文件夹下的 autostart 文件以自启动应用。

### 安装 labwc

```shell
chroot $TARGET_ROOTFS /bin/bash -c "apt-get update"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades upgrade"
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install labwc"
```

### 配置 getty@tty1.service

通过修改 getty@tty1.service 的配置，可以实现系统启动时自动登录指定用户。首先需要修改 getty@tty1.service 的 ExecStart ：

```shell
chroot $TARGET_ROOTFS /bin/bash -c "mkdir -p /usr/lib/systemd/system/getty@tty1.service.d"
chroot $TARGET_ROOTFS /bin/bash -c "cat > /usr/lib/systemd/system/getty@tty1.service.d/override.conf <<'EOF'
[Service]
ExecStart=
ExecStart=-/sbin/agetty --noclear --autologin root %I \$TERM
EOF"
```

### 配置 .bashrc 以自启动 labwc

通过修改 root 用户的shell（root用户默认为bash）配置文件，可以在用户登录后自动启动labwc。

在 /root/.bashrc 的最后添加 labwc 的启动命令，并指定labwc的配置文件夹为 /root/.config/labwc：

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "echo 'labwc -C /root/.config/labwc' >> /root/.bashrc"
```

### 配置 autostart

这里以自启动 glmark2-es2-wayland 为例。

首先安装 glmark2-es2-wayland：

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install glmark2-es2-wayland"
```

然后修改 /root/.config/labwc/autostart 文件：

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "mkdir -p /root/.config/labwc"
chroot "$TARGET_ROOTFS" /bin/bash -c "touch /root/.config/labwc/autostart"
chroot "$TARGET_ROOTFS" /bin/bash -c "printf 'glmark2-es2-wayland >/dev/null 2>&1 &\n' > /root/.config/labwc/autostart"
```

如需设置 labwc 启动时的壁纸，需将壁纸文件放在 rootfs 的系统目录下（如 /usr/share/wallpapers/wallpaper.png ），并修改 /root/.config/labwc/autostart 文件。这里以 bianbu-wallpapers deb 包提供的壁纸为例：

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get -y --allow-downgrades install bianbu-wallpapers swaybg"
chroot "$TARGET_ROOTFS" /bin/bash -c "printf 'swaybg -m fill -i /usr/share/backgrounds/SolidIsland-2.png >/dev/null 2>&1 &\n' >> /root/.config/labwc/autostart"
```

提示：完成全部包的安装后需执行如下命令清理一下缓存，以减少最终固件的大小

```shell
chroot $TARGET_ROOTFS /bin/bash -c "DEBIAN_FRONTEND=noninteractive apt-get clean"
```

### 配置 rc.xml

rc.xml 是 labwc 的核心配置文件，用于定义窗口管理器的行为、快捷键、主题、窗口规则等，具体配置见 labwc 的官网介绍：[https://github.com/labwc/labwc/blob/master/docs/rc.xml.all](https://github.com/labwc/labwc/blob/master/docs/rc.xml.all)。这里对 /root/.config/labwc/rc.xml 文件进行简单的配置以实现在打开 glmark2-es2-wayland 时自动将窗口最大化：

```shell
chroot "$TARGET_ROOTFS" /bin/bash -c "mkdir -p /root/.config/labwc"
chroot "$TARGET_ROOTFS" /bin/bash -c "cat > /root/.config/labwc/rc.xml <<'EOF'
<?xml version=\"1.0\"?>
<labwc_config>
  <windowRules>
    <windowRule identifier=\"glmark2-es2-wayland\" matchOnce=\"true\">
      <skipTaskbar>yes</skipTaskbar>
      <action name=\"ToggleMaximize\" />
      <action name=\"ToggleAlwaysOnTop\" />
    </windowRule>
  </windowRules>
</labwc_config>
EOF"
```

完成以上配置后，系统启动时即可自动登录 root 用户并启动 glmark2-es2-wayland。

## 生成分区镜像

见 [bianbu 2.1/2.2 ROOTFS制作](./bianbu_2.1_rootfs_create.md) 的对应章节。

## 制作固件

见 [固件制作指南](./image.md)。
