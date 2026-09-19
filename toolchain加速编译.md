# 用外部 toolchain 加速编译固件

从源码编译固件很费时间，使用**官方预编译好的 toolchain** 可以省掉"自举编译工具链"这一大段，编译总时长会明显缩短（实测 4 核 8 线程大约省下 20 分钟）。

- **toolchain 必须和固件架构对应**，不要乱用（x86-64 就用 x86-64 的）。
- 建议多看官方文档：[External toolchain](https://openwrt.org/docs/guide-developer/external_toolchain)，动手期间多使用搜索引擎。

> [!NOTE]
> 本文以 **x86-64 + ImmortalWrt 23.05** 为例。各版本线的内核、GCC 版本和归档格式不同（24.10 起官方产物是 `.tar.zst`，GCC 13/14），下载链接里的**版本号和文件名请按需替换**，对照表见 [README](./README.md) 的「版本与命令对照」。

## 1. 准备环境（Ubuntu 22.04 桌面版）

虚拟机或物理机都行。

```sh
sudo apt update -y
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

> [!NOTE]
> 上面的依赖列表以 Ubuntu 22.04 为准。**Ubuntu 24.04 已移除 `python2.7` 和 `python3-distutils`**，请把这两个包名删掉再执行，否则 `apt` 会直接报"无法定位软件包"。

## 2. 下载源码

```sh
git clone -b openwrt-23.05 https://github.com/immortalwrt/immortalwrt.git
cd immortalwrt
```

## 3. 加载 feeds

`./scripts/feeds install -a` **建议连续执行两次**（第一次常常装不全，缺包会在编译中途才爆出来）：

```sh
./scripts/feeds update -a
```

```sh
./scripts/feeds install -a
```

```sh
./scripts/feeds install -a
```

## 4. 下载并解压 toolchain

回到上一级目录（HOME）下载，**注意版本号要和源码分支对应**：

```sh
cd ~
```

```sh
wget https://downloads.immortalwrt.org/releases/23.05.2/targets/x86/64/immortalwrt-toolchain-23.05.2-x86-64_gcc-12.3.0_musl.Linux-x86_64.tar.xz
```

```sh
tar -xf immortalwrt-tool*.tar.xz
```

```sh
rm immortalwrt-tool*.tar.xz
```

> [!TIP]
> 24.10 / 25.12 的官方 toolchain 是 `.tar.zst`，文件名类似 `immortalwrt-toolchain-24.10.6-x86-64_gcc-13.3.0_musl.Linux-x86_64.tar.zst`，解压命令改成 `tar --zstd -xf immortalwrt-tool*.tar.zst`（系统没有 zstd 时先 `sudo apt install zstd`）。

## 5. 把 toolchain 路径告诉编译系统

**先回到源码目录再执行**（这一步很容易漏掉，漏了会报 `./scripts/ext-toolchain.sh: No such file or directory`，因为此时你还在 HOME）：

```sh
cd ~/immortalwrt
```

```sh
./scripts/ext-toolchain.sh --toolchain /home/<你的用户名>/immortalwrt-tool*/toolchain-*/ --config x86/64
```

```sh
make defconfig
```

`/home/<你的用户名>/immortalwrt-tool*` 就是第 4 步解压出来的目录，路径不确定时直接在文件管理器里复制完整路径。

## 6. 编译

```sh
make
```

`make defconfig` 生成的是 x86-64 的默认配置（`.config`），生成后可以继续加插件，但**不要随意改动**。例如要生成非 EFI 类型的固件：

```sh
sed -i 's/# CONFIG_GRUB_IMAGES is not set/CONFIG_GRUB_IMAGES=y/' .config
```

固件产物位置：`immortalwrt/bin/targets/`
