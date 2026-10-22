# PPPoE 定时重拨与 wan6 只拿到 PD 的排查

## 1. 两个症状，常常一起出现

- **症状 A**：宽带每隔一段时间就断一次（几分钟到十几分钟不等，有的地区稳定在 3 分钟左右），`logread` 里反复出现 `LCP terminated by peer`，随后自动重拨。
- **症状 B**：`网络 → 接口` 里拨号接口只有 `IPv6-PD`，**IPv6 地址一栏是空的**；`wan6` 的 `ipv6-address` 为 `[]`。

这两件事经常同时被看到，但**它们不是因果关系**。实测：`wan` 完全没有 IPv6 地址时，会话也能稳定存活十几分钟才被断一次。

所以先分清是哪一种，再决定要不要折腾配置。

## 2. 一分钟自查：到底是本地问题还是运营商问题

### 2.1 先看断线间隔有没有规律

```sh
logread | grep -E "LCP terminated|is now up" | tail -20
```

- 间隔**规律**（比如每 3 分钟一次、误差几秒）→ 高度怀疑对端设备主动踢线
- 间隔**随机**、且伴随 `Timeout waiting for PADS packets` → 多半是本机短时间内重拨太频繁，触发了运营商侧的抑制

同时排除本地因素：有没有装 `watchcat`、`mwan3`、自动重拨脚本、定时重启任务。

### 2.2 关键对照：换一台非 OpenWrt 设备

这是最省事的判定方法：**同一根线、同一个账号**，先用一台非 OpenWrt 路由器（原厂固件、光猫自身拨号都行）拨号观察同样时长。

| 对照结果 | 结论 |
|---|---|
| 非 OpenWrt 设备正常，换回 OpenWrt 就复现 | **运营商侧问题**，本地只能规避，见第 6 节 |
| 两台 OpenWrt 设备都断，非 OpenWrt 不断 | 同上 |
| 所有设备都断 | 线路 / 账号 / 光猫问题，先找装维 |

> [!IMPORTANT]
> 如果你观察到的是「**只有 OpenWrt 系设备被周期性断开**」，这大概率不是固件 bug，改插件、改 IPv6 设置、重刷固件都治不好，直接报障给运营商才是正路。

## 3. 已知案例：华为 vBRAS（VNE9000）默认配置

2026 年 10 月，重庆电信辖区内的实测结论（运营商已确认）：

- 辖区内**几乎所有基于 OpenWrt 的路由器**约 3 分钟重拨一次；
- 同期 TP-Link / Netgear / ASUS / 华为 的家用路由**没有**这个问题，部分中兴型号也没有；
- 电信联系华为 L3 定位，结论是 **VNE9000 的默认配置**所致（配置未被改动）；运营商据此出具事故报告。

也就是说：**这是 BRAS 侧的行为，不是 CPE 侧能修好的东西**。

### 想确认自己的 BRAS 是不是华为 vBRAS

PPPoE 发现阶段，AC 会回一个带自己名字的 PADO：

```sh
tcpdump -i eth0 -n -tttt -s 0 2>/dev/null | grep -m1 "AC-Name"
```

看到类似下面的输出，就是华为 vBRAS 平台：

```
PPPoE PADO [Service-Name] [Host-Uniq 0x7D630000] [AC-Name "XXX-BASCP-6.NMAN.VNE9000"]
```

> [!NOTE]
> 抓包一定要在 `eth0`（PPPoE 物理口）上、且**不加过滤器**。`tcpdump -i eth0 "pppoes"` 在部分 libpcap 版本下匹配不到任何包；而拨号接口 `pppoe-wan` 在重拨时会被删除，抓包会中断。

## 4. 为什么 wan6 只有 PD、没有地址

这不是 ImmortalWrt 的问题，是运营商的 DHCPv6 策略。抓包能看到完整原因：

```
客户端 → dhcp6 solicit   (IA_NA) + (IA_PD)
服务器 → dhcp6 advertise (IA_NA ... (status-code NoAddrsAvail))
                         (IA_PD ... 240e:xxxx:xxxx:xxxx::/56 (status-code Success))
```

即：**只给前缀（IA_PD），明确拒绝分配地址（IA_NA）**。

此时 `wan` 的 IPv6 地址只能来自 **RA（路由通告）里的 SLAAC 前缀**：

- 运营商在会话建立后会发 RA，前缀带 `auto` 标志；
- OpenWrt 里由 `odhcp6c` 读到该前缀并交给 `netifd`，加地址时带 `noprefixroute` 标志；
- **内核的 `accept_ra` 不是必需条件**，只要 RA 能到达、并且被应用，地址就会配上。

所以「明明抓到了 RA，但接口上还是没有地址」也是会发生的——取决于 RA 到达的时机与次数。这也是为什么有的人重启一次就有了，有的人一直没有。

对使用的影响：**LAN 上网不受影响**（IPv4 正常、LAN 的 IPv6 靠 PD 下发），但路由器**自身**依赖 WAN 公网地址的应用（DDNS、部分插件）会工作不正常。

## 5. 如果确实需要路由器自身有一个 WAN IPv6 地址

先确认运营商确实在发 RA：

```sh
tcpdump -i any -n -tttt "icmp6 and (ip6[40]==133 or ip6[40]==134)"
```

- 一条 RA 都没有 → 运营商没发，本地怎么配都没用；
- 有 RA 但地址没配上 → 按下面两步处理。

### 5.1 打开 RA 处理（转发场景要用 2）

```sh
sysctl -w net.ipv6.conf.all.accept_ra=2
sysctl -w net.ipv6.conf.default.accept_ra=2
```

要持久化，写进 `/etc/sysctl.conf`（注意 `all.*` 与 `default.*` **两层都要设**，只设 `default` 时新建的 `pppoe-wan` 仍然拿不到）。

### 5.2 RA 不稳定时，用 PD 兜底编址

`/etc/hotplug.d/iface/30-wan6-pdaddr`：等 5 秒还没有地址，就用 PD 里的第一个 /64 给 WAN 编址。

```sh
#!/bin/sh
[ "$INTERFACE" = "wan_6" ] || exit 0
case "$ACTION" in ifup|ifupdate) ;; *) exit 0 ;; esac

sleep 5
ip -6 addr show dev pppoe-wan scope global | grep -q 'inet6 .*scope global' && exit 0

PD=$(ubus call network.interface.wan_6 status 2>/dev/null \
     | sed -n '/"ipv6-prefix"/,/]/p' \
     | sed -n 's/^[[:space:]]*"address": "\([^"]*\)".*/\1/p' | head -1)
[ -n "$PD" ] || exit 0

ip -6 addr replace "$(echo "$PD" | cut -d: -f1-4)::1/64" dev pppoe-wan
```

> [!WARNING]
> **不要**为了「让 RA 更容易来」把 `router_solicitations` 调得很大。实测中，会话建立初期连发 9~11 个 RS 的会话反而更容易被 BRAS 断开；默认的 `3 × 4s` 就够用。

## 6. 报障：普通用户意义有限

> [!WARNING]
> 这一节只对**能直接联系到运营商后台 / 机房**的用户有效（例如企业宽带大客户，或有内部渠道的人）。
>
> 普通用户打 10000、或者找一线装维说「BRAS 周期性断开 OpenWrt 设备」，基本传不到核心网那一层，对方通常只会建议换路由器、查线路、重启光猫——**白耗时间，不建议在这条路上花精力**。

如果能直达后台，可以直接复制下面这段：

> 您好，我的宽带每隔几分钟自动断线重拨一次，间隔很规律。已用其它品牌路由器在**同一条线路、同一个账号**上做过对照：其它品牌正常，只有 OpenWrt 系（ImmortalWrt）路由器会周期性断线。
>
> 断线时路由器日志显示 `LCP terminated by peer`，说明是 BRAS 侧主动终止会话，不是线路质量或设备故障。
>
> 请转核心网 / BRAS 侧核查：接入 AC 标识为 `XXX.VNE9000`（华为 vBRAS）。据华为 L3 反馈，VNE9000 默认配置会导致 OpenWrt 系 CPE 被周期性断开，请协助调整配置。

附上这些信息会更快：

- 断线时间点（连续 5 次以上）；
- 拨号账号；
- 出问题的设备型号 + 固件版本（`cat /etc/openwrt_release`）；
- 对照设备的品牌型号。

### 6.1 只是嫌断线麻烦：可以先关掉 WAN 的 IPv6

如果不想折腾报障，只想先用着，可以先关掉 WAN 侧的 IPv6（很多人的困扰主要来自 IPv6 相关的现象）：

界面：`网络 → 接口 → wan → 高级设置 → IPv6 分配长度 = 已禁用`，并删除 `wan6` 接口。

或者用命令：

```sh
uci set network.wan.ipv6='0'
uci -q delete network.wan6
uci commit network
ifup wan
```

> [!NOTE]
> 这是**规避**，不是修复：断线是否随之消失取决于当地 BRAS 的策略。
> 在本案例的实测中，关闭 IPv6（WAN 长期无地址）后仍然出现过一次周期性重拨，所以**不能保证有效**。
> 如果关闭后依然周期性断线，说明与 IPv6 无关，那就是第 3 节说的 BRAS 侧问题，本地怎么调都没用。

## 7. 几个常见但不建议的排查方向

| 做法 | 为什么没用 |
|---|---|
| 反复 `ifup wan` / `ifdown wan` 观察 | 短时间内大量重拨会触发运营商侧 PPPoE 发现抑制（日志出现 `Timeout waiting for PADS packets`），把现象搞得更乱 |
| 改 `delegate` / `ip6assign` 想让 WAN 拿到地址 | 这两个只决定前缀如何下发给下游，跟 WAN 自身有没有地址无关 |
| 重刷固件 / 换插件版本 | 若属于第 3 节的情况，换版本不会改变 BRAS 的行为 |
| 在 `pppoe-wan` 上抓包看断线 | 重拨瞬间该接口会被删除，抓包随之中断；应抓 `eth0` |

## 8. 相关

- 上游 issue 讨论：[immortalwrt/immortalwrt#2505](https://github.com/immortalwrt/immortalwrt/issues/2505)
- IPv6 地址后缀怎么来的、为什么用它做端口转发：[关于eui64的一些说明](./关于eui64的一些说明.md)
