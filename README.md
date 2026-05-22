# Hackintosh Hasee K650D i7-D3

在神舟 K650D i7-D3 笔记本上使用 OpenCore 引导安装 macOS。

当前配置已在 **macOS 15.7.7 Sequoia** 上完美运行，使用 **OpenCore 1.0.7** + **OCLP**（OpenCore Legacy Patcher）进行驱动补丁。macOS 26 Tahoe 可安装启动，除显卡（HD4600 暂无 OCLP Patch）外基本可用。

> i5 机型请移步 [k650d-i5-d3-clover](https://github.com/daggeryu/k650d-i5-d3-clover)

## 电脑配置

| 规格 | 详细信息 |
| ---- | -------- |
| CPU | Intel Haswell i7-4710MQ |
| 显卡 | Intel HD Graphics 4600 + NVIDIA GeForce GTX 950M（已屏蔽） |
| 声卡 | VIA VT1802P |
| 有线网卡 | Realtek RTL8111 |
| 无线网卡 | BCM94352HMB（原装 AR5B125 无法驱动，已更换） |
| 硬盘 | ORICO M200 256GB + SanDisk Plus 240G + 希捷酷鱼 2T |

## 硬件驱动情况

- **显卡**：屏蔽独显，使用核显 HD4600。`ig-platform-id` 为 `0x0a260006`，仿冒 `device-id` 为 `0x12040000`。使用 WhateverGreen 1.7.1d7（定制版）修复显存、花屏及 HDMI，并兼容 macOS 26 Tahoe 的安装和启动。需配置 `framebuffer-cursormem` 属性防止第三方 App 花屏卡死。macOS 15+ 需通过 OCLP 打 HD4600 显卡补丁
- **声卡**：使用 AppleALC 仿冒内建声卡，`layout-id` 为 `3`。macOS 15 Sequoia 原生支持无需额外操作；macOS 26 Tahoe 需通过 OCLP 打 AppleALC Patch 恢复声音
- **有线网卡**：使用 RealtekRTL8111.kext 驱动，DeviceProperties 中设置 `built-in` 改善 iMessage 登录兼容性
- **无线网卡**：BCM94352HMB（Mini PCIe 接口，需屏蔽 51 针脚）。使用 AirportBrcmFixup 配合 OCLP 打补丁进行仿冒驱动。`AirPortBrcm4360_Injector` 必须通过 MaxKernel 禁用，否则高版本系统开机卡顿。DeviceProperties 中设置 `built-in` 并关闭 `pci-aspm`
- **蓝牙**：使用 BrcmFirmwareData + BrcmPatchRAM3 + BlueToolFixup 驱动。通过 boot-args 和 NVRAM 变量进行仿冒，跳过系统硬件匹配验证，正常驱动非原生蓝牙模块。可完美使用 AirDrop、Handoff、Sidecar
- **键盘+触控板**：使用 VoodooPS2Controller 驱动（ApplePS2SmartTouchPad 在新系统上已不兼容）
- **电池**：使用 SMCBatteryManager（替代旧的 ACPIBatteryManager）
- **睡眠**：正常休眠及键鼠唤醒，通过 `darkwake=0` 禁用 Dark Wake
- **USB**：通过 USBPorts.kext 端口定制 + ACPI 补丁（EHC1→EH01, EHC2→EH02）改善兼容性
- **读卡器**：RTL8411B，使用 Sinetek-rtsx 驱动（当前未启用）

## OpenCore 配置说明

### ACPI

| Table | 说明 |
| ----- | ---- |
| SSDT-PLUG-_PR.CPU0.aml | CPU 电源管理 |
| SSDT-SBUS.aml | SBUS 补丁 |
| SSDT-LPC.aml | AppleLPC 电源管理 |
| SSDT-PNLF.aml | 屏幕亮度调节 |
| SSDT-HPET_RTC_TIMR-fix.aml | 声卡 IRQ 修复 |

DSDT 补丁：
- `Rename EHC1 to EH01` — USB 控制器重命名，Sonoma 之后系统 USB 变动需要此补丁
- `Rename EHC2 to EH02` — 同上

### DeviceProperties

| 设备路径 | 用途 |
| -------- | ---- |
| PciRoot(0x0)/Pci(0x2,0x0) | HD4600 显卡注入，ig-platform-id、device-id、framebuffer 补丁 |
| PciRoot(0x0)/Pci(0x1B,0x0) | 声卡 layout-id 注入 |
| PciRoot(0x0)/Pci(0x1C,0x2)/Pci(0x0,0x0) | BCM94352HMB WiFi，设置 built-in + 关闭 pci-aspm |
| PciRoot(0x0)/Pci(0x1C,0x3)/Pci(0x0,0x1) | RTL8111 有线网卡，设置 built-in 改善 iMessage 兼容 |

### Kernel

关键 Kext 说明：

| Kext | 说明 |
| ---- | ---- |
| Lilu | 内核补丁框架，所有其他补丁 kext 的依赖 |
| VirtualSMC + 插件 | SMC 仿冒（SMCBatteryManager、SMCProcessor、SMCSuperIO） |
| WhateverGreen 1.7.1d7 (MOD) | 显卡补丁，使用定制版以兼容 macOS 26 Tahoe 的安装和启动 |
| AppleALC | 声卡驱动 |
| VoodooPS2Controller | 键盘和触控板驱动 |
| CPUFriend + DataProvider | CPU 调频优化 |
| AirportBrcmFixup | WiFi 仿冒，配合 OCLP 使用 |
| BrcmFirmwareData + BrcmPatchRAM3 | 蓝牙固件加载 |
| BlueToolFixup | 蓝牙注入（替代旧的 BrcmBluetoothInjector） |
| AMFIPass | OCLP 用，绕过 AMFI 验证（MinKernel 23.0.0，仅 Sonoma+） |
| RestrictEvents | 屏蔽系统不兼容提示 |
| IOSkywalkFamily | OCLP WiFi 补丁，替换系统原生 Skywalk |
| IO80211FamilyLegacy | OCLP WiFi 补丁，提供旧版无线驱动框架 |

Kernel Block：
- `com.apple.iokit.IOSkywalkFamily` — 屏蔽系统原生 Skywalk，使用补丁版替代

### NVRAM (boot-args)

```
keepyms=1 debug=0x100 darkwake=0 -amfipassbeta revpatch=sbvmm ipc_control_port_options=0 -brcmfxbeta brcmfx-country=#a -btlfxboardid
```

| 参数 | 说明 |
| ---- | ---- |
| keepyms=1 | Panic 时保留内核符号，停在报错位置而非重启 |
| debug=0x100 | 内核 Panic 时不自动重启，方便调试 |
| darkwake=0 | 禁用 Dark Wake |
| -amfipassbeta | 配合 AMFIPass.kext，OCLP 打补丁必需 |
| revpatch=sbvmm | 配合 RestrictEvents，OCLP 打补丁必需 |
| ipc_control_port_options=0 | macOS 15+ 显卡补丁必需，OCLP 修补 HD4600 依赖此参数 |
| -brcmfxbeta | 博通 WiFi 仿冒参数 |
| brcmfx-country=#a | 解锁 WiFi 地区限制 |
| -btlfxboardid | 蓝牙仿冒，跳过 Board ID 检查 |

NVRAM 蓝牙变量：
- `bluetoothExternalDongleFailed` + `bluetoothInternalControllerInfo` — 覆盖原始硬件信息，跳过系统设备匹配验证。若不设置，系统检测到仿冒信息不匹配会写入 Flag 中断蓝牙启动

### Security

- **SecureBootModel**: Disabled — OCLP 必需
- **ScanPolicy**: 扩大扫描范围，识别 USB 安装盘中的 Install macOS

## 系统兼容性

| macOS 版本 | 状态 |
| ---------- | ---- |
| macOS 12 Monterey | 原生支持，无需 OCLP |
| macOS 13 Ventura | 需要 OCLP |
| macOS 14 Sonoma | 需要 OCLP |
| macOS 15 Sequoia | 需要 OCLP，当前完美运行于 15.7.7 |
| macOS 26 Tahoe | 可安装启动，但 OCLP 尚未发布 HD4600 显卡 Patch，仅 VESA 模式（可做无头服务器），声卡需 OCLP 补丁 |

## 版本历史

| Tag | 说明 |
| --- | ---- |
| oc-v2.0 | OpenCore 1.0.7，完美支持 macOS 15，除显卡外支持 macOS 26 |
| oc-monterey-native-final | macOS Monterey 原生支持的最终完善版本 |
| oc-v1.2 | OC config 更新，启用 CPUFriend 等 |
| oc-v1.1 | OpenCore 早期版本 |
| oc-v1.0 | 首次迁移至 OpenCore |
| clover-v1.1 | Clover 引导 |
| clover-v1.0 | Clover 引导初始版本 |

## References

- https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/haswell.html
- https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.cn.md
- https://github.com/acidanthera/AppleALC/wiki/Installation-and-usage
- https://github.com/acidanthera/AirportBrcmFixup
- https://dortania.github.io/OpenCore-Post-Install/
- https://github.com/dortania/OpenCore-Legacy-Patcher
- https://github.com/YBronst/OCLP-Plus
- https://www.insanelymac.com/forum/topic/362543-the-latest-the-oclp-plus-3x-tahoe-patch-set/
- https://github.com/laobamac/OCLP-Mod
