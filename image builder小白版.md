**<sup>推荐</sup> 在线构建网址：(https://firmware-selector.immortalwrt.org 或 https://fwselector.kyarucloud.moe)**
> 详细教程：[ImmortalWrt 在线构建服务 使用说明](https://github.com/1715173329/blog/issues/9)

在虚拟机（Linux物理机）内使用image builder编译生成的固件可以设置固件大小。 **必须在非ROOT环境下运行。** **主要适用于有大容量存储的设备（例如：X86、R2S、~~H28K~~ 等等）。** 
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
安装完成后，使用`wget`命令将目标架构所需要的目标image builder下载到/home/Downloads文件夹下。大多数人都是用桌面版的Ubuntu，所以在文件管理器内的Downloads文件夹内用鼠标右键点击开启Terminal（命令行）粘贴下面的命令（以X86-X64设备为例）
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
举例：`make image PACKAGES="luci-i18n-base-zh-cn luci-i18n-opkg-zh-cn luci-i18n-firewall-zh-cn -kmod-r8125" ROOTFS_PARTSIZE="800"` \
说明：双引号内的是需要安装或删除的插件。`-kmod-r8125`是删除8125驱动，`-`就是删除，不带`-`就是增加，800是rootfs分区大小是800MB（不严谨的说就是固件大小）。举例给出的命令在虚拟机和Linux物理机内同样适用。\
举例：常用的插件名称（带语言包）：
```
luci-i18n-homeproxy-zh-cn # HP
luci-i18n-diskman-zh-cn #磁盘管理
luci-i18n-mwan3-zh-cn #多拨
luci-i18n-nlbwmon-zh-cn #流量统计
luci-i18n-samba4-zh-cn #文件共享
luci-i18n-smartdns-zh-cn #smartdns
luci-i18n-transmission-zh-cn #BT or PT
luci-i18n-upnp-zh-cn #公网映射 
luci-i18n-zerotier-zh-cn # zerotier
```
插件名称的规律,带语言包的是：`luci-i18n-xxx-zh-cn`；不带语言包的是：`luci-app-xxx`，插件的依赖有能力的需要自己去查插件的`makefile`\
举个例子：
```
make image PACKAGES="luci-i18n-homeproxy-zh-cn luci-i18n-base-zh-cn luci-i18n-opkg-zh-cn luci-i18n-firewall-zh-cn -kmod-r8125" ROOTFS_PARTSIZE="800"
```
说明：800MB带homeproxy，不带8125驱动的x86-64固件\
生成的固件位置一般在`/bin/targets/`或者是`/build_dir/`目录下\
[immortalwrt插件清单](https://downloads.immortalwrt.org/releases/23.05.1/packages/x86_64/)内的`base` `luci` `pakcages`基本上包含了所需要的插件，在这三个里面找不到的就是没有。

