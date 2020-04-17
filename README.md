## NP530U3B | OpenCore 0.57 | macOS Catalina 10.15.4

### Summary

Opencore EFI folder, containing all necessary files for running macOS Catalina 10.15.4 on Samsung NP530U3B.

Everything works except Intel Centrino Advanced-N 6230 WiFi/Bluetooth adapter (no drivers exist for macOS), but it can be easily replaced by any [compatible half-sized mini PCIe adapter](https://dortania.github.io/Wireless-Buyers-Guide/types-of-wireless-card/mpcie.html) (laptop must be disassembled for this procedure).

### BIOS pre-requirements

Set AHCI mode to Manual/Enabled and enable UEFI Boot Support

To avoid kernel panic on Sandy Bridge you should disable CFG Lock. Unfortunately, there is no option to control it from BIOS GUI, so you either correct UEFI image and flash it, or stuck with AppleXcpmCfgLock/AppleCpuPmCfgLock quirks in OpenCore kernel config. These quirks didn't work for me, so I had to use DummyPowerManagement and completely disable PM for the installation.

#### CFG Lock disable

For correct power management on Sandy Bridge macOS should be able to control CPU multiplier by writing to MSR register. As stated above, for this laptop there is no way to achieve it from BIOS GUI. So we have to modify UEFI image. I had Windows 10 PE thumb drive near at hand, so the job was done in Windows 10.

- Download image

http://sbuservice.samsungmobile.com/upload/BIOSUpdateItem/ITEM_20130405_1079_WIN_13XK.exe

- Launch installer (do not install) and copy extracted directory

/Users/*Name*/AppData/Local/Temp/__Samsung_Update

- Unlock MSR_PMG_CST_CONFIG_CONTROL (0xE2)

Open 13XK.rom from copied dir in [UEFITool](https://github.com/LongSoft/UEFITool) and change **75080FBAE80F** hex bytes to **EB080FBAE80F** (JE to JMP) in PE32 image of PowerManagement2.efi (the same could be done by [UEFIPatch](https://github.com/LongSoft/UEFITool/releases) too, but I haven't tried)

- Write modified image

WinFlash.exe 13XK_mod.rom /v /cs /sd /sv /svs

If you're too lazy for this, I've uploaded [original and modified ROMs](BIOS)

### Intel HD Graphics 3000 drivers

Sandy Bridge graphics kexts were removed since Mojave, so you have to install them from High Sierra if you want QE/CI support (no metal support btw). Check [chris1111/Legacy-Video-patch](https://github.com/chris1111/Legacy-Video-patch)

### Workaround for backlight control

For some reason vanilla AppleBacklight.kext with WhateverGreen's backlight patches does not work correctly with this display ("fog" effect with low brightness), so we stuck with deprecated but still working IntelBacklight.kext, although without backlight control by default.

Backlight control fix found by [dioxine](https://www.tonymacx86.com/threads/guide-laptop-backlight-control-using-applebacklightfixup-kext.218222/page-186#post-1962813):
remount / with write permissions and replace

```
/System/LIbrary/Extensions/AppleBacklight.kext
/System/LIbrary/Extensions/AppleBacklightExpert.kext
/System/LIbrary/PrivateFrameworks/DisplayServices.framework
```

with the ones from [macOS Sierra 10.12.3 Combo Update](https://support.apple.com/kb/DL1905) (I've extracted and uploaded them to [backlight fix](backlight%20fix) dir) and fix permissions

Still looking for a better solution.

### Fixing iServices

If you need iMessage, FaceTime etc, follow [this manual](https://dortania.github.io/OpenCore-Desktop-Guide/post-install/iservices.html)

### Content, links and copyright

BOOTx64.efi, OpenCore.efi, OpenRuntime.efi — [acidanthera/OpenCorePkg](https://github.com/acidanthera/OpenCorePkg), [LICENSE](LICENSES/OpenCorePkg.txt)

ApfsDriverLoader.efi, VBoxHfs.efi — [acidanthera/AppleSupportPkg](https://github.com/acidanthera/AppleSupportPkg), [LICENSE](LICENSES/AppleSupportPkg-LICENSE.txt)

VirtualSMC.kext, SMCProcessor.kext, SMCBatteryManager.kext, SMCSuperIO.kext — [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC), [LICENSE](LICENSES/VirtualSMC.txt)

Lilu.kext — [acidanthera/Lilu](https://github.com/acidanthera/Lilu), [LICENSE](LICENSES/Lilu.txt)

WhateverGreen.kext — [acidanthera/WhateverGreen](https://github.com/acidanthera/WhateverGreen), [LICENSE](LICENSES/WhateverGreen.txt)

AppleALC.kext — [acidanthera/AppleALC](https://github.com/acidanthera/AppleALC), [LICENSE](LICENSES/AppleALC.txt)

RealtekRTL8111.kext — [Mieze/RTL8111_driver_for_OS_X](https://github.com/Mieze/RTL8111_driver_for_OS_X)

ApplePS2SmartTouchPad.kext — [EMlyDinEsH](https://osxlatitude.com/forums/topic/1948-elan-focaltech-and-synaptics-smart-touchpad-driver-mac-os-x/)

GenericUSBXHCI.kext — [RehabMan/OS-X-Generic-USB3](https://github.com/RehabMan/OS-X-Generic-USB3)

IntelBacklight.kext — [RehabMan/OS-X-Intel-Backlight](https://github.com/RehabMan/OS-X-Intel-Backlight)

USBPorts.kext — generated with [Hackintool](https://github.com/headkaze/Hackintool)

SSDT.aml — generated with [ssdtPRGen.sh](https://github.com/Piker-Alpha/ssdtPRGen.sh)

DSDT.aml — original version by [m.gbt](https://www.osx86.net/topic/20310-guide-uefi-mavericks-109x-el-capitan-10112-on-samsung-np530u3b/), with several patches from [RehabMan](https://github.com/RehabMan/Laptop-DSDT-Patch) and [myself](https://github.com/2b)