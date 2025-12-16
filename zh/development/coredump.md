---
sidebar_position: 6
---

# Coredump

通常情况下，core文件（core dump file）会包含了程序运行时的内存，寄存器状态，堆栈指针，内存管理信息还有各种函数调用堆栈信息等，我们可以理解为是程序工作当前状态存储生成第一个文件，许多的程序出错的时候都会产生一个core文件，通过工具分析这个文件，我们可以定位到程序异常退出的时候对应的堆栈调用等信息，找出问题所在并进行及时解决。

## 使用 apport 收集

Bianbu 2.0rc1及之后版本，已默认安装 apport 收集崩溃信息。

### 简介

apport 是一个错误收集系统，会收集软件崩溃、未处理异常并生成崩溃报告。当一个应用程序崩溃或者出现 Bug 时候，apport 就会通过弹窗警告用户并且询问用户是否提交崩溃报告。一旦程序崩溃过一次，就会生成一个 `.crash`文件，记录程序崩溃信息并保存在目录 `/var/crash`。同一程序，只会保存第一次 crash 的那份，调试时可删除。

### 调试

先从崩溃报告中提取 Coredump 到指定目录，

```shell
apport-unpack /var/crash/_usr_bin_bash.0.crash /tmp/crash
```

gdb 调试，

```shell
gdb /usr/bin/bash /tmp/crash/CoreDump
```

如想查看完整符号表，需提前安装相关dbg包（如bash-dbgsym）。

### 上传崩溃报告

如遇到无法自行解决的崩溃，可选择上传崩溃报告到工单系统，我们将协助您解决。

#### 前提条件

配置工单系统的 [api key](https://ticket.spacemit.com/my/api_key) 到 `~/.config/apport/redmine.credentials` 文件中

#### 上传

崩溃发生时，会有如下弹窗，按照提示填写相关信息，点击**发送**即可自动上传崩溃报告并打开浏览器跳转到刚刚上传的崩溃报告。

![apport崩溃弹窗](./static/apport.png)

也可暂时选择不发送，后续可使用如下命令重新显示弹窗。

```shell
touch /var/crash/_usr_bin_bash.0.crash
```

#### 查看已上传的崩溃报告

可前往 [时空服务](https://ticket.spacemit.com/issues) 查看，我们也将在其中回复您的问题。

## 不用apport

如果希望内核直接生成 core 文件，而不经过 apport，可参考如下内容。

### 开启coredump

系统默认关闭 coredump，可以通过`ulimit -c`查看，

```shell
ulimit -c
0
```

为0说明不生成 core dump 文件。如需开启 coredump，需要设置 core 文件大小和路径。

#### 设置core文件大小

通过命令`ulimit -c unlimited`​设置生成的core文件大小不限，也可以按自己的需求设置大小。

```shell
ulimit -c unlimited
```

命令只对当前终端有效，如果希望系统启动设置，可以编辑`/etc/security/limits.conf`文件，添加以下两行：

```shell
* soft core unlimited
* hard core unlimited
```

#### 设置core文件路径

设置 core 文件的路径和名称，

```shell
echo /var/crash/core-%e-%p-%t | sudo tee /proc/sys/kernel/core_pattern
```

命令只对当前终端有效，如果希望系统启动设置，可以编辑`/etc/sysctl.conf`文件，添加以下行：

```shell
kernel.core_pattern = /var/crash/core-%e-%p-%t
```

运行以下`sudo sysctl -p`使配置生效。

### 测试

运行以下命令，测试是否生效，

```shell
bash -c 'kill -SEGV $$'
```

如果生效，可以看到以下输出，

```shell
[1]    PID segmentation fault (core dumped)  bash -c 'kill -SEGV $$'
```

其中`PID`是bash当前进程号，与此同时，`/var/crash`目录会生成对应的core文件。
