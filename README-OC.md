# Hackintosh-Catalina-hasee-k650d-i7-d3
更多详情请查阅[神舟k650d i7 d3安装和使用黑苹果10.15 Catalina - s1973](https://s1973.top/blog/0015730470006545b49e611cb91470daaacb539b7034eae000)

## to do
1. ~~刚开机卡顿~~
2. ~~声卡异常~~
3. ~~键盘触摸板驱动异常~~
4. ~~USB定制~~

## Tips
- **PS2键盘和触控板** 使用 ApplePS2SmartTouchPad ,注意在opencore里需要加载3项
- **声卡** 使用 AplleALC 驱动，但需要修复IRQ否则panic，直接使用ssdt-hpet_rtc_timr-fix三件套
- **电源管理** 使用SMCBatteryManager以及ssdt-lpc ssdt-plug加载

## About Upgrade to Mac OS BigSur
- **博通系网卡失效且开机 IOKit Daemon stall[0] PXSX** 这是因为bigsur移除了AirPortBrcm4360驱动，导[AirportBrcmFixup](https://github.com/acidanthera/AirportBrcmFixup)注入冲突，新版已经单独分离4360injector并以插件加载，我们这里给4360injector加上maxkernel 19.9.9限制其在10.15以上系统的加载即可。
    > In 11 (Big Sur) class AirPortBrcm4360 has been completely removed. Using of injector kext with such class name and matched vendor-id:device-id blocks loading of original airport kext. To address this issue and keep compatibility with older systems injectors for AirPortBrcm4360 and AirPortBrcmNIC were removed from main Info.plist file. Instead, the two new kext injectors are deployed in PlugIns folder: AirPortBrcm4360_Injector.kext and AirPortBrcmNIC_Injector.kext. You have to block (or remove) AirPortBrcm4360_Injector.kext in BigSur. In OpenCore you can specify MaxKernel 19.9.9 for AirPortBrcm4360_Injector.kext. In Clover you can have two different AirportBrcmFixup.kext, but in kext folder with version name 11 AirportBrcmFixup.kext must not contain AirPortBrcm4360_Injector.kext. You don't need these injectors at all if your vendor-id:device-id is natively supported by AirPortBrcmNIC or AirPortBrcm4360 (your device-id is included into Info.plist in these kexts).
- **对AppleIntelFramebufferAzul的热替换补丁无法生效** 参见[Failure with com.apple.iokit.IONDRVSupport patching
](https://github.com/acidanthera/bugtracker/issues/1244)
    > Updating kextcache will not bring it there as the kernel caches are now separated in 3 files (kernel collections): boot kernel extensions, auxiliary kernel extensions, and user kernel extensions. First two are built by Apple and are never modified by the end user. Last two are loaded by the kernel and thus are unavailable during boot. All GPU drivers belong to auxiliary kernel collection, making the only possible way to patch them is via Lilu and friends.
    > 
    > The line you mentioned relates to kernel injection in regards to kernel collection header size reservation, so it is rather WEG.  
    > by vit9696

    因此我们这里借助whatevergreen的WEG补丁通过lilu来修复开机鼠标卡顿的问题。

    | key  | value     |
    | ---- | ----------  |
    | framebuffer-con1-enable  | 01000000     |  
    | framebuffer-con1-alldata | 02040900 00080000 87000000 FF000000 01000000 40000000   |

## ACPI Table
| Table  | Comment     |
| ---- | ----------  |
| **SSDT-PLUG-_PR.CPU0.aml** | XCPM CPU battery manager |
| **SSDT-SBUS.aml** | SBUS patch |
| **SSDT-LPC.aml** | Load AppleLPC battery manager |
| **SSDT-PNLF.aml** | Set Intelbacklight brightness |
| **SSDT-HPET_RTC_TIMR-fix.aml** | Audio IRQ fix |

## References
- https://blog.skk.moe/post/from-clover-to-opencore/
- https://oc.skk.moe/
- https://blog.xjn819.com/post/opencore-guide.html
- https://blog.daliansky.net/OpenCore-BootLoader.html
- https://ocbook.tlhub.cn/
- https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html
- https://github.com/zhutuizhi/k650d-i5-d3-opencore
- https://kirainmoe.com/blog/post/opencore-migration-experience/