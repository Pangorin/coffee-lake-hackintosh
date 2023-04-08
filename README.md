# Intel Coffee Lake macOS installation guide
Hackintosh guide for Intel Coffee Lake systems
## Things to remember
+ DO AT YOUR OWN RISK, THIS GUIDE WILL NOT COVER ANY KIND OF HARDWARE ISSUES!
+ Knowing your hardware properly (trust me, you'll not able to install macOS if you do not know anything about your PC)
+ Time and patience, because Hackintosh is not that easy, even if you follow the guide properly.

## Getting started
Download these files:
- Bootloader: [OpenCore Bootloader](https://github.com/acidanthera/OpenCorePkg/releases)
+ Kexts:
  - [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases)
  - [Lilu](https://github.com/acidanthera/Lilu/releases)
  - [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases)
  - [AppleALC](https://github.com/acidanthera/AppleALC/releases)
  - [Realtek RTL8111 Ethernet](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)
    - Remember, this depends on what motherboard you're using, and your Ethernet card
  - [USBToolBox](https://github.com/USBToolBox/kext/releases/tag)
  - [XHCIUnsupported](https://github.com/RehabMan/OS-X-USB-Inject-All)
    - Common chipsets needing this: H310, H370, B360, Z390 (not needed on Mojave and newer), X79, X99, ASRock Intel board
 + SSDTs:
    - [SSDT-PLUG](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-PLUG-DRTNIA.aml)
    - [SSDT-EC-USBX-DESKTOP](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-EC-USBX-DESKTOP.aml)
    - [SSDT-AWAC](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-AWAC.aml)
    - [SSDT-PMC](https://github.com/dortania/Getting-Started-With-ACPI/blob/master/extra-files/compiled/SSDT-PMC.aml)

## config.plist configuration
Tools we need:
- [ProperTree](https://github.com/corpnewt/ProperTree)
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
- config.plist file: Included in the OpenCore Bootloader file, original name Sample.plist, just rename it and copy into EFI/OC folder

Starting point:
Snapshot your OC folder in order to make this config.plist file can read all your SSDTs and kexts

- ACPI: There's nothing to change here
- Booter: Head to Quirks section, change these setting:
  + DevirtualiseMmio: YES
  + EnableWriteUnprotector: NO
  + ProtectUefiServices: YES (needed on Z390 systems)
  + RebuildAppleMemoryMap: YES
  + ResizeAppleGpuBars: -1 (If your firmware supports increasing GPU Bar sizes (ie Resizable BAR Support), set this to 0)
  + SyncRuntimePermissions: YES
- DeviceProperties: In the Add section, add this device properties:
  + PciRoot(0x0)/Pci(0x2,0x0)
    - This section is set up via WhateverGreen's Framebuffer Patching Guide (opens new window)and is used for setting important iGPU properties. If you have a -F series CPU, you can ignore this section as you do not have an iGPU.
    - The config.plist doesn't already have a section for this so you will have to create it manually.
    - If you have a dedicated GPU, do this under the PciRoot(0x0)/Pci(0x2,0x0) you created:
      - Add AAPL,ig-platform-id, set it to Data, and change the value to this: 0300913E
    - If you want to use your iGPU to drive a display:
      - Add AAPL,ig-platform-id, set it to Data, and change the value to this: 07009B3E
      - Add framebuffer-patch-enable, set it to Data, and change the value to this: 01000000
      - Add framebuffer-stolenmem, set it to Data, and change the value to this: 	00003001
- Kernel: Head to Quirks section, change these setting:
  + AppleXcpmCfgLock: YES (Not needed if CFG-Lock is disabled in the BIOS)
  + DisableIoMapper: YES (Not needed if VT-D is disabled in the BIOS)
  + LapicKernelPanic: NO (HP machines will require this quirk)
  + PanicNoKextDumpL: YES
  + PowerTimeoutKernelPanic: YES
  + XhciPortLimit: YES (Disable if running macOS 11.3+)
Misc: 
  + In Boot section, set HideAuxiliary to NO
  + In Debug section:
    + AppleDebug: YES
    + ApplePanic: YES
    + DisableWatchDog: YES
    + Target: 67
  + In Security section:
    + AllowSetDefault: YES
    + BlacklistAppleUpdate: YES
    + ScanPolicy: 0
    + SecureBootModel: Default
    + Vault: Optional
NVRAM:
  + In 7C436110-AB2A-4BBB-A880-FE41995C9F82 section, add these boot-args:
    + debug=0x100, alcid=1, unfairgva=1 (Used for fixing hardware DRM support on supported AMD GPUs), wegnoegpu (use this if you have a unsupported dedicated GPU, and want to use only your iGPU)
    + prev-lang:kbd: change to 656e2d55533a30 (Russian moment, this will change your installer language to English)
PlatformInfo:
  + For setting up the SMBIOS, , we'll use CorpNewt's GenSMBIOS application. Coffee Lake systems have 2 main SMBIOS: iMac19,1 for Mojave and newer, iMac18,3 for High Sierra and older.
    - Run GenSMBIOS, pick option 1 for downloading MacSerial and Option 3 for selecting out SMBIOS. This will give us an output similar to the following
  
  | iMac19,1 SMBIOS Info |                                      |  
  | ---------------------| -------------------------------------|
  | Type                 | iMac19,1                             |
  | Serial               | C02XG0FDH7JY                         |
  | Board Serial         | C02839303QXH69FJA                    |
  | SmUUID               | DBB364D6-44B2-4A02-B922-AB4396F16DA8 |
  | Apple ROM            | 11223300 0000                        |
  
  (DO NOT COPY THIS! THIS SMBIOS IS JUST AN EXAMPLE)

The Type part gets copied to Generic -> SystemProductName.

The Serial part gets copied to Generic -> SystemSerialNumber.

The Board Serial part gets copied to Generic -> MLB.

The SmUUID part gets copied to Generic -> SystemUUID.

- UEFI: nothing to change here

## That's it, you have created an EFI for your system, now you just have to change some settings in BIOS
Things to disable:
  - Fast Boot
  - Secure Boot
  - Serial/COM Port
  - Parallel Port
  - Compatibility Support Module (CSM) (Must be off in most cases, GPU errors/stalls like gIO are common when this option is enabled)
  - Thunderbolt (For initial install, as Thunderbolt can cause issues if not setup correctly)
  - Intel SGX
  - Intel Platform Trust
  - CFG Lock
Things to enable:
- VT-x
- Above 4G Decoding
- Hyper-Threading
- Execute Disable Bit
- EHCI/XHCI Hand-off
- OS type: Windows 8.1/10 UEFI Mode (some motherboards may require "Other OS" instead)
- DVMT Pre-Allocated(iGPU Memory): 64MB or higher (only enable this if you want to use your iGPU to drive a display)
- SATA Mode: AHCI
