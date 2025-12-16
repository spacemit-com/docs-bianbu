---
sidebar_position: 10
---

# 包托管服务

## 简介

包托管服务[http://archive.spacemit.com/bianbu-community/](http://archive.spacemit.com/bianbu-community/)是Bianbu源的补充，支持社区开发者上传包并提供访问服务。

本文介绍了如何上传包到包托管服务，整个步骤可完全在开发板上完成。和debian上传方式类似。

## 前提条件

### 创建 GPG key（已有则跳过）

安装签名工具，

```bash
apt install gpg devscripts
```

生成OpenPGP密钥，注意生成不要设置密码，

```bash
gpg --gen-key
```

最终提示如下，复制密钥 ID(pub 部分)，之后导出公钥时要用到，

```bash
gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

注意：使用 “gpg --full-generate-key” 以获得一个全功能的密钥生成对话框。

GnuPG 需要构建用户标识以辨认您的密钥。

真实姓名： test-user
电子邮件地址： <test@spacemit.com>
您选定了此用户标识：
    “test-user <test@spacemit.com>”

更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？ O
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。
我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
发生器有更好的机会获得足够的熵。

# 这里会提示输入密码，可设可不设

# 如通过ssh在板子上执行此命令，则可能会在桌面弹窗提示输入密码

gpg: 密钥 42D4AEAF977345E3 被标记为绝对信任
gpg: 吊销证书已被存储为‘/home/qiu/.gnupg/openpgp-revocs.d/9F9C52A13E434AB3DC45693A42D4AEAF977345E3.rev’
公钥和私钥已经生成并被签名。

pub   rsa3072 2024-11-04 [SC] [有效至：2026-11-04]
      9F9C52A13E434AB3DC45693A42D4AEAF977345E3
uid                      test-user <test@spacemit.com>
sub   rsa3072 2024-11-04 [E] [有效至：2026-11-04]
```

### 配置上传地址

添加如下条目到 `~/.dput.cf`

```bash
[bianbu]
fqdn                    = ftp.upload.spacemit.com
incoming                = /UploadQueue/
login                   = anonymous
allow_dcut              = 1
method                  = ftp
allowed_distributions   = (experimental)
```

### 申请上传权限

导出签名公钥

根据前面提示的公钥id导出公钥

```bash
gpg -a --export 9F9C52A13E434AB3DC45693A42D4AEAF977345E3
```

点击 [https://developer.spacemit.com/auth/#/user/userInfo](https://developer.spacemit.com/auth/#/user/userInfo)，按照图示步骤添加GPG公钥

![图片1：GPG 公钥导入示例](./static/gpg_1.png)

注意粘贴内容包含开头的BEGIN和结尾的END行

![图片2：GPG 公钥导入示例](./static/gpg_2.png)

## 准备源码包和deb包

这里以hello包作为示例，首先下载源码包

```bash
apt source hello
```

切换到项目目录，

```bash
cd ~/hello-2.10
```

构建，

```bash
dpkg-buildpackage -sa -uc -us -etest@spacemit.com -mtest@spacemit.com
```

- -m：本次发布的维护者，会记录在changes文件的Maintainer字段。
- -e：本次构建的维护者，会记录在changes文件的Changed-By字段。
- -sa：changes文件总是记录orig压缩包。
简单起见，可以填写为签名者（刚才创建的GPG key用户）的邮箱。

## 检查 changes 文件

1. 执行如下命令确保../xx.changes的Distribution字段为experimental，以便后续能入库到正确suite。

```bash
sed -i 's/^Distribution: .*/Distribution: experimental/' ../hello_2.10-3build1_riscv64.changes
```

2. 确保../xx.changes的Maintainer和Changed-By字段是一个正确的邮箱格式，否则无法入库

```bash
Maintainer: xxx@yyy
Changed-By: xxx@yyy
```

3. 注意changes文件中一般需要同时包含源码包和deb包（binNMU除外，见FAQ1）。

## 签名

使用devscripts包中的debsign命令对changes文件签名，

```bash
debsign ../hello_2.10-3build1_riscv64.changes -exxx@yyy
```

- -e：前面生成OpenPGP密钥时填的邮箱

## 上传

dput命令上传，

```bash
dput bianbu ../hello_2.10-3build1_riscv64.changes
```

之后会陆续收到邮件通知上传状态。

第一次上传，

| 顺序 | 邮件标题                                   | 发送时间                     | 说明                                 |
|------|--------------------------------------------|------------------------------|--------------------------------------|
| 1    | Processing of xxx.changes                  | 上传后5分钟内                | 通知包是否成功上传到 FTP 服务器      |
| 2    | xxx.changes is NEW                         | 每小时的第 2、17、32、47 分  | 进入 new 队列，等待仓库管理员人工审核 |
| 3    | xxx.changes ACCEPTED into experimental     | 每小时的第 52 分             | 审核通过，且已入库                   |

后续上传，

| 顺序 | 邮件标题                                 | 发送时间                     | 说明                                 |
|------|------------------------------------------|------------------------------|--------------------------------------|
| 1    | Processing of xxx.changes                | 上传后5分钟内                | 通知包是否成功上传到 FTP 服务器      |
| 2    | xxx.changes ACCEPTED into experimental   | 每小时的第 2、17、32、47 分  | 审核通过，每小时第 52 分入库一次     |

## 安装

确保 /etc/apt/sources.list.d/bianbu.sources配置了experimental suite，

```bash
原有内容...

# experimental suite

Types: deb
URIs: http://archive.spacemit.com/bianbu-community/
Suites: experimental
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/bianbu-archive-keyring.gpg
```

更新索引列表

```bash
apt update
```

安装

```bash
apt install xxx
```

## FAQ

### Q1. 如何使用 binNMU（Binary Non-Maintainer Upload，二进制非维护者上传）？

前提条件：已经上传入库过一次源码包。

后续上传可不上传源码包，只上传deb包。

只需在源码目录下，执行debchange命令在debian/changelog中生成bin-nmu条目

```bash
debchange --force-bad-version --bin-nmu 'description'
```

之后正常编译，签名，上传。
