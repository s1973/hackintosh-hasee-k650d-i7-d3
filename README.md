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

## 硬件驱动情况
- **显卡**：必须屏蔽独显，日常使用核显hd4600足够了，*ig-platform-id*为`0x0a260006`,仿冒*device-id*为`0x12040000`,结合WhateverGreen使用WEG自定义补丁修复显存 花屏及HDMI等，开机8苹果及花屏请启用bios里的csm功能，尤其需要注意配置*framebuffer-cursormem*属性，否则容易出现第三方app花屏卡死的情况
- **声卡**：使用VoodooHDA万能驱动
- **有线网卡**：使用RealtekRTL8111.kext驱动就可以了
- **无线网卡+蓝牙**：自带AR5B125无法驱动也没有蓝牙，更换为Mini pci-e接口的*博通 BCM94352HMB*，需要用胶带屏蔽51针脚以及USB端口定制（蓝牙模块走usb总线），使用AirportBrcmFixup+BrcmFirmwareData+BrcmPatchRAM3+BrcmBluetoothInjector驱动，可完美使用AirDrop HandOff SideCar等
- **睡眠+休眠**：正常，可以正常休眠及键鼠唤醒，此类问题一般都可以通过成功驱动电源管理，SSDT变频，显卡及USB端口定制解决
- **SSDT**：CPU变频正常，推荐[`CPU-S`](http://bbs.pcbeta.com/viewthread-1698338-1-1.html)软件根据实际测试CPU运行频率生成ssdt
- **USB**：使用usbinjectall配合hackintool进行端口定制，同时禁用clover里USB端口限制补丁，可正常使用USB
- **内置键盘+触控板**：使用ApplePS2SmartTouchPad驱动
- **刚开机时系统卡顿**：需要屏蔽多余的hdmi端口0105，在clover里打如下内核和驱动补丁：
    ```
    Name：AppleIntelFramebufferAzul
    Find：01050900 00040000 87000000
    replace：FF000900 00040000 87000000
    ```
- **内置键盘command和option键错位**：使用[Karabiner](https://pqrs.org/osx/karabiner/)软件更改按键映射
- **无法正常使用键盘上的home+end等键**：使用*Karabiner*或者在终端里输入以下命令保存文件重启电脑
    ```
    $ cd ~/Library
    $ mkdir KeyBindings
    $ cd KeyBindings
    $ vim DefaultKeyBinding.dict
    {
    /* Remap Home / End keys to be correct */
    "\UF729" = "moveToBeginningOfLine:"; /* Home */
    "\UF72B" = "moveToEndOfLine:"; /* End */
    "$\UF729" = "moveToBeginningOfLineAndModifySelection:"; /* Shift + Home */
    "$\UF72B" = "moveToEndOfLineAndModifySelection:"; /* Shift + End */
    "^\UF729" = "moveToBeginningOfDocument:"; /* Ctrl + Home */
    "^\UF72B" = "moveToEndOfDocument:"; /* Ctrl + End */
    "$^\UF729" = "moveToBeginningOfDocumentAndModifySelection:"; /* Shift + Ctrl + Home */
    "$^\UF72B" = "moveToEndOfDocumentAndModifySelection:"; /* Shift + Ctrl + End */
    }
    ```

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