## TP-LINK TL-WDN5200H USB无线网卡驱动

驱动来源: [TP-LINK官方网站](https://service.tp-link.com.cn/detail_download_8874.html)

---

fatal error: net/ipx.h: 没有那个文件或目录
net/ipx.h在Linux 5.15-rc1中已经被删除, 解决方法: 去除相关代码. [参考](https://github.com/aircrack-ng/rtl8188eus/pull/146/files)


