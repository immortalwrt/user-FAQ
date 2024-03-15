在线构建网址：(https://firmware-selector.immortalwrt.org 或 https://fwselector.kyarucloud.moe)
> 详细教程：(https://github.com/1715173329/blog/issues/9）

使用[GitHub actions](https://github.com/science2468/immortalwrt-IB/blob/main/.github/workflows/immortalwrt-imagebuilder-x64.yml)或者在虚拟机（Linux物理机）内使用image builder编译生成的固件可以设置固件大小。**主要适用于有大容量存储的设备（例如：X86、R2S、H28K 等等）**
>参考命令：https://github.com/openwrt/openwrt/blob/master/target/imagebuilder/files/Makefile \
>参考文档：https://openwrt.org/docs/guide-user/additional-software/imagebuilder

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
