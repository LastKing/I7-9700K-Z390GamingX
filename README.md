# 安装

当前使用`oc0.8.5`  macOs版本`13.0.0`

警告：此efi定制了 技嘉z390-GamingX 的usb，如果不是此主板请自行定制

## Kext

1. lilu 一个开放源码内核扩展为 macOS 系统中的 kext、库和程序补丁提供了一个平台
2. ViturlSMC 先进的苹果 SMC 仿真器的内核。完全运转需要 Lilu。
   1. SMCProcessor 给 Penryn CPU 或以上提供温度传感器支持
   2. SMCSuperIO 风扇信息读取
   3. SMCLightSensor 通过新的 SMC 事件 API，是一个光线传感器的例子 (需要 ACPI0008_ALI)
   4. SMCBatteryManager 添加 SMC 跟 SMBus 协议完整的 AppleSmartBattery 模拟层，电池相关的传感器
3. WhateverGreen Lilu 插件，提供在 macOS 上选择 gpu 的补丁。需要 Lilu 1.4.0 或更新版本。
4. AppleALC 一个开放源码的内核扩展，允许本地 macOS HD 音频没有正式支持的编解码器，没有任何文件系统修改。
5. Ethernet
   1. IntelMausi  ** 选择这个
   2. SmallTreeIntel82576 kext
   3. AtherosE2200Ethernet
   4. RealtekRTL8111
   5. LucyRTL8125Ethernet

## Driver

1. OpenRuntime.efi 替换 AptioMemoryFix.efi 公司，用作 OpenCore 的扩展，以帮助修补启动.efi 用于 NVRAM 修复和更好的内存管理。
2. HfsPlus.efi 需要查看 HFS 卷（即 macOS 安装程序和恢复分区/映像）。不要混用其他 HFS 驱动程序

- `EFI/BOOT/BOOTx64.efi` 电脑启动的时候支持 UEFI 的主板会去读取这个文件
- `EFI/OC` OpenCore 存放目录
  - `ACPI` 存放自定义的 ssdt aml 文件, 比如 (aml 为二进制文件, 是我们需要的最终文件. dsl 为源文件, 需要用 [MaciASL](https://github.com/acidanthera/MaciASL/releases)打开另存为 aml.)
    - [`SSDT-PLUG.dsl`](https://github.com/acidanthera/OpenCorePkg/tree/master/Docs/AcpiSamples) 开启硬件变频功能, 作用于 CPU, iGPU, dGPU. 需要自行编译为 aml
    - [`SSDT-AWAC.dsl`](https://github.com/acidanthera/OpenCorePkg/tree/master/Docs/AcpiSamples) 300 系列的主板用最近更新的 BIOS 后, RTC 失效, 这个 SSDT 作用是启用 RTC, 需要自行编译为 aml
    - [`SSDT-EC-USBX.dsl`](https://github.com/acidanthera/OpenCorePkg/tree/master/Docs/AcpiSamples) Fake EC 和 USBX, 给 macOS 提供一个虚假的 EC 设备, 同时提供 USB 大电流支持, 需要自行编译为 aml
  - `Drivers` 存放文件系统驱动文件, 比如
    - [`ApfsDriverLoader.efi`](https://github.com/acidanthera/AppleSupportPkg/releases) 用于加载 macOS 内置的 apfs.efi , 读取 APFS 分区, **必要文件**
    - [`FwRuntimeServices.efi`](https://github.com/acidanthera/OpenCorePkg/releases) 提供模拟 nvram 等其他功能, **必要文件**
    - [`HFSPlus.efi`](EFI/OC/Drivers/HFSPlus.efi) 提供 HFS+文件系统的支持, 读取 macOS 的安装 U 盘以及 Recovery 分区需要此 efi, **必要文件**
  - `Kexts` 存放各种设备和硬件的驱动或者补丁.
    - [`Lilu.kext`](https://github.com/acidanthera/Lilu/releases) 一个框架式的 kext,自身单独使用没有作用, 是其他 kext 的依赖, 必须第一个被加载. **必要文件**
    - [`AppleALC.kext`](https://github.com/acidanthera/AppleALC/releases) 让 macOS 可以正确识别大部分主板上的集成声卡
    - [`VirtualSMC.kext`](https://github.com/acidanthera/VirtualSMC/releases) 模拟 SMC, **必要文件**
    - [`WhateverGreen.kext`](https://github.com/acidanthera/WhateverGreen/releases) 解决集成/独立显卡的各种问题 **必要文件**
    - 其他的几个 kext, 根据需要使用, 比如:
    - [`IntelMausi.kext`](https://github.com/acidanthera/IntelMausi/releases), Intel 有线网卡驱动.
    - [`RealtekR1000SL.kext`](https://github.com/SergeySlice/RealtekLANv3/releases), Realtek 有线网卡驱动
    - [`SMCProcessor.kext SMCSuperIO.kext`](https://github.com/acidanthera/VirtualSMC/releases) 让 macOS 下的监控软件可以读取主板上的传感器信息温度,频率等
  - `Tools` 工具类 efi, 这些工具在 OpenCore 启动界面可以看到, 目前只有下面 2 个工具, **不可以放入 Drivers 文件夹**
    - ~~`CleanNvram.efi`~~ 作用为清空 nvram, 新版本 OpenCore 已经用内置的选项 AllowNvramReset=YES 取代这个 efi, 等效于启动到 macOS 恢复模式之后, 运行 `nvram -c`
    - [`Shell.efi`](https://github.com/acidanthera/OpenCoreShell/releases) 一个修改版的 `UEFI SHELL`, 可以做很多有趣的事情.
    - [`VerifyMsrE2`](https://github.com/acidanthera/OpenCorePkg/releases), 检查主板是否有 CFG LOCK
  - `OpenCore.efi` OpenCore 的主引导文件
  - `config.plist` OpenCore 的主要配置文件, 可以使用 PlistEdit Pro 或者 ProperTree 可视化编辑.

boot-args = -v keepsyms=1 debug=0x100 agdpmod=pikera alcid=1

## 自身配置 信息记录

1. i7-9700k (coffee lake)
2. 蓝宝石 Rx550xt
3. 有线网卡 Intel I219V7
4. 板载声卡 ALC892


1. 用新版的OpenCore.efi替换你现在的
2. 用新版的BOOTX64.efi替换你现在的
3. 阅读文档里的Differences.pdf，并对你的config.plist做出相应更改（有时直接去diff新旧两个Sample.plist会更方便一些，但你会不理解每处更改有什么用）
4. Acidanthera通常会同时释出新版的OpenCore和全套Kext。（通常于每月月初）所以每次请检查Acidanthera的Github组织首页，下载所有更新了的Kext，并替换掉你原来所有用到的。
同时，别忘了更新OpenRuntime。它会随着OpenCore的Release zip一同释出。

## 参考：

1. [oc-guide](https://github.com/cattyhouse/oc-guide)
2. [oc 引导黑苹果](https://iiong.com/booting-hackintosh-with-opencore/)
3. [使用 ssdtPRGen.sh 生成处理器变频配置 SSDT.aml --- 自己未实验](https://heipg.cn/tutorial/using-ssdtprgen-generate-ssdt-aml.html)
4. [OpenCore 入门配置构建引导详细使用说明 OC 引导完整教程](http://imacos.top/2020/04/04/1616/)
5. [黑苹果 DIY 各机型 EFI 引导文件集合，含 Clover 和 OpenCore，长期更新](https://heipg.cn/clover/diy-hackintosh-efi.html)
6. [使用 OpenCore 引导黑苹果](https://blog.xjn819.com/post/opencore-guide.html)
