**<sup>推荐</sup> 在线构建网址：(https://firmware-selector.immortalwrt.org 或 https://fwselector.kyarucloud.moe)**
> 详细教程：(https://github.com/1715173329/blog/issues/9）

使用[GitHub actions](https://github.com/science2468/immortalwrt-IB/blob/main/.github/workflows/immortalwrt-imagebuilder-x64.yml)或者在虚拟机（Linux物理机）内使用image builder编译生成的固件可以设置固件大小。 **必须在非ROOT环境下运行。** **主要适用于有大容量存储的设备（例如：X86、R2S、H28K 等等）。** 
>参考命令：https://github.com/openwrt/openwrt/blob/master/target/imagebuilder/files/Makefile \
>参考文档：https://openwrt.org/docs/guide-user/additional-software/imagebuilder

在虚拟机内或者是Linux物理机上需要安装的依赖。以Ubuntu22.04举例。
在命令行（terminal）粘贴运行以下命令安装image builder所必须的依赖：
```
sudo apt install build-essential libncurses-dev zlib1g-dev gawk git gettext libssl-dev xsltproc rsync wget unzip python3 python3-distutils
```
当然你也可以安装编译固件所需要的全部依赖：
```
sudo apt update -y
```
```
sudo apt full-upgrade -y
```
```
sudo apt install -y ack antlr3 asciidoc autoconf automake autopoint binutils bison build-essential \
  bzip2 ccache clang cmake cpio curl device-tree-compiler ecj fastjar flex gawk gettext gcc-multilib \
  g++-multilib git gnutls-dev gperf haveged help2man intltool lib32gcc-s1 libc6-dev-i386 libelf-dev \
  libglib2.0-dev libgmp3-dev libltdl-dev libmpc-dev libmpfr-dev libncurses5-dev libncursesw5 \
  libncursesw5-dev libpython3-dev libreadline-dev libssl-dev libtool lld llvm lrzsz mkisofs msmtp \
  nano ninja-build p7zip p7zip-full patch pkgconf python2.7 python3 python3-pip python3-ply \
  python3-docutils python3-pyelftools qemu-utils re2c rsync scons squashfs-tools subversion swig \
  texinfo uglifyjs upx-ucl unzip vim wget xmlto xxd zlib1g-dev
```
安装完成后，使用`wget`命令将目标架构所需要的目标image builder下载到/home/Downloads文件夹下。大多数人都是用桌面版的Ubuntu，所以在文件管理器内的Downloads文件夹内用鼠标右键点击开启Terminal（命令行）粘贴下面的命令
```
wget https://downloads.immortalwrt.org/releases/23.05.1/targets/x86/64/immortalwrt-imagebuilder-23.05.1-x86-64.Linux-x86_64.tar.xz
```
用以下命令解压并进入到immortalwrt image builder文件夹内
```
tar -J -x -f immortalwrt-imagebuilder-*.tar.xz
```
```
cd immortalwrt-imagebuilder-*/
```

主要使用`make image`命令\
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
上面的命令中使用`make image PACKAGES`和`make image ROOTFS_PARTSIZE`。\
举例：`make image PACKAGES="kmod-alx luci-i18n-base-zh-cn luci-i18n-opkg-zh-cn luci-i18n-firewall-zh-cn -kmod-r8125" ROOTFS_PARTSIZE="800"` \
说明：双引号内的是需要安装或删除的插件。`-kmod-r8125`是删除8125驱动，`-`就是删除，不带`-`就是增加，800是rootfs分区大小是800MB（不严谨的说就是固件大小）。举例给出的命令在虚拟机和Linux物理机内同样适用。
