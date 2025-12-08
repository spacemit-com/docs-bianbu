---
sidebar_position: 4
---

# ROS使用指南

目前我们仅提供ROS Noetic的Prebuilt包。

## 使用ROS Prebuilt包

我们支持的 ROS Noetic 是在 Bianbu 2.0 上构建的，因此，为了避免不必要的环境问题，请您使用 Bianbu 2.0 来开发和使用。prebuilt包含了 ros-noetic-desktop-full 中的所有包。

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

#### 安装先决条件

* 安装 ROS Noetic 的依赖库

```shell
sudo apt-get install -y \
    python-is-python3 \
    cmake \
    python3-mock \
    python3-nose \
    google-mock \
    libgtest-dev \
    python3-catkin-pkg \
    python3-empy \
    libboost-dev \
    python3-pygraphviz \
    python3-pydot \
    qtbase5-dev \
    python3-rospkg \
    libconsole-bridge-dev \
    python3-defusedxml \
    libapr1-dev \
    libaprutil1-dev \
    libboost-regex-dev \
    liblog4cxx-dev \
    libboost-system-dev \
    libboost-thread-dev \
    libbz2-dev \
    libboost-filesystem-dev \
    libgpgme-dev \
    libssl-dev \
    libopencv-dev \
    libboost-all-dev \
    python3-opencv \
    uuid-dev \
    graphviz \
    libpyside2-dev \
    libshiboken2-dev \
    pyqt5-dev \
    python3-pyqt5 \
    python3-pyqt5.qtsvg \
    python3-pyside2.qtsvg \
    python3-sip-dev \
    shiboken2 \
    qt5-qmake \
    liburdfdom-headers-dev \
    liborocos-kdl-dev \
    libtinyxml-dev \
    libtinyxml2-dev \
    python3-coverage \
    libboost-program-options-dev \
    libboost-chrono-dev \
    libpoco-dev \
    libboost-date-time-dev \
    python3-gnupg \
    libeigen3-dev \
    liblz4-dev \
    liborocos-kdl1.5 \
    python3-matplotlib \
    python3-opengl \
    python3-pykdl \
    python3-pyqt5.qtopengl \
    liburdfdom-dev \
    libogre-1.9-dev \
    libassimp-dev \
    libyaml-cpp-dev \
    libgl1-mesa-dev \
    libglu1-mesa-dev \
    libqt5opengl5-dev \
    lm-sensors \
    libcurl4-openssl-dev \
    tango-icon-theme \
    libpcl-dev \
    libbullet-dev \
    libsdl1.2-dev \
    libsdl-image1.2-dev
```

* 安装系统Python的依赖库

```shell
sudo apt-get install -y \
    python3-argcomplete \
    python3-babeltrace \
    python3-catkin-pkg \
    python3-empy \
    python3-flake8 \
    python3-flake8-builtins \
    python3-flake8-comprehensions \
    python3-flake8-docstrings \
    python3-flake8-import-order \
    python3-flake8-quotes \
    python3-jsonschema \
    python3-lark \
    python3-matplotlib \
    python3-mypy \
    python3-psutil \
    python3-pycodestyle \
    python3-pydot \
    python3-pygraphviz \
    python3-pykdl \
    python3-pyqt5 \
    python3-pyqt5.qtsvg \
    python3-pyside2.qtsvg \
    python3-pytest \
    python3-pytest-cov \
    python3-pytest-mock \
    python3-pytest-timeout \
    python3-sip-dev
```

如果您想使用rviz，请设置环境变量：

```shell
export QT_QPA_PLATFORM=xcb
```

### 下载prebuilt包

* 进入[发布页面](https://archive.spacemit.com/ros/prebuilt)

* 下载 Bianbu OS ROS 的最新软件包，在本示例中，它下载位于：~/ros-noetic-desktop-full-linux-riscv64-20240930.tar.gz

**提示**

> 随着版本的迭代，可能有多个prebuilt包的下载选项，这可能会导致文件名不同。

### 安装prebuilt包

* 解压软件包

```shell
sudo mkdir -p /opt/ros/noetic
cd /opt/ros/noetic
sudo tar -xzvf ~/ros-noetic-desktop-full-linux-riscv64-20240930.tar.gz
```

这会将prebuilt包的文件安装到 /opt/ros/noetic 。建议安装到 /opt/ros/noetic 路径下，使用其他路径会出现一些问题。


解压完成后的文件夹应该如下所示：

```shell
➜  ~ ls /opt/ros/noetic
bin  env.sh  etc  include  lib  local_setup.bash  local_setup.sh  local_setup.zsh  setup.bash  setup.sh  _setup_util.py  setup.zsh  share
```

## 尝试一些例子

使用 echo $0 确定您使用的是zsh还是bash，本示例中使用的是 zsh。

如果您使用的是 bash，后续示例中的所有 zsh 请替换成 bash ，否则可能导致一些运行时错误。

```shell
echo $0
-zsh # 这一行是输出，请不要执行
```

当您在任意位置打开一个终端时，请使用 source /opt/ros/noetic/setup.zsh 命令更新 ROS 的环境变量。或者将其追加到 ~/.zshrc 文件末尾。

```shell
echo "source /opt/ros/noetic/setup.zsh" >> ~/.zshrc
```

以下示例默认执行了该语句。

### 小海龟

本示例请在桌面中启动终端运行，使用 ssh 连接的终端无法拉起 turtlesim 的界面

要启动turtlesim，请在终端中输入以下命令：

```shell
rosrun turtlesim turtlesim_node
```

您应该可以看到模拟器窗口弹出，中间有一只随机的海龟

![alt text](static/ROS-turtlesim.png)

在终端中的命令下，您将看到来自节点的消息：

```shell
QSocketNotifier: Can only be used with threads started with QThread
[INFO] [1727264504.878540498]: Starting turtlesim with node name /turtlesim
[INFO] [1727264504.918640500]: Spawning turtle [turtle1] at x=[5.544445], y=[5.544445], theta=[0.000000]
```

在另一个终端运行一个新节点来控制第一个节点中的海龟：

```shell
rosrun turtlesim turtle_teleop_key
```

使用键盘上的方向键来控制乌龟。它会在屏幕上移动，用它附带的“笔”画出它到目前为止所经过的路径。

### 编译自己的包--基本话题通信

#### 1、创建并初始化自己的工作空间

```shell
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/src
catkin_init_workspace
```

您应该可以看到类似如下输出：

```shell
Creating symlink "/home/zq-pi2/catkin_ws/src/CMakeLists.txt" pointing to "/opt/ros/noetic/share/catkin/cmake/toplevel.cmake"
```

执行一次 catkin_make 以验证：

```shell
cd ~/catkin_ws
catkin_make
```

完成后，~/catkin_ws 目录的内容如下：

```shell
➜  catkin_ws ls
build  devel  src
```

#### 2、创建一个示例包

```shell
cd ~/catkin_ws/src
catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
```

std_msgs rospy roscpp 是 beginner_tutorials 依赖的包。

这将在 src 下创建一个名为 beginner_tutorials 的标准 ROS 包，其初始包含内容如下：

```shell
➜  beginner_tutorials ls
CMakeLists.txt  include  package.xml  src
```

关于包的更多信息，请参考[创建ROS包](https://wiki.ros.org/ROS/Tutorials/CreatingPackage)

#### 3、构建包

首先确保环境变量已经被设置：

```shell
source /opt/ros/noetic/setup.zsh
```

使用 catkin_make 构建包:

```shell
cd ~/catkin_ws
catkin_make
```

您应该看到 cmake 和 make 的大量输出：

```shell
➜  catkin_ws catkin_make
Base path: /home/zq-pi2/catkin_ws
Source space: /home/zq-pi2/catkin_ws/src
Build space: /home/zq-pi2/catkin_ws/build
Devel space: /home/zq-pi2/catkin_ws/devel
Install space: /home/zq-pi2/catkin_ws/install
####
#### Running command: "cmake /home/zq-pi2/catkin_ws/src -DCATKIN_DEVEL_PREFIX=/home/zq-pi2/catkin_ws/devel -DCMAKE_INSTALL_PREFIX=/home/zq-pi2/catkin_ws/install -G Unix Makefiles" in "/home/zq-pi2/catkin_ws/build"
####
CMake Deprecation Warning at CMakeLists.txt:4 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Using CATKIN_DEVEL_PREFIX: /home/zq-pi2/catkin_ws/devel
-- Using CMAKE_PREFIX_PATH: /opt/ros/noetic
-- This workspace overlays: /opt/ros/noetic
CMake Warning (dev) at /opt/ros/noetic/share/catkin/cmake/python.cmake:4 (find_package):
  Policy CMP0148 is not set: The FindPythonInterp and FindPythonLibs modules
  are removed.  Run "cmake --help-policy CMP0148" for policy details.  Use
  the cmake_policy command to set the policy and suppress this warning.

Call Stack (most recent call first):
  /opt/ros/noetic/share/catkin/cmake/all.cmake:164 (include)
  /opt/ros/noetic/share/catkin/cmake/catkinConfig.cmake:20 (include)
  CMakeLists.txt:58 (find_package)
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Using PYTHON_EXECUTABLE: /usr/bin/python3
-- Using Debian Python package layout
-- Using empy: /usr/bin/empy
-- Using CATKIN_ENABLE_TESTING: ON
-- Call enable_testing()
-- Using CATKIN_TEST_RESULTS_DIR: /home/zq-pi2/catkin_ws/build/test_results
-- gtest not found, C++ tests can not be built. Please install the gtest headers globally in your system or checkout gtest (by running 'git clone  https://github.com/google/googletest.git -b release-1.8.0' in the source space '/home/zq-pi2/catkin_ws/src' of your workspace) to enable gtests
-- Using Python nosetests: /usr/bin/nosetests3
-- catkin 0.8.10
-- BUILD_SHARED_LIBS is on
-- BUILD_SHARED_LIBS is on
-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
-- ~~  traversing 1 packages in topological order:
-- ~~  - beginner_tutorials
-- ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
-- +++ processing catkin package: 'beginner_tutorials'
-- ==> add_subdirectory(beginner_tutorials)
CMake Deprecation Warning at beginner_tutorials/CMakeLists.txt:1 (cmake_minimum_required):
  Compatibility with CMake < 3.5 will be removed from a future version of
  CMake.

  Update the VERSION argument <min> value or use a ...<max> suffix to tell
  CMake that the project does not need compatibility with older versions.


-- Configuring done (10.0s)
-- Generating done (0.1s)
-- Build files have been written to: /home/zq-pi2/catkin_ws/build
####
#### Running command: "make -j8 -l8" in "/home/zq-pi2/catkin_ws/build"
####
```

如果您想了解更多的包构建知识，请参考[ROS 包构建](https://wiki.ros.org/ROS/Tutorials/BuildingPackages)

#### 4、编写一个 Python 的发布者

```shell
cd ~/catkin_ws/src/beginner_tutorials
mkdir scripts
cd scripts
```

```shell
vim talker.py
```

粘贴如下内容：

```shell
#!/usr/bin/env python3

import rospy  # 导入 ROS Python 客户端库
from std_msgs.msg import String  # 从标准消息库导入 String 消息类型

def talker():  # 定义 talker 函数
    pub = rospy.Publisher('chatter', String, queue_size=10)  # 创建一个发布者，发布到话题 'chatter'，消息类型为 String，队列大小为 10
    rospy.init_node('talker', anonymous=True)  # 初始化 ROS 节点，节点名称为 'talker'，anonymous=True 表示 ROS 会自动为节点名添加唯一后缀
    rate = rospy.Rate(10) # 10hz  # 设置发布频率为 10 Hz，即每秒发布 10 次
    while not rospy.is_shutdown():  # 当 ROS 节点未关闭时持续执行
        hello_str = f"Publishing hello world ! {rospy.get_time()}"  # 创建一个字符串，包含当前时间
        rospy.loginfo(hello_str)  # 打印日志信息到控制台，信息内容为 hello_str
        pub.publish(hello_str)  # 发布 hello_str 消息到 'chatter' 话题
        rate.sleep()  # 根据设定的频率暂停，使得发布频率为 10 Hz

if __name__ == '__main__':
    try:
        talker()  # 调用 talker 函数
    except rospy.ROSInterruptException:  # 捕获 ROS 中断异常
        pass  # 如果发生中断异常，直接忽略并退出

```

对于使用 Python 编写的节点，请赋予可执行权限，否则可能出现 rosrun 找不到可执行程序的问题。

```shell
chmod +x talker.py
```

#### 5、编写一个 Python 的订阅者

```shell
cd ~/catkin_ws/src/beginner_tutorials
mkdir scripts
cd scripts
```

```shell
vim listener.py
```

粘贴如下内容：

```shell
#!/usr/bin/env python3
import rospy  # 导入 ROS 的 Python 客户端库
from std_msgs.msg import String  # 从 std_msgs 消息库中导入 String 消息类型

def callback(data):  # 定义一个回调函数，用于处理接收到的消息
    rospy.loginfo(f"{rospy.get_caller_id()} I heard {data.data}")  # 使用 f-string 打印日志信息，包含节点 ID 和接收到的消息内容

def listener():  # 定义 listener 函数，设置节点并订阅话题

    # 在 ROS 中，节点有唯一的名称。如果有两个节点使用相同的名字启动，前一个节点会被踢下线。
    # anonymous=True 表示 rospy 将为该节点选择一个唯一的名字，这样可以让多个 listener 节点同时运行。
    rospy.init_node('listener', anonymous=True)  # 初始化名为 'listener' 的节点，anonymous=True 确保其名称唯一

    rospy.Subscriber("chatter", String, callback)  # 订阅 'chatter' 话题，消息类型为 String，并调用 callback 函数处理消息

    # spin() 函数可以保持 Python 程序不退出，直到该节点停止运行
    rospy.spin()  # 进入循环等待，直到节点被关闭

if __name__ == '__main__':
    listener()  # 调用 listener 函数，启动监听器
```

```shell
chmod +x listener.py
```

#### 6、构建编写好的节点

```shell
cd ~/catkin_ws
catkin_make
```

#### 7、演示发布者和订阅者

首先确保 roscre 已经被运行。以下示例默认已经执行了 source /opt/ros/noetic/setup.zsh 来加载 ROS 环境。

打开一个终端，执行：

```shell
source ~/catkin_ws/devel/setup.zsh
rosrun beginner_tutorials talker.py
```

打开另一个终端，执行：

```shell
source ~/catkin_ws/devel/setup.zsh
rosrun beginner_tutorials listener.py
```

您应该可以看到 talker 说它正在 Publishing 消息，而 listener 说 I heard 这些消息。这验证了 Python API 是否正常工作。

关于编写简单的 C++ 发布者和订阅者，请参考[创建C++的简单发布者和订阅者](https://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber%28c%2B%2B%29)

### 小结

ros-noetic 的更多的示例教程请参考[官方教程](https://wiki.ros.org/ROS/Tutorials)

由于使用的是预编译好的 ROS ，当您在官网教程中遇到安装 ROS 包的步骤时请先选择跳过。

如果提示缺少包，考虑从源码包编译或者联系我们。
