# ImmortalWrt SDK 小白版

> [!IMPORTANT]
> 本文只教你**用现成的源码**编译出适合你设备架构 / 固件版本的插件。想学怎么从零写一个插件的源码，请直接略过本文。
>
> **SDK 是把某个插件的源码编译成安装包的工具（23.05/24.10 是 `.ipk`，25.12 起是 `.apk`），不能编译生成固件。**

参考文档：<https://openwrt.org/docs/guide-developer/toolchain/using_the_sdk>

## 使用前的准备

- Linux 发行版：Ubuntu 22.04 桌面版（desktop），虚拟机或物理机都行。
- **SDK 版本必须和目标固件的版本线一致**（23.05 的 SDK 编出来的包，不要装到 24.10 / 25.12 的固件上），架构也必须一致。

## 需要安装的依赖

```sh
sudo apt update -y
```

```sh
sudo apt full-upgrade -y
```

```sh
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache clang cmake cpio curl device-tree-compiler ecj fastjar flex gawk gettext gcc-multilib \
  g++-multilib git gnutls-dev gperf haveged help2man intltool lib32gcc-s1 libc6-dev-i386 libelf-dev \
  libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5 \
  libncursesw5-dev libpython3-dev libreadline-dev libssl-dev libtool lld llvm lrzsz mkisofs msmtp \
  nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pip python3-ply \
  python3-docutils python3-pyelftools qemu-utils re2c rsync scons squashfs-tools subversion swig \
  texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```

> [!IMPORTANT]
> 依赖列表以 **Ubuntu 22.04** 为准。Ubuntu 24.04 已移除 `python2.7` 和 `python3-distutils`，请把这两个包名删掉再执行。

## 下载对应架构的 SDK

以 x86-64 + 23.05.1 为例（版本号和文件名请按需替换），在文件管理器的 Downloads 目录里右键"在终端中打开"再粘贴：

```sh
wget https://downloads.immortalwrt.org/releases/23.05.1/targets/x86/64/immortalwrt-sdk-23.05.1-x86-64_gcc-12.3.0_musl.Linux-x86_64.tar.xz
```

> [!NOTE]
> 24.10 / 25.12 的 SDK 是 `.tar.zst`（对应 GCC 13.3.0 / 14.3.0），文件名形如 `immortalwrt-sdk-25.12.2-x86-64_gcc-14.3.0_musl.Linux-x86_64.tar.zst`，解压命令见下一节。

## 解压并进入 SDK 目录

```sh
tar -J -x -f immortalwrt-sdk-*.tar.xz
```

```sh
cd immortalwrt-sdk-*/
```

24.10 / 25.12 换成：

```sh
tar --zstd -xf immortalwrt-sdk-*.tar.zst
```

（系统没有 zstd 时先 `sudo apt install zstd`）

## 加载软件包列表

```sh
./scripts/feeds update -a
```

```sh
./scripts/feeds install -a
```

## 以 homeproxy 为例

```sh
cd package
```

```sh
git clone https://github.com/immortalwrt/homeproxy.git
```

```sh
cd ..
```

## 编译插件

homeproxy 里面有现成的 `Makefile`，所以可以直接编译。

可以先 `make menuconfig` 进入菜单界面选择 `luci-app-homeproxy`，保存后再 `make`；也可以直接指定包名编译：

```sh
make package/homeproxy/luci-app-homeproxy/compile V=s
```

## 编译生成的安装包位置

- `bin/packages/`
- `bin/targets/`

拿到安装包后，拷到路由器上安装即可：

```sh
# 24.10 及更早
opkg install ./luci-app-homeproxy_xxx.ipk
# 25.12 / 快照
apk add --allow-untrusted ./luci-app-homeproxy-xxx.apk
```
