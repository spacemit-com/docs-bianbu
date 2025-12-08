---
sidebar_position: 2
---

# 软件包管理

## 概述

- **软件包**
Bianbu 软件包遵循 Debian 软件包规范，使用 `apt` 工具进行管理。

- **软件源**
Bianbu 提供以下官方 APT 软件包源：
  - **Bianbu 1.0**（已停止维护）: [http://archive.spacemit.com/bianbu-ports/](http://archive.spacemit.com/bianbu-ports/)
  - **Bianbu 2.0 & 3.0** : [http://archive.spacemit.com/bianbu/](http://archive.spacemit.com/bianbu/)

## 更新软件源

在终端，使用以下命令刷新本地软件包索引信息：
```shell
sudo apt update
```

## 安装或更新软件包

在终端，使用以下命令安装或升级指定软件包，例如安装 `hello`：

```shell
sudo apt install hello
```

## 卸载软件包

在终端，使用以下命令卸载软件包（保留配置文件），例如卸载 `hello`：

```shell
sudo apt  remove hello
```

如需连同配置文件一并删除，请在终端执行：

```shell
sudo apt purge hello
```
