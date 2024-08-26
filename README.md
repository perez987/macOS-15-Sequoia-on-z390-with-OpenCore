# macOS 15 Sequoia beta on Z390 using OpenCore

<p align="center">
<img width="128" src="img/Sequoia icon.png">
</p>

### Preface

Apple has released the first Public Beta and the third Developer Beta of macOS 15 Sequoia. Its installation is less problematic than previous versions of macOS that required deeper changes to OpenCore and kexts. This time, few changes have been necessary to make them compatible. The system works surprisingly well for being in such an early stage of development. Of course, there are things to polish but, in general, it is very usable even for daily use although it is not recommended in production environments.

---

### Hardware

* Motherboard Gigabyte Z390 Aorus Elite
* CPU Intel i9-9900K
* GPUs: iGPU Intel UHD 630 / AMD Radeon RX 6600 XT
* Audio Realtek ALC1220
* Ethernet Intel I219V7
* Wi-Fi + BT Fenvi FV-T919 (BCM94360CD).

### BIOS settings (F11 version)

* CSM: Disabled (mandatory)
* VT-d: Disabled
* Platform Power Management: Disabled
* XHCI Hand-Off: Enabled
* Network Stack: Disabled
* Wake on LAN: Disabled
* Initial Display Output: PCIe 1 Slot
* Integrated Graphics: Enabled
* DVMT Pre-allocated: 256M o higher
* Above 4G Decoding: Enabled
* CFG Lock: Disabled (mandatory)
* Fast Boot: Disabled
* OS Type: Windows 8/10
* Secure Boot: Disabled.

### What works well?

* dGPU AMD as main card
* iGPU *headless mode*
* Shutdown and restart
* Ethernet
* Sound (also HDMI)
* USB ports (USB port map for this board)
* Bluetooth Fenvi T919.

### What's not working?

* Fenvi T919 Wi-Fi: It needs a fix, still in the development phase, created by the OCLP team
* Sleep: It doesn't always work as it should, sometimes it goes to sleep properly and other times the screen goes black but the PC stays on.

---

### Mac models compatible with Sequoia

macOS 15 is compatible with these Mac models with Intel processors:

- iMac 2019+ (iMac19,1 / iMac20,1 / iMac20,2)
- iMac Pro 2017+ (iMacPro1,1)
- Mac mini 2018+ (MacMini8,1)
- MacBook Pro 2018+ (MacBookPro15,1 / MacBookPro15,2 / MacBookPro16,1 / MacBookPro16,2 / MacBookPro16,3 / MacBookPro16,4)
- MacBook Air 2020+ (MacBookAir9,1)
- Mac Pro 2019+ (MacPro7,1).

The list of supported models compared to macOS 14 Sonoma has one change: MacBook Air which included models from 2018 and now only supports those from 2020.

All supported models except iMac19,1 (2019 iMac) have a T2 security chip, this is important when we talk about updates notification in System Settings (OTA: Over The Air updates).

---

### Get macOS 15 Sequoia beta

Since the release of macOS 15 Public Beta it is easy to participate in the testing program. If you have macOS Ventura 13.4 or later:

- Go to the [Free Apple Beta Software Program](https://beta.apple.com/) site
- Sign in with your Apple account -> Get Started -> Enroll your Mac
- Go to Software Update -> Beta Updates
- Open the drop-down menu and choose macOS Sequoia Public Beta
- Download the package
- macOS 15 Beta Installer app is saved in the Applications folder.

Note: Apple account on your Mac must match the one in the beta program.

<img src="img/Betaupdates.png" width="660px">

This is the easiest way to get macOS 15.

There is another way which is to download the full installation package, it is not necessary to access the Apple beta program. There are free applications that are well known and tested enough to download macOS 15 Beta Installer, these are just 2 of them:

- [GibMacOS](https://github.com/corpnewt/gibMacOS) (Terminal application)
- [Mist](https://github.com/ninxsoft/Mist) (graphical application that can also create the installation USB device).

From here you can update the current system or create an installation USB device to install Sequoia from scratch. This task is sufficiently described on the Internet, here is a simple reminder:

- Get a USB device with at least 16 GB (preferably 32)
- Prepare the device with Disk Utility:
- Name: USB
- Partition scheme: GUID
- Format: MacOS Extended Journaled
- Open Terminal and run `sudo /Applications/Install\ macOS\ Sequoia\ Beta.app/Contents/Resources/createinstallmedia --volume /Volumes/USB /Applications/Install\ macOS\ Sequoia\ Beta.app --nointeraction`
- When finished, the USB device is named Install macOS Sequoia Beta
- Prepare OpenCore on the EFI partition of the USB device.
- You can now boot from this device and install macOS from scratch.

---

### OpenCore 1.0.0 (same as Sonoma) 

Most of the OpenCore settings that were valid for Sonoma are also valid for Ventura. Main differences are in the versions of some kexts (not all of them are used on my hardware):

- Lilu 1.6.8 beta (if you use Lilu 1.6.7 you have to add `-lilubetaall` in boot args)
- AirportBrcmFixup 2.1.9 beta
- CPUFriend 1.2.8 beta
- CpuTscSync 1.1.1 beta
- ECEnabler 1.0.5
- HibernationFixup 1.5.1 beta
- IntelBluetoothFirmware 2.5.0 beta
- RestrictEvents 1.1.3 or 1.1.4 beta (with both you need `-revbeta` in boot args)
- VoodooInput 1.1.5 beta
- WhateverGreen 1.3.7 beta.

Other extensions may be the most recent official versions. 

---

### config.plist

I get best results with iMac19.1 SMBIOS and the iGPU enabled in BIOS. These are the main details when configuring config.plist.

- ACPI: SSDT-EC-USBX.aml, SSDT-PLUG.aml and SSDT-PMC.aml. SSDT-AWAC.aml is not required on my system but, if in doubt, add it because it does not cause any harm if it is present without being needed
- ACPI >> Quirks: all = False

- Booter >> Quirks: AvoidRuntimeDefrag, DevirtualiseMmio, ProtectUefiServices, ProvideCustomSlide, RebuildAppleMemoryMap, SetupVirtualMap and SyncRuntimePermissions = True
- Booter >> ResizeAppleGpuBars=-1

- DeviceProperties >> Add
	- PciRoot(0x0)/Pci(0x2,0x0)
		- AAPL,ig-platform-id | Data | 0300913E
		- device-id | Data | 9B3E0000
		- enable-metal | Data | 01000000
		- rps-control | Data | 01000000
	- PciRoot(0x0)/Pci(0x1.0x0)/Pci(0x0.0x0)/Pci(0x0.0x0)/Pci(0x0.0x0)
		- unfairgva | Number | 6
	- PciRoot(0x0)/Pci(0x1F,0x3)
		- layout-id | Data | 07000000
	- PciRoot(0x0)/Pci(0x14,0x0)
		- acpi-wake-type | Data | 01
		- acpi-wake-gpe | Data | 6D

- Kernel > Add: Sequoia compatible kexts, Lilu.kext in the first place, UTBMap.kext specific for this motherboard
- Kernel >> Quirks: CustomSMBIOSGuid, DisableIoMapper, DisableIoMapperMapping, DisableLinkeditJettison, PanicNoKextDump and PowerTimeoutKernelPanic = True
- Kernel >> Quirks: SetApfsTrimTimeout = 0

- Misc >> Boot: HibernateMode=None, PickerAttributes=144, ShowPicker=True
- Misc >> Debug: AppleDebug, ApplePanic and DisableWatchDog = True, Target=3
- Misc >> Security: AllowSetDefault=True, BlacklistAppleUpdate=True, ExposeSensitiveData=6, SecureBootModel=x86legacy

- NVRAM
	- WriteFlash=True
	- Add >> 7C436110-AB2A-4BBB-A880-FE41995C9F82:
		- boot-args >> agdpmod=pikera
		- csr-active-config >> 00000000
		- run-efi-updater >> No
	- Delete >> 7C436110-AB2A-4BBB-A880-FE41995C9F82:
		- boot-args and csr-active-config

- PlatformInfo
	- Generic >> iMac19.1
	- UpdateDataHub, UpdateNVRAM and UpdateSMBIOS = True
	- UpdateSMBIOSMode >> Custom

- UEFI >> Quirks: EnableVectorAcceleration and RequestBootVarRouting = True
- UEFI >> Quirks >> ResizeGpuBars=-1.

### Updates Notification 

Only SMBIOS iMac19.1 model that lacks T2 security chip receives notifications of new updates in System Settings. Rest of the models that have T2 chip are only notified if you add:

- RestrictEvents.kext
- `-revbeta revpatch=sbvmm `in boot args: it makes macOS believe that it is in a virtual machine and it does not matter which SMBIOS model has a T2 chip. 

These settings can be disabled for daily use of the system but you have to re-enable them when you want to be notified of new updates. Another option is to download the full installer package each time, bypassing this limitation but they are large packages of around 15 GB so they are not practical for those who have to update a lot of computers at a time. 

<img src="img/RestrictEvents.png " width="660px">

---

### EFI folder settings

You can try the same settings that work in macOS Sonoma. Except for kext versions and what was said about update notification.

---

### Intel Wi-Fi and Bluetooth

- AirportItlwm does not work on Sequoia. itlwm does. Latest version is 2.3.0.
- Heliport (used in conjunction with itlwm) has also been updated to version 1.5.0 alpha which seems to work fine on Sequoia.
- Regarding Bluetooth, latest version 2.4.0 works fine on Sequoia.

Extended instructions:: [Intel AX210 wifi6 on macOS Sonoma](https://github.com/perez987/Intel-AX210-wifi6-on-macOS-Sonoma)

---

### Fenvi and Broadcom Wi-Fi

OCLP developers have released a beta version that works on Sequoia, it is different from the one we were using on Sonoma.

- Grab AMFIPass version 1.4.1 from [here](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Acidanthera)
- Grab IOSkyWalkFamily version 1.1.0 from [here](https://github.com/dortania/OpenCore-Legacy-Patcher/tree/main/payloads/Kexts/Wifi)
- Grab OCLP beta version 1.6.0 from [here](https://github.com/dortania/OpenCore-Legacy-Patcher/actions/runs/9953580191) (OpenCore-Patcher.pkg)

Other settings are as before: [Broadcom wifi back on macOS Sonoma with OCLP](https://github.com/perez987/Broadcom-wifi-back-on-macOS-Sonoma-with-OCLP). 
