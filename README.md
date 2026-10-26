# ImmortalWrt 用户 FAQ

面向 ImmortalWrt **普通用户**（非开发者）的常见问题与上手文档合集。提问之前先在这里搜一遍 —— 群里反复出现的问题，大部分本文档集里已经有答案。

## 文档目录

| 文档 | 讲什么 |
|---|---|
| [immortalwrt 常见问题指北](./immortalwrt%20常见问题指北.md) | 18 条高频问题速答：插件、IPv6、断流、DNS 泄露…… |
| [小白问题速查](./小白问题速查.md) | 自编译 / 在线构建（selector）相关的速查清单 |
| [image builder 小白版](./image%20builder小白版.md) | 用 Image Builder 在 Linux 上自己出固件（可定制插件、分区大小） |
| [immortalwrtSDK小白版](./immortalwrtSDK小白版.md) | 用 SDK 把某一个插件的源码单独编译成安装包 |
| [toolchain加速编译](./toolchain加速编译.md) | 用官方预编译 toolchain 加速从源码编译固件 |
| [关于eui64的一些说明](./关于eui64的一些说明.md) | EUI-64 地址怎么来的、为什么用它做 IPv6 端口转发 |

## 版本与命令对照（动手前先看这张表）

各版本线的包管理器、安装包后缀、官方产物归档格式**都不一样**，照抄别人的命令前先对照本表：

| 版本线 | 最新版本 | 包管理器 | 安装包后缀 | 官方产物归档 | 内核（x86-64） | 工具链 GCC |
|---|---|---|---|---|---|---|
| 23.05 | 23.05.7 | `opkg` | `.ipk` | `.tar.xz` | 5.15 | 12.3.0 |
| 24.10 | 24.10.6 | `opkg` | `.ipk` | `.tar.zst` | 6.6 | 13.3.0 |
| 25.12 | 25.12.2 | **`apk`** | **`.apk`** | `.tar.zst` | 6.12 | 14.3.0 |

由这张表可以直接推出两条最常踩的坑：

- **装包命令不同**：24.10 及更早用 `opkg update` / `opkg install 包名`；**25.12 以及 master 快照改用 `apk update` / `apk add 包名`**。
- **解压命令不同**：24.10 起 Image Builder / SDK / toolchain 的官方包是 `.tar.zst`，`tar -J -x -f` 只认 `.tar.xz`，新格式要 `tar --zstd -xf`（必要时先 `apt install zstd`）。

> 上表的版本号、安装包后缀、内核与 GCC 版本取自 [downloads.immortalwrt.org](https://downloads.immortalwrt.org/releases/) 的官方产物与镜像清单，可自行核对。

## 反馈与贡献

- 有疑问或发现文档写错，欢迎在这个仓库提 [Issue](https://github.com/immortalwrt/user-FAQ/issues) 或直接提交 PR。
- 提问时请先说清 **用的是哪个版本线**（23.05 / 24.10 / 25.12 / 快照）和 **什么设备**，这两条信息能省掉一半来回。
- 本仓库收集"群友能独立自主解决"的问题，如果你的问题适合写进文档，欢迎 PR。

## 授权

本仓库文档采用 **CC BY-SA 4.0** 授权，全文见仓库根目录的 [LICENSE](./LICENSE)。
