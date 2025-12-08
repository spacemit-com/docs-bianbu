---
sidebar_position: 3
---

# JavaScript 使用指南

## Node.js 使用指南

### 使用 `apt` 安装 Node.js

```shell
sudo apt-get update
sudo apt-get install -y nodejs npm
```

检查是否安装成功。

```shell
$ node -v
v18.13.0
$ npm -v
9.2.0
```

Bianbu 源中的 Node.js 默认版本为 v18.13.0，若想使用特定版本的 Node.js，请使用 NVM 安装。

### 使用 NVM 安装特定版本 Node.js

#### 安装 NVM

根据 [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm) 提供的命令行下载并执行安装脚本，请根据该仓库替换最新的 NVM 版本号。

使用 `curl`：

```shell
sudo apt install curl
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

或使用 `wget`：

```bash
sudo apt install wget
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

检查是否安装成功。

```shell
$ nvm -v
0.40.1
```

#### 安装 Node.js

目前官方源中的 node 尚未适配 RISC-V，直接安装会导致错误，因此我们从 unofficial-builds 中下载已适配 RISC-V 的node。更多信息请参考 [https://github.com/nodejs/unofficial-builds/](https://github.com/nodejs/unofficial-builds/)

```shell
NVM_NODEJS_ORG_MIRROR=https://unofficial-builds.nodejs.org/download/release nvm install 20.16.0
```

检查是否安装成功。

```shell
$ node -v
v20.16.0
```

### 示例

#### clumsy-bird

克隆 demo 项目。

```shell
git clone https://github.com/ellisonleao/clumsy-bird
```

安装依赖。

```shell
cd clumsy-bird
npm install
sudo apt install grunt
```

运行服务。

```shell
grunt connect
```

你可以看到如下输出，服务默认部署在 `http://0.0.0.0:8001`，在浏览器中打开该网页进行体验。

```shell
$ grunt connect
Running "connect:root" (connect) task
Waiting forever...
Started connect web server on http://0.0.0.0:8001
```

## Electron 使用指南

### 准备工作

请先安装Node.js。

### 快速开始

下载 [electron-quick-start](https://github.com/electron/electron-quick-start.git)，

```shell
git clone https://github.com/electron/electron-quick-start.git ~/electron-quick-start
```

安装 SpacemiT 的 Electron 包，

```shell
cd ~/electron-quick-start
ELECTRON_MIRROR=http://archive.spacemit.com/electron/ electron_use_remote_checksums=1 npm install electron@29.3.1
```

启动应用，

```shell
npm start
```

当看到下面界面，表示成功。

![electron-quick-start](./static/electron-quick-start.png)

## electron-builder使用指南

**electron-builder** 是一个用于简化 Electron 应用打包和发布的工具，支持多平台构建和自动更新功能。目前官方源中的 electron-builder 和相关组件未完全适配 RISC-V 平台，因此需要使用 SpacemiT 适配的内部版本。这里以 [electron-quick-start](https://github.com/electron/electron-quick-start) 为打包项目。

### 准备工作

请先安装 Node.js。

### 克隆仓库

```shell
git clone https://github.com/electron/electron-quick-start.git
cd electron-quick-start
```

### 配置 `package.json`

```shell
vim package.json
```

在 `scripts` 下添加打包命令，分别用于目录、压缩包。这里使用了 SpacemiT 适配 RISC-V 的 Electron 镜像。

```json
  "scripts": {
    "pack-dir": "ELECTRON_MIRROR=http://archive.spacemit.com/electron/ electron-builder --linux --dir",
    "pack-tgz": "ELECTRON_MIRROR=http://archive.spacemit.com/electron/ electron-builder --linux tar.gz"
  }
```

添加打包配置。

```json
  "build": {
    "productName": "demo",
    "directories": {
      "output": "build"
    },
    "linux": {
      "category": "Utility"
    }
  },
```

添加依赖项，请使用 `@【仓库】/【依赖包】` 来指定从哪个仓库下载包。`@electron` 指 SpacemiT 的 [Node.js包仓库](https://git.spacemit.com/electron/electron-builder/-/packages)。

```json
  "devDependencies": {
    "electron": "29.3.1",
    "@electron/electron-builder": "25.0.0-alpha.5"
  }
```

最后你的 `package.json` 会像这样：

```json
{
  "name": "electron-quick-start",
  "version": "1.0.0",
  "description": "A minimal Electron application",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "pack-dir": "ELECTRON_MIRROR=http://archive.spacemit.com/electron/ electron-builder --linux --dir",
    "pack-tgz": "ELECTRON_MIRROR=http://archive.spacemit.com/electron/ electron-builder --linux tar.gz"
  },
  "repository": "https://github.com/electron/electron-quick-start",
  "keywords": [
    "Electron",
    "quick",
    "start",
    "tutorial",
    "demo"
  ],
  "author": "GitHub",
  "license": "CC0-1.0",
  "build": {
    "productName": "demo",
    "directories": {
      "output": "build"
    },
    "linux": {
      "category": "Utility"
    }
  },
  "devDependencies": {
    "electron": "29.3.1",
    "@electron/electron-builder": "25.0.0-alpha.5"
  }
}
```

### 配置仓库地址

```shell
npm config set @electron:registry https://git.spacemit.com/api/v4/projects/36/packages/npm/
```

### 安装依赖

这里通过镜像地址指定使用适配 RISC-V 的 electron，请勿在 `npm config` 配置 Electron 和 electron-builder 的镜像地址，否则命令指定的镜像地址将失效。可以通过 `npm config list` 检查。

```shell
ELECTRON_MIRROR=http://archive.spacemit.com/electron/ electron_use_remote_checksums=1 npm install
```

### 项目打包

#### 以目录发布

```shell
npm run pack-dir
```

该命令会在输出目录（此处设为 build）下输出文件夹`linux-riscv64-unpacked`。

```shell
bianbu@k1:~/electron-quick-start$ tree build/linux-riscv64-unpacked/ -L 1
build/linux-riscv64-unpacked/
├── chrome_100_percent.pak
├── chrome_200_percent.pak
├── chrome_crashpad_handler
├── chrome_sandbox
├── electron-quick-start
├── icudtl.dat
├── libEGL.so
├── libffmpeg.so
├── libGLESv2.so
├── libvk_swiftshader.so
├── libvulkan.so.1
├── LICENSE.electron.txt
├── LICENSES.chromium.html
├── locales
├── resources
├── resources.pak
├── snapshot_blob.bin
├── v8_context_snapshot.bin
└── vk_swiftshader_icd.json

3 directories, 17 files
```

#### 以压缩包发布

```shell
npm run pack-tgz
```

该命令会在输出目录（此处设为 build）下输出压缩包 `electron-quick-start-1.0.0-riscv64.tar.gz`。

```shell
bianbu@k1:~/electron-quick-start$ tar -ztf build/electron-quick-start-1.0.0-riscv64.tar.gz 
electron-quick-start-1.0.0-riscv64/
electron-quick-start-1.0.0-riscv64/LICENSE.electron.txt
electron-quick-start-1.0.0-riscv64/LICENSES.chromium.html
electron-quick-start-1.0.0-riscv64/chrome_100_percent.pak
electron-quick-start-1.0.0-riscv64/chrome_200_percent.pak
electron-quick-start-1.0.0-riscv64/chrome_crashpad_handler
electron-quick-start-1.0.0-riscv64/chrome_sandbox
electron-quick-start-1.0.0-riscv64/electron-quick-start
electron-quick-start-1.0.0-riscv64/icudtl.dat
electron-quick-start-1.0.0-riscv64/libEGL.so
electron-quick-start-1.0.0-riscv64/libGLESv2.so
electron-quick-start-1.0.0-riscv64/libffmpeg.so
electron-quick-start-1.0.0-riscv64/libvk_swiftshader.so
electron-quick-start-1.0.0-riscv64/libvulkan.so.1
electron-quick-start-1.0.0-riscv64/locales/
electron-quick-start-1.0.0-riscv64/resources/
electron-quick-start-1.0.0-riscv64/resources.pak
electron-quick-start-1.0.0-riscv64/snapshot_blob.bin
electron-quick-start-1.0.0-riscv64/v8_context_snapshot.bin
electron-quick-start-1.0.0-riscv64/vk_swiftshader_icd.json
electron-quick-start-1.0.0-riscv64/locales/......
electron-quick-start-1.0.0-riscv64/resources/app-update.yml
electron-quick-start-1.0.0-riscv64/resources/app.asar
```

#### 以deb发布

发布 `.deb` 包的压缩耗时较长，请耐心等待。

打包 `.deb` 要求设置 email 等信息，请在 `package.json` 中添加下列信息。

```json
  "author": "lff <junzhao.liang@spacemit.com>",
  "email": "junzhao.liang@spacemit.com",
  "homepage": "www.google.com",
```

安装 FPM。

```shell
sudo apt install ruby3.1
sudo gem install fpm
```

打包 `.deb`，可相应修改命令参数。发布压缩包的压缩耗时较长，请耐心等待。

```shell
mkdir tmp
fpm -s dir --force -t deb -d libgtk-3-0 -d libnotify4 -d libnss3 -d libxss1 -d libxtst6 -d xdg-utils -d libatspi2.0-0 -d libuuid1 -d libsecret-1-0 --deb-recommends libappindicator3-1 \
--deb-compression xz \
--architecture riscv64 \
--description 'A minimal Electron application' \
--version 1.0.0 \
--package /home/bianbu/electron-quick-start/build/electron-quick-start_1.0.0_riscv64.deb \
--name electron-quick-start \
--maintainer 'lff <junzhao.liang@spacemit.com>' \
--url https://www.google.com \
--vendor 'lff <junzhao.liang@spacemit.com>' \
--deb-priority optional \
--license CC0-1.0 \
/home/bianbu/electron-quick-start/build/linux-riscv64-unpacked/=/opt/demo \
./tmp=/usr/share/applications/electron-quick-start.desktop
```

在 `fpm` 的打包命令中添加下列参数，可设置应用图标。

```shell
/home/bianbu/electron-quick-start/node_modules/@electron/app-builder-lib/templates/icons/electron-linux/16x16.png=/usr/share/icons/hicolor/16x16/apps/electron-quick-start.png
```

该命令会在输出目录（此处设为 build）下输出 `.deb` 包 `electron-quick-start-1.0.0-riscv64.deb`。

### 运行应用

#### 以目录发布

直接运行目录下的 `electron-quick-start`。

```shell
cd build/linux-riscv64-unpacked/
./electron-quick-start
```

#### 以压缩包发布

将压缩包解压，运行解压目录下的 `electron-quick-start`。

```shell
cd build
tar -zxf electron-quick-start-1.0.0-riscv64.tar.gz
cd electron-quick-start-1.0.0-riscv64
./electron-quick-start
```

#### 以deb包发布

安装 `.deb` 包，执行安装的应用。

```shell
cd build
sudo dpkg -i electron-quick-start_1.0.0_riscv64.deb
/opt/demo/electron-quick-start
```
