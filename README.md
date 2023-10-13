## TP-LINK TL-WDN5200H USB无线网卡驱动

驱动来源: [TP-LINK官方网站](https://service.tp-link.com.cn/detail_download_8874.html)

---

手上有一台J1900 CPU的小主机, 一时兴起, 安装了Debian 12 (Linux 6.1.0-12-amd64), 因为交换机没网口了, 正好有一个空闲的USB无线网卡 TL-WDN5200H 给装了上去, 随即发现无法使用. 

查了一下, 原来是缺少网卡驱动, 按照教程使用DKMS安装了一遍, 不知道为什么还是不行. 于是在Gayhub又找了一个, 尝试编译, 收获一堆错误. 

再次搜索发现TP-LINK有提供了Linux驱动, 于是下载后再次尝试编译, 失败. 不过也在意料之中, 毕竟人家说明文档明确写了最高支持Linux 5.8, 并且在readme.txt中贴心的给出了编译出错后的解决方法:

> 运行install.sh，即会自动编译和安装网卡驱动。
> 
> 若期间出现错误，请根据报错信息进行排查。


既然如此, 只能自己动手了, 编译后根据出错提示, 一个个解决: 

> fatal error: net/ipx.h: 没有那个文件或目录

net/ipx.h在Linux 5.15-rc1中已经被删除, 解决方法: 去除相关代码. [参考](https://github.com/aircrack-ng/rtl8188eus/pull/146/files)

> error: unknown type name ‘mm_segment_t’

> error: implicit declaration of function ‘get_fs’

> error: ‘KERNEL_DS’ undeclared

这一类错误实际上都与 set_fs() 有关, 而set_fs()已经在Linux 5.19之后被删除. [这个issue](https://github.com/coolsnowwolf/lede/issues/9170)提到可以删除set_fs()相关调用, 正好此源码中确实也是使用kernel_read()的, 故加了个条件宏, 去掉相关代码.


> error: implicit declaration of function ‘prandom_u32’

又是一个从内核中删除的函数, 用 get_random_u32() 替换.

...

写了一堆 `#if (LINUX_VERSION_CODE >= KERNEL_VERSION()` 终于将错误解决完了, 当然警告还是有的, 但真男人从来不管警告:)

按照官方文档的说明, 执行命令:

> `sudo bash ./install.sh`

驱动顺利安装完成.

接下来重启系统, 然后使用`lsusb`命令再次查看, 发现设备已经被识别:

> Bus 003 Device 007: ID 31b2:0010 KTMicro KT USB Audio

> Bus 001 Device 006: ID 30fa:0300  USB OPTICAL MOUSE

> Bus 001 Device 005: ID 1a2c:0e24 China Resource Semico Co., Ltd USB Keyboard

> Bus 001 Device 004: ID 0424:2514 Microchip Technology, Inc. (formerly SMSC) USB 2.0 Hub

> *Bus 001 Device 003: ID 0bda:1a2b Realtek Semiconductor Corp. RTL8188GU 802.11n WLAN Adapter (Driver CDROM Mode)*

> Bus 001 Device 002: ID 8087:07e6 Intel Corp.

> Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

不过此时网卡工作在CDROM模式.

使用命令 

`sudo usb_modeswitch -KW -v 0bda -p 1a2b`

切换模式, 然后拔下网卡再次插上, 网卡就可以正常工作了. 

使用ifconfig:

> wlx6cb1583f05b7: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500

>         inet 192.168.6.107  netmask 255.255.255.0  broadcast 192.168.6.255

>         inet6 fe80::6762:951f:30b0:af5a  prefixlen 64  scopeid 0x20<link>

>         ether 6c:b1:58:3f:05:b7  txqueuelen 1000  (Ethernet)

>         RX packets 82  bytes 12361 (12.0 KiB)

>         RX errors 0  dropped 0  overruns 0  frame 0

>         TX packets 32  bytes 5224 (5.1 KiB)

>         TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

