# Hackintosh-Catalina-hasee-k650d-i7-d3
## 在神舟笔记本上使用Clover引导安装并驱动Mac OS 10.15 Catalina

> i5机型请移步[https://github.com/daggeryu/k650d-i5-d3-clover](https://github.com/daggeryu/k650d-i5-d3-clover)，感谢 @daggeryu

## 电脑配置

| 规格  | 详细信息     |
| ---- | ----------  |
| CPU | Intel Haswell i7-4710MQ |
| 显卡 | Intel HD Graphics 4600 + NVIDIA GeForce GTX 950M（已屏蔽） |
| 声卡 | 瑞昱 VT1802 |
| 网卡 | RealtekRTL8111 + AR5B125（已更换 BCM94352HMB） |
| 硬盘 | ORICO M200 256GB + SanDisk Plus 240G + 希捷酷鱼2T |

## 最终效果
- **显卡**：必须屏蔽独显，日常使用核显hd4600足够了，ig-platform-id为*0x0a260006*,结合WhateverGreen使用WEG自定义补丁修复显存及花屏等
- **声卡**：使用VoodooHDA万能驱动
- **有线网卡**：使用RealtekRTL8111.kext驱动就可以了
- **无线网卡+蓝牙**：自带AR5B125无法驱动也没有蓝牙，更换为Mini pci-e接口的*博通 BCM94352HMB*，需要屏蔽51针脚以及USB端口定制（蓝牙是走内建usb线路），使用AirportBrcmFixup+BrcmFirmwareData+BrcmPatchRAM3+BrcmBluetoothInjector驱动，可完美使用AirDrop HandOff SideCar等

## References
- https://blog.daliansky.net/Common-problems-and-solutions-in-macOS-Catalina-10.15-installation.html
- https://segmentfault.com/a/1190000020642944?utm_source=tag-newest#articleHeader2
- https://zuiyu1818.cn/posts/Hac_Advanced.html
- https://www.tonymacx86.com/threads/guide-installing-3rd-party-kexts-el-capitan-sierra-high-sierra-mojave-catalina.268964/
- https://github.com/daggeryu/k650d-i5-d3-clover
- http://bbs.appleosx.cn/forum.php?mod=viewthread&tid=108
- https://www.seagate.com/cn/zh/support/downloads/item/ntfs-driver-for-mac-os-master-dl/
- https://mwholt.blogspot.com/2012/09/fix-home-and-end-keys-on-mac-os-x.html
- https://www.itpwd.com/268.html