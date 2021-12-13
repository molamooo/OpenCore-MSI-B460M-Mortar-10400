简单的经验总结。

## 镜像下载

Main reference: [url](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/mac-install.html#downloading-macos-modern-os)

```sh
mkdir -p ~/macOS-installer
cd ~/macOS-installer
curl https://raw.githubusercontent.com/munki/macadmin-scripts/main/installinstallmacos.py > installinstallmacos.py
sudo python installinstallmacos.py
```

可能需要从其他途径下载这边不提供的比较新的版本。

## 准备USB drive

将U盘格式化为GUID分区表格式并创建`Mac OS Extended (Journaled)`分区。

Next run this command: 

```
sudo <Location of your downloaded image>/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/<Your created partition>
```

接下来就是折腾EFI的步骤了。大体思路是把GUID分区中的EFI分区想办法挂载进来，然后放入opencore以及需要的文件。

## EFI Preparation

首先可以使用这个工具 [url](https://github.com/corpnewt/MountEFI) 来强行将EFI分区挂载进来折腾。

Main reference: [url](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/opencore-efi.html)

下载 OpenCore 的 Debug Release，放入EFI分区并保留一些必要文件：

```
.
├── EFI
│   ├── BOOT
│   │   └── BOOTx64.efi           <- important
│   └── OC
│       ├── ACPI
│       ├── Drivers
│       │   └── OpenRuntime.efi   <- important
│       ├── Kexts
│       ├── OpenCore.efi          <- important
│       ├── Resources
│       │   ├── Audio
│       │   ├── Font
│       │   ├── Image
│       │   └── Label
│       ├── Tools
│       │   └── OpenShell.efi     <- optional
├── Guide.md
└── README.md
```

> 注意，需要将整个EFI文件夹放入EFI分区中。即，在macos中看到的目录应该是 `/Volumes/EFI/EFI/OC/...`


### 补充需要的 Driver/SSDT/Kext

Main Reference [url](https://dortania.github.io/OpenCore-Install-Guide/ktext.html)

日后更新需要check一下他们的版本。

* HfsPlus [Important] [url](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi) (now up to 12.0.1?)
* VirtualSMC [Important] [url](https://github.com/acidanthera/VirtualSMC/releases)
* Lilu [Important] [url](https://github.com/acidanthera/Lilu/releases)
* WhateverGreen [Important] [url](https://github.com/acidanthera/WhateverGreen/releases)

* AppleALC [Onboard Audio] [url](https://github.com/acidanthera/AppleALC/releases)

* Ethernet kexts. Mine is LucyRTL8125Ethernet [url](https://www.insanelymac.com/forum/topic/343542-lucyrtl8125ethernetkext-for-realtek-rtl8125/)
* XHCI-unsupported [url](https://github.com/RehabMan/OS-X-USB-Inject-All)

* NVMeFix [url](https://github.com/acidanthera/NVMeFix/releases)

* SMCProcessor [Optional]
* SMCSuperIO [Optional]

### SSDT

需要什么SSDT取决于你的硬件平台。参考上面的 Main Referece。笔者的平台是10代i5(Comet Lake)，需要下述文件：

* SSDT-PLUG [url](https://dortania.github.io/Getting-Started-With-ACPI/Universal/plug.html)[prebuild](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-PLUG-DRTNIA.aml)
* SSDT-EC-USBX [url](https://dortania.github.io/Getting-Started-With-ACPI/Universal/ec-fix.html)[prebuild](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-EC-USBX-DESKTOP.aml)
* SSDT-AWAC [url](https://dortania.github.io/Getting-Started-With-ACPI/Universal/awac.html)[prebuild](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-AWAC.aml)
* SSDT-RHUB [url](https://dortania.github.io/Getting-Started-With-ACPI/Universal/rhub.html)[prebuild](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-RHUB.aml)

收集完成后，笔者的EFI文件夹如下：
```
.
├── EFI
│   ├── BOOT
│   │   └── BOOTx64.efi
│   └── OC
│       ├── ACPI
│       │   ├── SSDT-AWAC.aml
│       │   ├── SSDT-EC-USBX-DESKTOP.aml
│       │   ├── SSDT-PLUG-DRTNIA.aml
│       │   └── SSDT-RHUB.aml
│       ├── Drivers
│       │   ├── HfsPlus.efi
│       │   └── OpenRuntime.efi
│       ├── Kexts
│       │   ├── AppleALC.kext
│       │   ├── Lilu.kext
│       │   ├── LucyRTL8125Ethernet.kext
│       │   ├── NVMeFix.kext
│       │   ├── SMCProcessor.kext
│       │   ├── SMCSuperIO.kext
│       │   ├── VirtualSMC.kext
│       │   ├── WhateverGreen.kext
│       │   └── XHCI-unsupported.kext
│       ├── OpenCore.efi
│       ├── Resources
│       │   ├── Audio
│       │   ├── Font
│       │   ├── Image
│       │   └── Label
│       ├── Tools
│       │   └── OpenShell.efi
│       └── config.plist
├── Guide.md
└── README.md
```

### 定制 config.plist

Main Reference [url](https://dortania.github.io/OpenCore-Install-Guide/config.plist)

将OpenCore中 `Docs/Sample.plist` 拷贝到 `EFI/OC/config.plist`。同时准备编辑plist文件的工具。这里推荐使用 [properTree](https://github.com/corpnewt/ProperTree)。另备 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)备用。

> 虽然有一些fancy的国产工具可以更友好的编辑plist文件，但是会改变文件的排布，不方便后续升级opencore时比较版本，以及只能编辑opencore的config文件，例如在usb port mapping时需要修改的plist文件便不能用这类编辑器编辑。

定制 config.plist 的主要内容是：
* 根据自己使用的kext、efi，在plist文件中添加对应的条目；
* 根据自己的平台情况调整quirk（也就opencore中的各种选项开关）
* 修改SMBIOS，也就是设备信息（序列号，设备型号等）
* 定制一些内容，例如PCIE设备？

根据平台不通，需要调整的方式大同小异。笔者的平台是Comet Lake，大部分是直接参照dotania的博客进行的 [url](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html)。对于其他平台，请参照博客中其他部分。

笔者的定制：大部分quirk都是按照博客中进行调整。以下仅列出额外的部分：
* ACPI：
  * 增加了SSDT entry
  * 增加了一些Patch（q11 to xq11，q12 to xq12，似乎是在laptop上调整亮度用的，日后看看删掉...）
* Booter
  * 移除了macOS to hacOS的重命名
* DeviceProperties
  * 增加了关于 UHD 630 的部分(i5-10400，有核显。等到后期使用6600xt时需要调整此项)
  * 增加了声卡支持（layer 1）
* Kernel
  * 增加了自己的Kext entry
  * 移除了一个未enable的patch
* Misc
  * 增加了debug信息（appledebug, applepanic, disablewatchdog, target to 67)
* Tools 
  * 仅保留openshell.efi
* NVRAM
  * 修改prev-lang（语言）
  * 移除了delete中的 `ForceDisplayRotationInEFI`
  * 可以考虑为opencore添加hidpi支持
  * 保留了boot-args中的verbose输出
* PlatformInfo
  * 自己生成的SMBIOS，目前是iMac20,1
* UEFI
  * 删除大部分，只保留了openruntime.efi
```

