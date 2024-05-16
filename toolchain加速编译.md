# 有问题先看[文档](https://openwrt.org/docs/guide-developer/external_toolchain)
编译自己需求的op固件需要耗费大量时间，使用toolchain可以加快编译速度。根据我使用的经历，4核心8线程的编译过程缩短21分钟。toolchain不要乱用，要用相同架构。建议多看教程、文档；在动手期间多使用搜索引擎！
### 以x86-64设备为例
# 要求：Ubuntu22.04桌面版（desktop）
```
sudo apt update -y
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
# 在桌面右键打开命令行（terminal）下载23.05源码
```
git clone -b openwrt-23.05 https://github.com/immortalwrt/immortalwrt.git
```
```
cd immor*
```
# 加载依赖。
**`./scripts/feeds install -a`建议执行2次**
```
./scripts/feeds update -a
```
```
./scripts/feeds install -a
```
```
cd
```
# 下载toolchain后解压文件再删除源文件
```
wget https://downloads.staging.immortalwrt.org/releases/23.05.2/targets/x86/64/immortalwrt-toolchain-23.05.2-x86-64_gcc-12.3.0_musl.Linux-x86_64.tar.xz
```
```
tar -xf immortalwrt-tool*.tar.xz
```
```
rm immortalwrt-tool*.tar.xz
```
# 把toolchain路径添加到`./scripts/ext-toolchain.sh`
```
./scripts/ext-toolchain.sh --toolchain /home/xxx/immortalwrt-tool*/toolchain-*/ --config x86/64
```
```
make defconfig
```
`/home/xxx/immortalwrt-tool*`的xxx是安装系统时设置的用户名，最好是直接复制文件路径。\
以上生成的是x86-64设备默认配置文件（`.config`）。配置文件在生成后可以更改，但不要随意更改，可以增加插件
（给个例子：在immortalwrt文件夹内右键点击空白处运行命令行（terminal）执行\
`sed -i 's/# CONFIG_GRUB_IMAGES is not set/CONFIG_GRUB_IMAGES=y/' .config`代表生成非efi类型的固件。
# 编译固件
```
make
```
# 固件位置在以下文件夹内
`immortalwrt/bin/targets/`
