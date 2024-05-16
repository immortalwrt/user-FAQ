**特别注意：本文用途只是教你用现成的源码编译出适合你设备架构或op版本的插件，想学怎么编译没有现成源码的插件直接略过本文**
# **SDK是将你某个插件从源码编译成后缀为`.ipk`插件的工具，不能编译生成固件。**
>参考文档：https://openwrt.org/docs/guide-developer/toolchain/using_the_sdk

### 使用前的准备
Linux发行版：Ubuntu22.04桌面版（desktop），用虚拟机或者是物理机都行。
### 需要安装的依赖
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
### 下载对应架构的immortalwrtSDK（23.05版本）文件,在文件管理器内的Downloads文件夹内用鼠标右键点击开启Terminal（命令行）粘贴下面的命令（以X86-64为例）
```
wget https://downloads.immortalwrt.org/releases/23.05.1/targets/x86/64/immortalwrt-sdk-23.05.1-x86-64_gcc-12.3.0_musl.Linux-x86_64.tar.xz
```
### 解压文件后进入到SDK文件夹内
```
tar -J -x -f immortalwrt-sdk-*.tar.xz
```
```
cd immortalwrt-sdk-*/
```
### 加载软件包列表
```
./scripts/feeds update -a
```
```
./scripts/feeds install -a
```
### 以homeproxy为例
```
cd package
```
```
git clone https://github.com/immortalwrt/homeproxy.git
```
```
cd ..
```
### 编译插件
homeproxy里面有现成的`Makefile`。\
可以使用`make menuconfig`命令进入菜单界面选择`luci-app-homeproxy` save（保存）后使用`make`命令编译

```
make package/homeproxy/luci-app-homeproxy/compile V=s
```

### 编译生成的`.ipk`文件位置目录
`bin/packages` `bin/targets`
