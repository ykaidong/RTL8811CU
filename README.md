## TP-LINK TL-WDN5200H USB无线网卡驱动

驱动来源: [TP-LINK官方网站](https://service.tp-link.com.cn/detail_download_8874.html)

---

fatal error: net/ipx.h: 没有那个文件或目录
net/ipx.h在Linux 5.15-rc1中已经被删除, 解决方法: 去除相关代码. [参考](https://github.com/aircrack-ng/rtl8188eus/pull/146/files)

error: unknown type name ‘mm_segment_t’
error: implicit declaration of function ‘get_fs’
error: ‘KERNEL_DS’ undeclared
这一类错误实际上都与 set_fs() 有关, 而set_fs()已经在Linux 5.19之后被删除. [这个issue](https://github.com/coolsnowwolf/lede/issues/9170)提到可以删除set_fs()相关调用, 正好此源码中确实也是使用kernel_read()的, 故加了个条件宏, 去掉相关代码.

error: implicit declaration of function ‘prandom_u32’
又是一个从内核中删除的函数, 用 get_random_u32() 替换.



