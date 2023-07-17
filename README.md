# macOS-Dragonair

## Table of Contents
- [Current Status](#current-status)
- [Versions Tested](#versions-tested)
- [Requirements](#requirements)
- [Issues](#current-issues)
- [**1. Installation**](#1-installation)
   - [Required Steps](#these-steps-are-required-for-proper-functioning)
   - [Fixing coreboot 4.2.0](#fixing-coreboot-420)
   - [Kext Folder](#kexts)
   - [ACPI Folder](#acpi-folder)
- [Misc. Information](#misc-information)
## Current Status

| **Feature**        | **Status**           | **Notes**                                                                                     |
|--------------------|----------------------|-----------------------------------------------------------------------------------------------|
| WiFi               | Working              | With `Airportitlwm` on Ventura, `itlwm` + `heliport` for Sonoma                                          |
| Bluetooth          | Working              | With `IntelBluetoothFirmware` and `BlueToolFixup.kext`.                                       |
| Suspend / Sleep    | Working partially    | Only on battery power, working with `EmeraldSDHC.kext`.                                       |
| Trackpad           | Working              | With `VoodooI2C.kext` and `VoodooI2CELAN.kext`.                                               | 
| Graphics Accel.    | Working              | With `-igfxnotelemetryload` in the `boot-args`.                                               |
| Internal Speakers  | Not working          | Unsupported codec.                                                                            |
| Keyboard backlight | Working              | With `SSDT-KBBl.aml` _**and**_ the custom `VoodooPS2.kext`.                                   |           
| Keyboard & Remaps  | Working              | With the custom `VoodooPS2.kext`.                                                             |
| eMMC Storage       | Working              | With `EmeraldSDHC.kext`and patched HPET                                                       |    
| SD Card Reader     | Not working          | WIP with `EmeraldSDHC.kext`.                                                                  |
| Headphone Jack     | Not working          | Unsupported codec                                                                             |
| HDMI Audio         | Untested             | Don't have any monitors with HDMI audio                                                       |
| HDMI Video         | Working              | Working OOTB                                                                                  |
| USB Ports          | Working              | Working with USB mapping                                                                      |
| Webcam             | Working              | Working OOTB                                                                                  |
| Internal Mic.      | Not working          | Same reason why internal speakers don't work; unsupported codec.                              |
| Logout / Lock      | Working              | Working OOTB.                                                                                 |
| Shutdown / Restart | Working              | Working with `ProtectMemoryReigons` set to true in `config.plist`.                            |    
| Continuity         | Not Working          | Limitation with Intel WiFI cards / `itlwm`.                                                   |    
                                                                          
--------------------------------------------------------------------------------------------------------------------------------------------------------
### Versions Tested

- 13 (Ventura)
- 14 (Sonoma) 

--------------------------------------------------------------------------------------------------------------------------------------------------------

### Requirements

Before you start, you'll need to have the following items to complete the process:

- **An understanding that this process has the potential to damage and/or brick your device, potentially causing it to become inoperable.**
- An external storage device (can range from a SD card to a USB Disk / Drive) for creating the installer USB.  
- The latest OpenCore version (**at least 0.8.8**) for eMMC boot drive support.   
- An internet connection.


### Current Issues
>**Note**: coreboot 4.20 (5/15/2023 release) is known to cause issues with booting macOS. A fix can be found [below](#fixing-coreboot-420).
- https://github.com/meghan06/ChromebookOSX/issues/10 Chromium based apps breaking after sleep. [help needed] 
  - A temp. workaround is to not let the device sleep, and if it does sleep, reboot the system.

## 1. Installation

Here are the steps to go from chromeOS to macOS via OpenCore on your Chromebook. 

--------------------------------------------------------------------------------------------------------------------------------------------------------

> **Warning** Pay _close_ attention to the Chromebook specific parts in the Dortania guide, specifically in `Booter -> Quirks` and the iGPU `boot-args`.

> **Warning** Pay _very_ close attention to the following steps, if you miss **even one**, your Chromebook will lose some functionally and might not even boot.

> **Note**: C434 and C433 users (SHYVANA) will need [VoodooI2CHID](https://github.com/VoodooI2C/VoodooI2C/releases/).kext for touchscreen to function. It is bundled with the VoodooI2C package.

> **Note**: Those who are installing to an external disk like a USB drive can skip steps 10 and 11.


--------------------------------------------------------------------------------------------------------------------------------------------------------

### **These steps are **required** for proper functioning.**

1. If you haven't already, flash your Chromebook with [MrChromebox's UEFI firmware](https://mrchromebox.tech) via his scripts. To complete this process, you must turn off write protection either by using a SuzyQable  or temporarily removing the battery, with latter being less cumbersome.
2. Setup your EFI folder using the [OpenCore Guide](https://dortania.github.io/OpenCore-Install-Guide/). Use [Laptop Coffee Lake Plus & Comet Lake](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/coffee-lake-plus.html#acpi) for your `config.plist`. 
3. Re-visit this guide when you're done setting up your EFI. There are a few things we need to tweak to ensure our Chromebook works with macOS. Skipping these steps will result in a **very** broken hack.
4. Add the compiled version of [SSDT-PLUG-ALT](https://github.com/meghan06/croscorebootpatch) to your ACPI folder.
5. In your `config.plist`, under `Booter -> Quirks` set `ProtectMemoryRegions` to `TRUE`. It should look something like this in your `config.plist` when done correctly:

   | Quirk                | Type | Value    |
   | -------------------- | ---- | -------- |
   | ProtectMemoryRegions | Boolean | True  |
   
   > **Warning** **This must be enabled.**

6. Under `DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x2,0x0)`, make the following modifications:
  
   | Key                  | Type | Value    |
   | -------------------- | ---- | -------- |
   | AAPL,ig-platform-id  | data | 00009B3E |
   | device-id            | data | 9B3E0000 |
   | framebuffer-patch-enable | data | 01000000 |
   | framebuffer-stolenmem | data | 00003001 |
   | framebuffer-fbmem | data | 00009000 |
     
   > **Warning** **These should be the only items `in PciRoot(0x0)/Pci(0x2,0x0)`.**
8. If you haven't already, add `-igfxblr` and `-igfxnotelemetryload` to your `boot-args`, under `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82,`. Both are for iGPU support, **you will regret it if you don't add these.**
9. **Set your SMBIOS as `MacBookAir9,1`**. Ignore what Dortania tells you to use, `MacBookAir9,1` works better with our Chromebook.
10. Switch the VoodooPS2 from acidanthera with this [custom build that's designed for Chromebooks](https://github.com/meghan06/ChromebookPS2) for keyboard backlight control + custom remaps. 
   - Keyboard backlight SSDT (`SSDT-KBBL.aml`) can be found [here](https://github.com/meghan06/ChromebookPS2/blob/master/Docs/SSDT-KBBL.aml). Drag it to your ACPI folder.
     > **Note**: This SSDT only works with the custom VoodooPS2 linked above.
11. Download [EmeraldSDHC](https://github.com/acidanthera/EmeraldSDHC/releases) for eMMC storage support. Put it in your Kexts folder. 
12. Download corpnewt's SSDTTime, then launch it and select `FixHPET`. Next, select `'C'` for default, and drag the SSDT it generated (`SSDT-HPET.aml`) into your `ACPI` folder. Finally, copy the patches from `oc_patches.plist` into your `config.plist` under `ACPI -> Patch`. 

    > **Warning** Steps 11 and 12 are **required** for macOS to recognize the internal eMMC disk. 
    > **Note** If eMMMC isn't recognized in Disk Utility, you probably made a mistake in steps 11 and 12.

13. Map your USB ports³ before installing ~~to prevent dead hard drives, thermonuclear war, or you getting fired.~~ See [Misc. Information](#misc-information) for a note to USBToolBox users.    
14. Using corpnewt's SSDTTime, dump your DSDT, generate `SSDT-USB-RESET.aml`, drag it to your ACPI folder, and reload your `config.plist`. **Required** for working USB ports.

    > **Note**:  USBToolBox users can skip this step, otherwise you must do this or your USB ports won't work.

15. Snapshot (cmd +r) or (ctrl + r) your `config.plist`. 

    > **Warning**: NEVER do clean snapshots (`ctrl/cmd+shift+r`) after adding your HPET patches, they will be **wiped**. Only do regular snapshots. (`ctrl/cmd+r`)

16. Install macOS and enjoy!

> **Note**: In depth information about OpenCore can be found [here.](https://dortania.github.io/docs/latest/Configuration.html)

> **Note**: if Installing Sonoma add `-lilubetall` to the boot args
--------------------------------------------------------------------------------------------------------------------------------------------------------

### Fixing coreboot 4.20
coreboot 4.20 (5/15/2023 release) has a known issue where macOS will hang on boot due to coreboot not defining CPU cores by default. To fix this, we'll use a SSDT to manually define them. Credits to [ExtremeXT](https://github.com/ExtremeXT) for the fix.

- If you haven't already, add the compiled version of [SSDT-PLUG-ALT](https://github.com/meghan06/croscorebootpatch) to your ACPI folder.


--------------------------------------------------------------------------------------------------------------------------------------------------------

### Kexts

```
AirportItlwm.kext (itlwm + heliport for Sonoma)
BlueToolFixup.kext
ECEnabler.kext
EmeraldSDHC.kext
HibernationFixup.kext
IntelBluetoothFirmware.kext
IntelBTPatcher.kext
Lilu.kext
SMCBatteryManager.kext
SMCProcessor.kext
UTBMap.kext
USBToolBox.kext
VirtualSMC.kext
VoodooI2C.kext
VoodooI2CELAN.kext
VoodooI2CHID.kext
VoodooPS2Controller.kext
WhateverGreen.kext
```

--------------------------------------------------------------------------------------------------------------------------------------------------------

### ACPI Folder

```
SSDT-EC-USBX-LAPTOP.aml
SSDT-HPET.aml
SSDT-KBBL.aml
SSDT-PNLF-CFL.aml
SSDT-PLUG-ALT.aml
```
>**Note**: These SSDTs were generated with [SSDTTime](https://github.com/corpnewt/SSDTTime)

>**Note**: `SSDT-HPET` and it's `config.plist` patch is only required for eMMC support.

--------------------------------------------------------------------------------------------------------------------------------------------------------

## Misc. Information

- When formatting the eMMC drive in Disk Utility, make sure to toggle "Show all Drives" and erase the entire drive.
- Format the drive as `APFS` and `GUID Partition Table / GPT`
- Map your USB ports prior to installing macOS³ for a painless install. You **will** reget it if you don't. You can use [USBToolBox](https://github.com/USBToolBox/tool) to do that. You *will* need a second kext that goes along with it for it to work. [Repo here.](https://github.com/USBToolBox/kext). USBToolBox will not work without this kext. 
- AppleTV and other DRM protected services may not work.
- Control keyboard backlight with left `ctrl` + left `alt` and `<` `>`. 
    - `<` to decrease, `>` to increase.
- To fix  battery life, use CPUFriend to tweak power settings. 
- To hide the OpenCore boot menu, set `ShowPicker` to `False` in `Misc` ->` Boot` -> `ShowPicker`
- `AppleXcpmCfgLock` and `DisableIOMapper` can be enabled or disabled. There is no difference.
- eMMC will not be recognized if `ScanPolicy` is set to `0`.
>**Note**: SSDT-USB-Reset / SSDT-RHUB is not needed if using USBToolBox.

credit to [meghan06](https://github.com/meghan06/) for his guide which I based this one on. 
