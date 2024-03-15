# immortalwrt 常见问题指北

**这是一篇常见问题指南，帮助你更好的使用immortalwrt。**

### 1. 为什么我用不了SSRP

在23.0版本之后我们使用了nftable替换了iptable，所以一些使用iptables的插件无法使用了。

### 2. 我的passwall有问题

Passwall的使用问题并不在本群的支持范围，请访问passwall社区获取更多帮助。<https://github.com/xiaorouji/openwrt-passwall/issues>

### 3. 我的设备莫名重启

检查一下您是否有使用SFE转发加速，如果有请禁用并重新安装一个不带SFE的固件。

### 4. 设备能获取到IPV6但是无法通过IPV6上网

请不保留配置完全干净的重新刷入一个不包含任何MWAN和多拨插件的固件。

### 5. 如何优雅的使用IPV6？

如图所示  
![Alt text](/img/image.png)
![Alt text](/img/image-1.png)
### 6. IPV6如何正确配置端口转发？

IPV6并不需要您去固定设备的地址只需要在防火墙-通信规则里面添加一个包含设备eui64地址的转发就可以了，之后就可以通过你的前缀+eui64地址+端口（可选）访问您的设备

![Alt text](/img/image-2.png)

关于eui64的一些说明 <https://github.com/immortalwrt/user-FAQ/blob/main/%E5%85%B3%E4%BA%8Eeui64%E7%9A%84%E4%B8%80%E4%BA%9B%E8%AF%B4%E6%98%8E.md>

### 7. 我应当在路由器上开启bbr吗？

不应当

### 8. 我的米家设备使用不正常了

请将您路由器上面的广告拦截插件把小米相关的规则设置为放行。

### 9. 我编译出错了

我们不建议新手用户自行编译使用immortalwrt。请使用imagebulider在线构建:<https://fwselector.kyarucloud.moe>   
具体使用教程请参阅<https://github.com/1715173329/blog/issues/9>

### 10. 我家里的苹果设备耗电快

请检查家里是否有使用小米路由器，如果有请替换掉。之后检查是否有开启BBR如果有请关闭。最后在所有带分流功能的插件里将苹果有关的服务全部设置为直连。

### 11. 在一些网站上它显示我的DNS泄露了我需要做任何措施吗？

您不需要对所谓的DNS泄露做任何处理，出现所谓的泄露情况证明您的分流插件是正常工作的。

### 12. 如何开启UPNP

请确保您拥有公网IPV4地址的情况下在服务-UPNP里勾选 启动 UPnP 与 NAT-PMP 服务 启用之后列表里不显示已经打洞成功的程序属正常现象。

### 13. 为什么我找不到wireguard了

在新版本的内核中，luci-app-wireguard已被合并为内核模块，请在编译/构建的时候添加以下软件包
```
wireguard-tools
kmod-wireguard
luci-proto-wireguard
```

### 14. 当Wireguard的服务器端是动态IP我应该怎么配置自动更新对端IP呢？

客户端配置好wireguard之后在 **系统-计划任务** 里面添加
```
*/2 * * * * /usr/bin/wireguard_watchdog
```
