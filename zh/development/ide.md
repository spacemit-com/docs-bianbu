---
sidebar_position: 5
---

# 集成开发环境

## VScodium

[VSCodium](https://vscodium.com/) 是 VS Code 的开源版本，可以从 Bianbu 源（仅限 2.0.2 之后的版本）安装。

```shell
sudo apt-get update
sudo apt-get install -y codium
```

或者从 VSCodium [发布页面](https://github.com/VSCodium/vscodium/releases)下载、安装。例如下载`1.94.2.24286`：

```shell
wget https://github.com/VSCodium/vscodium/releases/download/1.94.2.24286/VSCodium-linux-riscv64-1.94.2.24286.tar.gz
```

安装：

```shell
sudo mkdir /opt/vscodium
sudo tar xf VSCodium-linux-riscv64-1.94.2.24286.tar.gz -C /opt/vscodium
```

创建桌面快捷方式：

1. 创建桌面文件夹下创建一个文件，文件名为`vscodium.desktop`，内容如下：

   ```
   [Desktop Entry]
   Name=VSCodium
   Exec=/opt/vscodium/bin/codium
   Terminal=false
   Icon=/opt/vscodium/resources/app/resources/linux/code.png
   Type=Application
   ```

   注意，从 Bianbu 源安装的路径为 `/usr/share/codium`，请替换上面的 `/opt/vscodium`。

2. 保存后即可在桌面看到一个新的 VSCodium 快捷方式，右键点击，选择`允许运行`。
3. 双击即可运行。

已知问题：

- 交互流畅度欠佳

## 远程开发

远程开发模式：

![architecture ssh](./static/architecture-ssh.png)

### Visual Studio Code Remote - SSH

尚未支持。

### VS Code in the browser

部署 [code-server](https://github.com/coder/code-server) 远程 VS Code 网页，在浏览器上写代码。

在Bianbu v2.0rc1或更新版本上安装`code-server`：

```shell
sudo apt-get install code-server
```

运行`code-server`，其中`IP`要换成本机IP地址：

```shell
code-server --host IP --port PORT
```

注意，不指定`IP`时，默认为localhost，只能在本机浏览器访问，`PORT`通常可以不指定。当看到以下信息，表示`code-server`启动成功。

```
info HTTP server listening on http://IP:PORT/
info  Session server listening on ~/.local/share/code-server/code-server-ipc.sock
```

在任何电脑、平板上打开浏览器，访问`http://IP:PORT`，即可打开远程的文件夹和文件。

![code-server](./static/code-server.png)

已知问题：

- 并非所有扩展可以正常使用

### VSCodium Open Remote - SSH

VSCodium 的插件 [Open Remote - SSH](https://open-vsx.org/extension/jeanp413/open-remote-ssh) 支持远程开发。

1. 安装 VSCodium，参考[官方指南](https://vscodium.com/#install)。
2. 打开 VSCodium，进入扩展页面，搜索 `Open Remote - SSH`，然后安装。
3. 打开远程资源管理器，即可连接 SSH Target。
4. 连接成功，即可打开远程的文件夹和文件。
