---
sidebar_position: 4
---

# ROS2使用指南

Bianbu OS 支持了 **ROS2 Humble** 和 **ROS2 Jazzy** 两个版本的 deb 包。目前 ROS2 Humble 的支持较为完善，我们当前主要维护 Humble 版本，因此建议您优先安装和使用 ROS2 Humble。

**注意:** 不要同时安装 Humble 和 Jazzy 到您的系统，以免引起不必要的问题。

## Humble deb 包安装

### 支持的固件版本

- 固件列表：[Bianbu固件](https://archive.spacemit.com/image/k1/version/bianbu/)
- Desktop 或者 minimal 固件版本大于 2.0.4 均可
- 建议使用 V2.1 以上的 Desktop 版本

### 环境准备

#### 设置语言环境

确保您有一个支持UTF-8的区域设置

```shell
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

#### 启用所需的存储库

您需要将 ROS2 apt 存储库添加到您的系统中。

```shell
grep -q '^Suites:.*\bnoble-ros\b' /etc/apt/sources.list.d/bianbu.sources || sudo sed -i '0,/^Suites:/s//& noble-ros/' /etc/apt/sources.list.d/bianbu.sources
```

设置源

```shell
if ! dpkg -s bianbu-desktop-lite >/dev/null 2>&1; then
  echo "bianbu-desktop-lite not installed, proceeding..."

  if [ ! -f /etc/apt/preferences.d/noble-ros.pref ]; then
    sudo tee /etc/apt/preferences.d/noble-ros.pref > /dev/null <<EOF
Package: src:opencv
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50

Package: src:qtbase-opensource-src
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50

Package: src:qtbase-opensource-src-gles
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50

Package: src:pyqt5
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50
EOF
  else
    echo "/etc/apt/preferences.d/noble-ros.pref already exists, skipping..."
  fi

else
  echo "bianbu-desktop-lite is already installed, skipping preference setup."
fi
```

#### 安装开发工具（推荐）

如果您要构建ROS包或以其他方式进行开发，您还可以安装开发工具（推荐）：

```shell
sudo apt update && sudo apt install ros-dev-tools
```

### 安装 ROS2

设置存储库后更新 apt 存储库缓存

```shell
sudo apt update
```

ROS2 软件包构建在经常更新的 Bianbu 系统上。始终建议您在安装新软件包之前确保系统是最新的

```shell
sudo apt upgrade
```

桌面安装（推荐），包含：ROS、RViz、demo、教程包等

```shell
sudo apt install ros-humble-desktop
```

ROS-Base 安装，包含：通信库、消息包、命令行工具。没有 GUI 工具

```shell
sudo apt install ros-humble-ros-base
```

安装额外的 RMW 实现（可选）

ROS2 使用的默认中间件是Fast DDS ，但可以在运行时替换中间件（RMW）。请参阅有关如何使用多个 RMW 的[指南](https://docs.ros.org/en/humble/How-To-Guides/Working-with-multiple-RMW-implementations.html)

### 获取 ROS2 环境

通过 source 以下文件来设置您的 ROS2 环境

```shell
source /opt/ros/humble/setup.zsh
```

如果您使用 bash ，请将 setup.zsh 替换成 setup.bash

### 尝试一些例子

使用 echo $0 确定您使用的是zsh还是bash，本示例中使用的是 zsh。

如果您使用的是 bash，后续示例中的所有 zsh 请替换成 bash ，否则可能导致一些运行时错误。

```shell
echo $0
-zsh # 这一行是输出，请不要执行
```

#### 例子1: 基本话题通信

在任意位置打开一个终端，使用 source 命令更新 ROS2 的环境变量，然后运行 ​​C++ talker ：

```shell
source /opt/ros/humble/setup.zsh
ros2 run demo_nodes_cpp talker
```

在另一个终端中使用 source 命令更新 ROS2 的环境变量，然后运行 ​​Python listener ：

```shell
source /opt/ros/humble/setup.zsh
ros2 run demo_nodes_py listener
```

您应该可以看到 talker 说它正在 Publishing 消息，而 listener 说 I heard 这些消息。

这验证了 C++ 和 Python API 是否正常工作。万岁！

**提示**

> 当如果当前终端已经执行：source /opt/ros/humble/setup.zsh，则不必重复执行

#### 例子2: 小海龟

如果您新打开了一个终端，不要忘记执行：source /opt/ros/humble/setup.zsh

本示例请在桌面中启动终端运行，使用 ssh 连接的终端无法拉起 turtlesim 的界面

要启动turtlesim，请在终端中输入以下命令：

```shell
ros2 run turtlesim turtlesim_node
```

您应该可以看到模拟器窗口弹出，中间有一只随机的海龟

![alt text](static/ROS2-turtlesim.png)

在终端中的命令下，您将看到来自节点的消息：

```shell
[INFO] [1726820259.299762059] [turtlesim]: Starting turtlesim with node name /turtlesim
[INFO] [1726820259.366410375] [turtlesim]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
```

在另一个终端运行一个新节点来控制第一个节点中的海龟：

```shell
ros2 run turtlesim turtle_teleop_key
```

使用键盘上的方向键来控制乌龟。它会在屏幕上移动，用它附带的“笔”画出它到目前为止所经过的路径。

### 小结 (Humble 版本)
现在，您可以继续学习[官方教程和演示](https://docs.ros.org/en/humble/Tutorials.html)来配置您的环境，创建您自己的工作区和包，并学习 ROS2 核心概念。

如果在开发过程中需要其他功能包，可以使用sudo apt install ros-humble-package_name 来进行安装，这样，可以避免未使用到的功能包占用系统的存储空间。

## Jazzy deb 包安装

### 支持的固件版本

固件列表：[Bianbu固件](https://archive.spacemit.com/image/k1/version/bianbu/)
Desktop 或者 minimal 固件版本大于 2.0.4 均可
建议使用 v2.1 以上的 Desktop 版本

### 环境准备

#### 设置语言环境

确保您有一个支持UTF-8的区域设置

```shell
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings
```

#### 启用所需的存储库

您需要将 ROS2 apt 存储库添加到您的系统中。

```shell
grep -q '^Suites:.*\bnoble-ros\b' /etc/apt/sources.list.d/bianbu.sources || sudo sed -i '0,/^Suites:/s//& noble-ros/' /etc/apt/sources.list.d/bianbu.sources
```

设置源

```shell
if ! dpkg -s bianbu-desktop-lite >/dev/null 2>&1; then
  echo "bianbu-desktop-lite not installed, proceeding..."

  if [ ! -f /etc/apt/preferences.d/noble-ros.pref ]; then
    sudo tee /etc/apt/preferences.d/noble-ros.pref > /dev/null <<EOF
Package: src:opencv
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50

Package: src:qtbase-opensource-src
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50

Package: src:qtbase-opensource-src-gles
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50

Package: src:pyqt5
Pin: release o=Spacemit, n=noble-ros
Pin-Priority: 50
EOF
  else
    echo "/etc/apt/preferences.d/noble-ros.pref already exists, skipping..."
  fi

else
  echo "bianbu-desktop-lite is already installed, skipping preference setup."
fi
```

#### 安装开发工具（可选）

如果您要构建ROS包或以其他方式进行开发，您还可以安装开发工具：

```shell
sudo apt update && sudo apt install ros-dev-tools
```

### 安装 ROS2

设置存储库后更新 apt 存储库缓存

```shell
sudo apt update
```

ROS2 软件包构建在经常更新的 Bianbu 系统上。始终建议您在安装新软件包之前确保系统是最新的

```shell
sudo apt upgrade
```

桌面安装（推荐），包含：ROS、RViz、demo、教程包等

```shell
sudo apt install ros-jazzy-desktop
```

ROS-Base 安装，包含：通信库、消息包、命令行工具。没有 GUI 工具

```shell
sudo apt install ros-jazzy-ros-base
```

安装额外的 RMW 实现（可选）

ROS2 使用的默认中间件是Fast DDS ，但可以在运行时替换中间件（RMW）。请参阅有关如何使用多个 RMW 的[指南](https://docs.ros.org/en/jazzy/How-To-Guides/Working-with-multiple-RMW-implementations.html)

### 设置环境

通过 source 以下文件来设置您的 ROS2 环境

```shell
source /opt/ros/jazzy/setup.zsh
```

如果您使用 bash ，请将 setup.zsh 替换成 setup.bash

### 尝试一些例子

如果您在上面安装了ros-jazzy-desktop ，您可以尝试一些示例

在一个终端中，source /opt/ros/jazzy/setup.zsh ，然后运行 ​​C++ talker

```shell
source /opt/ros/jazzy/setup.zsh
ros2 run demo_nodes_cpp talker
```

在另一个终端中，source /opt/ros/jazzy/setup.zsh，然后运行 ​​Python listener ：

```shell
source /opt/ros/jazzy/setup.zsh
ros2 run demo_nodes_py listener
```

您应该看到talker说它正在Publishing消息，而listener说I heard这些消息。这验证了 C++ 和 Python API 是否正常工作。

### 后续步骤

现在，您可以继续学习[官方教程和演示](https://docs.ros.org/en/jazzy/Tutorials.html)来配置您的环境，创建您自己的工作区和包，并学习 ROS2 核心概念。
