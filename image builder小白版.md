# Image Builder 小白版

**<sup>推荐</sup> 在线构建网址**

- 官方：[firmware-selector.immortalwrt.org](https://firmware-selector.immortalwrt.org)
- 镜像：[fwselector.kyarucloud.moe](https://fwselector.kyarucloud.moe)
- 详细教程：[ImmortalWrt 在线构建服务 使用说明](https://github.com/1715173329/blog/issues/9)

在虚拟机（或 Linux 物理机）内使用 Image Builder 编译固件，**可以自己设置固件大小**。**必须在非 ROOT 环境下运行。** **主要适用于有大容量存储的设备（例如：X86、R2S、~~H28K~~ 等等）。**

> 参考命令：<https://github.com/openwrt/openwrt/blob/master/target/imagebuilder/files/Makefile>
>
> 参考文档：<https://openwrt.org/docs/guide-user/additional-software/imagebuilder>

> [!NOTE]
> 本文以 **x86-64 + 23.05.1** 为例（下载链接里的版本号请按需替换）。24.10 及更新版本的官方产物是 `.tar.zst` 而不是 `.tar.xz`，解压命令不同，对照表见 [README](./README.md) 的「版本与命令对照」。

## 1. 安装依赖（以 Ubuntu 22.04 为例）

只装 Image Builder 必需的部分：

```sh
sudo apt install build-essential libncurses-dev zlib1g-dev gawk git gettext libssl-dev xsltproc rsync wget unzip python3 python3-distutils
```

当然也可以装编译固件所需的全部依赖：

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
> 上面的依赖列表以 **Ubuntu 22.04** 为准。Ubuntu 24.04 已移除 `python2.7` 和 `python3-distutils`，请把这两个包名删掉再执行。

## 2. 下载并解压 Image Builder

用 `wget` 把**目标架构**对应的 Image Builder 下到 `~/Downloads` 下（大多数人用桌面版 Ubuntu，直接在文件管理器里进入 Downloads 目录、右键"在终端中打开"再粘贴命令即可）。

以 x86-64 为例：

```sh
wget https://downloads.immortalwrt.org/releases/23.05.1/targets/x86/64/immortalwrt-imagebuilder-23.05.1-x86-64.Linux-x86_64.tar.xz
```

解压并进入目录：

```sh
tar -J -x -f immortalwrt-imagebuilder-*.tar.xz
```

```sh
cd immortalwrt-imagebuilder-*/
```

> [!TIP]
> 24.10 / 25.12 的官方包是 `.tar.zst`（例如 `immortalwrt-imagebuilder-25.12.2-x86-64.Linux-x86_64.tar.zst`），解压命令改成 `tar --zstd -xf immortalwrt-imagebuilder-*.tar.zst`；系统没有 zstd 时先 `sudo apt install zstd`。

## 3. 生成固件

主要使用 `make image` 命令：

```
image:
	By default 'make image' will create an image with the default
	target profile and package set. You can use the following parameters
	to change that:

	make image PROFILE="<profilename>" # override the default target profile
	make image PACKAGES="<pkg1> [<pkg2> [<pkg3> ...]]" # include extra packages
	make image FILES="<path>" # include extra files from <path>
	make image BIN_DIR="<path>" # alternative output directory for the images
	make image EXTRA_IMAGE_NAME="<string>" # Add this to the output image filename (sanitized)
	make image DISABLED_SERVICES="<svc1> [<svc2> [<svc3> ..]]" # Which services in /etc/init.d/ should be disabled
	make image ADD_LOCAL_KEY=1 # store locally generated signing key in built images
	make image ROOTFS_PARTSIZE="<size>" # override the default rootfs partition size in MegaBytes
```

实际最常用的是 `PACKAGES` 和 `ROOTFS_PARTSIZE`。

**例子：**

```sh
make image PACKAGES="luci-i18n-base-zh-cn luci-i18n-opkg-zh-cn luci-i18n-firewall-zh-cn -kmod-r8125" ROOTFS_PARTSIZE="800"
```

说明：

- 双引号内是需要安装或删除的插件，**带 `-` 就是删除，不带就是增加**（例子里 `-kmod-r8125` 是删掉 8125 驱动）；`800` 表示 rootfs 分区大小 800MB（不严谨地说就是固件大小）。这条命令在虚拟机和 Linux 物理机内同样适用。
- 对于 24.10 及以上版本，`luci-app-opkg` 已更名为 `luci-app-package-manager`，上述 `luci-i18n-opkg-zh-cn` 需替换为 `luci-i18n-package-manager-zh-cn`

**常用插件名称（带语言包）：**

```
luci-i18n-homeproxy-zh-cn     # HomeProxy
luci-i18n-diskman-zh-cn       # 磁盘管理
luci-i18n-mwan3-zh-cn         # 多拨
luci-i18n-nlbwmon-zh-cn       # 流量统计
luci-i18n-samba4-zh-cn        # 文件共享
luci-i18n-smartdns-zh-cn      # SmartDNS
luci-i18n-transmission-zh-cn  # BT or PT
luci-i18n-upnp-zh-cn          # 公网映射
luci-i18n-zerotier-zh-cn      # ZeroTier
```

插件名称的规律：**带语言包的是 `luci-i18n-xxx-zh-cn`，不带语言包的是 `luci-app-xxx`**；插件之间还有依赖关系，有能力的可以自己去查插件的 `Makefile`。

完整例子：

```sh
make image PACKAGES="luci-i18n-homeproxy-zh-cn luci-i18n-base-zh-cn luci-i18n-opkg-zh-cn luci-i18n-firewall-zh-cn -kmod-r8125" ROOTFS_PARTSIZE="800"
```

即：800MB、带 homeproxy、不带 8125 驱动的 x86-64 固件。

生成的固件位置一般在 **`bin/targets/`** 目录下（相对于 Image Builder 解压出来的目录，注意不是系统根目录的 `/bin`）。

> [!TIP]
> `PACKAGES` 里只能填**官方软件源里已有的包**，第三方插件（不在官方源里的）要么先加 feed 再装，要么用完整源码编译。可用插件清单看官方软件包目录，例如 [23.05.1 / x86_64](https://downloads.immortalwrt.org/releases/23.05.1/packages/x86_64/) 下的 `base`、`luci`、`packages` 三个目录，基本覆盖了常用插件，这三个里面找不到的就是没有。
