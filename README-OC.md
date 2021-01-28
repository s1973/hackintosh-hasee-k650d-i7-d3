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


## References
- https://blog.skk.moe/post/from-clover-to-opencore/
- https://oc.skk.moe/
- https://blog.xjn819.com/post/opencore-guide.html
- https://blog.daliansky.net/OpenCore-BootLoader.html
- https://ocbook.tlhub.cn/
- https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html
- https://github.com/zhutuizhi/k650d-i5-d3-opencore
- https://kirainmoe.com/blog/post/opencore-migration-experience/