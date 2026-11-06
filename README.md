# Dell OptiPlex 5060 SFF - OpenCore Hackintosh EFI

This repository contains the OpenCore EFI configuration and bootable USB structure for running macOS on a **Dell OptiPlex 5060 Small Form Factor (SFF)** desktop.

> **Disclaimer:** This configuration is provided for educational and personal reference purposes only. Every system (even identical models with different BIOS revisions, drives, or peripherals) can behave differently. Use these files at your own risk—the author is not responsible for any data loss, hardware damage, or boot issues. Always back up your important data and verify your [`config.plist`](EFI/OC/config.plist) against [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) before booting.

---

## System Specifications

| Component | Specification |
| :--- | :--- |
| **Model** | Dell OptiPlex 5060 SFF (Small Form Factor) |
| **Motherboard / Chipset** | Dell OEM Intel Q370 Chipset |
| **CPU** | Intel Core i5-8400 (6 Cores / 6 Threads, 2.8 GHz Base / 4.0 GHz Turbo, Coffee Lake) |
| **Memory** | 16 GB DDR4 |
| **Graphics (iGPU)** | Intel UHD Graphics 630 (`AAPL,ig-platform-id`: `07009B3E`, dual DisplayPort outputs) |
| **Storage** | 256 GB SATA SSD |
| **Audio** | Realtek ALC3234 (ALC255) via `AppleALC.kext` (`alcid=3`) |
| **Ethernet** | Intel I219-LM Gigabit Ethernet via `IntelMausi.kext` |
| **USB** | Custom Dell OptiPlex 5060 SFF port map via `USBToolBox.kext` + `UTBMap.kext` (Front Type-C, USB 3.1, USB 2.0, Rear USB 3.1/2.0) |
| **SMBIOS** | `iMac19,1` |
| **Bootloader** | OpenCore (Debug Configuration) |

---

## Repository Structure

```text
.
├── .gitignore
└── EFI/
    ├── BOOT/
    │   └── BOOTx64.efi
    └── OC/
        ├── ACPI/
        │   ├── SSDT-AWAC.aml
        │   ├── SSDT-EC-USBX-DESKTOP.aml
        │   ├── SSDT-PLUG-DRTNIA.aml
        │   └── SSDT-PMC.aml
        ├── Drivers/
        │   ├── HfsPlus.efi
        │   └── OpenRuntime.efi
        ├── Kexts/
        │   ├── AppleALC.kext
        │   ├── IntelMausi.kext
        │   ├── Lilu.kext
        │   ├── SMCProcessor.kext
        │   ├── SMCSuperIO.kext
        │   ├── USBToolBox.kext
        │   ├── UTBMap.kext
        │   ├── VirtualSMC.kext
        │   └── WhateverGreen.kext
        ├── Resources/
        │   ├── Audio/
        │   ├── Font/
        │   ├── Image/
        │   └── Label/
        ├── Tools/
        │   ├── OpenShell.efi
        │   ├── libaistat.dylib
        │   ├── rtcread
        │   ├── smc
        │   └── smcread
        ├── OpenCore.efi
        └── config.plist
```

> **Note:** The local `com.apple.recovery.boot/` folder (containing `BaseSystem.dmg` and `BaseSystem.chunklist` for macOS Recovery installation) is ignored via [`.gitignore`](.gitignore) to keep binary recovery images out of version control.

---

## Detailed Configuration Breakdown

### 1. ACPI Tables (`EFI/OC/ACPI`)

| Table | Purpose |
| :--- | :--- |
| [`SSDT-AWAC.aml`](EFI/OC/ACPI/SSDT-AWAC.aml) | Disables the modern AWAC system clock on the Intel Q370 motherboard and enables the legacy RTC clock required by macOS. |
| [`SSDT-EC-USBX-DESKTOP.aml`](EFI/OC/ACPI/SSDT-EC-USBX-DESKTOP.aml) | Creates a fake Embedded Controller (`EC`) device and injects desktop USB power properties (`USBX`). |
| [`SSDT-PLUG-DRTNIA.aml`](EFI/OC/ACPI/SSDT-PLUG-DRTNIA.aml) | Enables native XCPM CPU power management (`plugin-type = 1`) for the Intel Core i5-8400. |
| [`SSDT-PMC.aml`](EFI/OC/ACPI/SSDT-PMC.aml) | Restores the `PMCR` device in ACPI to enable native NVRAM support on the Intel Q370 300-series chipset. |

### 2. UEFI Drivers (`EFI/OC/Drivers`)

| Driver | Purpose |
| :--- | :--- |
| [`HfsPlus.efi`](EFI/OC/Drivers/HfsPlus.efi) | Enables reading HFS+ volumes (used by macOS recovery images and legacy macOS installers). |
| [`OpenRuntime.efi`](EFI/OC/Drivers/OpenRuntime.efi) | Core runtime driver for OpenCore providing memory fixes, NVRAM variable routing, and booter quirks. |

### 3. Kernel Extensions (`EFI/OC/Kexts`)

| Kext | Version | Purpose |
| :--- | :---: | :--- |
| [`Lilu.kext`](EFI/OC/Kexts/Lilu.kext) | `1.7.2` | Core patching engine required by `WhateverGreen`, `AppleALC`, and `VirtualSMC` plugins. |
| [`VirtualSMC.kext`](EFI/OC/Kexts/VirtualSMC.kext) | `1.3.8` | Advanced Apple SMC emulator required to boot macOS. |
| [`SMCProcessor.kext`](EFI/OC/Kexts/SMCProcessor.kext) | `1.3.8` | VirtualSMC plugin for monitoring Intel Core i5-8400 CPU core temperatures. |
| [`SMCSuperIO.kext`](EFI/OC/Kexts/SMCSuperIO.kext) | `1.3.8` | VirtualSMC plugin for monitoring fan speeds via SuperIO. |
| [`WhateverGreen.kext`](EFI/OC/Kexts/WhateverGreen.kext) | `1.7.1` | Graphics patching for Intel UHD Graphics 630 framebuffer and DisplayPort outputs. |
| [`AppleALC.kext`](EFI/OC/Kexts/AppleALC.kext) | `1.9.8` | Native macOS HD audio support for the Dell OptiPlex 5060's Realtek ALC3234 (ALC255) codec. |
| [`IntelMausi.kext`](EFI/OC/Kexts/IntelMausi.kext) | `1.0.8` | Ethernet driver for the onboard Intel I219-LM Gigabit NIC. |
| [`USBToolBox.kext`](EFI/OC/Kexts/USBToolBox.kext) | `1.2.0` | USB port mapping engine. |
| [`UTBMap.kext`](EFI/OC/Kexts/UTBMap.kext) | `1.1` | Custom USB port map tailored for the Dell OptiPlex 5060 SFF `XHC` controller (`HS01`–`HS10` and `SS01`–`SS06`, including the front USB 3.1 Gen 2 Type-C port on `HS02`/`SS02`, front USB 2.0/3.0 ports, and rear USB 3.1/2.0 ports). |

### 4. DeviceProperties & NVRAM

- **Integrated Graphics (`PciRoot(0x0)/Pci(0x2,0x0)`) — Intel UHD Graphics 630**:
  - `AAPL,ig-platform-id`: `BwCbPg==` (`07009B3E`) — Desktop Coffee Lake framebuffer with hardware acceleration.
  - `framebuffer-patch-enable`: `AQAAAA==` (`01000000`) — Enables WhateverGreen framebuffer patching.
  - `framebuffer-stolenmem`: `AAAwAQ==` (`00003001`) — Patches stolen memory to 19 MB so macOS boots cleanly with Dell's default 32 MB DVMT pre-allocated memory (no BIOS DVMT mod required).
- **Audio (`PciRoot(0x0)/Pci(0x1b,0x0)`) — Realtek ALC3234 (ALC255)**:
  - `layout-id`: `AQAAAA==` (`1`), overridden by `alcid=3` in `boot-args` for internal speaker and front headset jack support.
- **Boot Arguments (`NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`)**:
  - `-v`: Enables verbose boot mode.
  - `keepsyms=1`: Preserves kernel symbols on panic for debugging.
  - `debug=0x100`: Prevents automatic reboot on a kernel panic.
  - `alcid=3`: Configures `AppleALC` with layout ID `3` for Realtek ALC3234 / ALC255.

### 5. Dell OptiPlex Specific Quirks

- **`CustomSMBIOSGuid = True`** (`Kernel -> Quirks`) & **`UpdateSMBIOSMode = Create`** (`PlatformInfo`): Essential for Dell systems so that OpenCore's `iMac19,1` SMBIOS spoofing is isolated to macOS and does not corrupt or overwrite Dell BIOS/Windows SMBIOS strings when dual-booting.
- **`AppleXcpmCfgLock = True`** (`Kernel -> Quirks`): Allows macOS XCPM power management to work out-of-the-box without requiring a hidden BIOS variable mod (`CFG Lock` is enabled by default in Dell OptiPlex 5060 firmware).
- **`DisableIoMapper = True`** (`Kernel -> Quirks`): Bypasses VT-d issues if `VT for Direct I/O` is left enabled in BIOS.

---

## Setup & Installation Guide

### 1. Generate Unique SMBIOS Identifiers (Required)
For security and iServices compatibility, unique identifiers in [`EFI/OC/config.plist`](EFI/OC/config.plist) have been replaced with `FILL_IN_YOURSELF`. Before booting:

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and [ProperTree](https://github.com/corpnewt/ProperTree).
2. Run `GenSMBIOS` and generate values for **`iMac19,1`**.
3. Open [`EFI/OC/config.plist`](EFI/OC/config.plist) in ProperTree and populate the following fields under `PlatformInfo -> Generic`:
   - `MLB` (Board Serial)
   - `ROM` (Your Intel I219-LM network card's 6-byte MAC address as `<data>`)
   - `SystemSerialNumber` (Serial)
   - `SystemUUID` (SmUUID)

### 2. Dell OptiPlex 5060 SFF BIOS Settings

Press **F2** at the Dell splash screen to enter BIOS Setup and configure the following settings:

- **General -> Boot Sequence**:
  - Boot List Option: **UEFI**
- **General -> Advanced Boot Options**:
  - Enable Legacy Option ROMs: **Unchecked (Disabled)**
- **System Configuration -> SATA Operation**:
  - Select **AHCI** *(Crucial: Dell defaults to `RAID On`, which prevents macOS from detecting the 256 GB SATA SSD)*
- **System Configuration -> Serial Port**:
  - Select **Disabled**
- **Security -> PTT Security**:
  - PTT On: **Unchecked (Disabled)** (optional, recommended if not required by Windows 11 dual-boot)
- **Secure Boot -> Secure Boot Enable**:
  - Select **Disabled**
- **Intel Software Guard Extensions -> Intel SGX Enable**:
  - Select **Disabled**
- **Virtualization Support -> Virtualization**:
  - Enable Intel Virtualization Technology: **Checked (Enabled)**
- **Virtualization Support -> VT for Direct I/O**:
  - Enable VT for Direct I/O: **Unchecked (Disabled)**
- **POST Behavior -> Fastboot**:
  - Select **Thorough**

### 3. Preparing the USB Installer

1. Format a USB drive (16 GB+ recommended) as **FAT32** with a **GUID Partition Map (GPT)**.
2. Copy the [`EFI`](EFI) folder to the root of the USB drive (or its EFI System Partition).
3. Obtain the macOS Recovery image using [`macrecovery.py`](https://github.com/acidanthera/OpenCorePkg/tree/master/Utilities/macrecovery) (bundled with OpenCorePkg) and place the `com.apple.recovery.boot/` directory containing `BaseSystem.dmg` and `BaseSystem.chunklist` at the root of the USB drive alongside [`EFI`](EFI).
4. Insert the USB drive into the OptiPlex 5060 SFF, press **F12** at boot to open the Dell One-Time Boot Menu, and select your UEFI USB drive.
5. In macOS Recovery Disk Utility, select **View -> Show All Devices**, erase the **256 GB SATA SSD** as **APFS** with **GUID Partition Map**, and proceed with the macOS installation.

---

## Post-Install Recommendations

- **Move EFI to the 256 GB SATA SSD**: Once macOS is installed, mount the EFI partition of your internal 256 GB SATA SSD using [MountEFI](https://github.com/corpnewt/MountEFI) and copy the `EFI` folder from the USB drive to the SSD's EFI partition so the system boots without the USB drive.
- **Disable Debug Logging (Optional)**: Once everything is verified working, you can remove `-v keepsyms=1 debug=0x100` from `boot-args` (leaving `alcid=3`), set `Misc -> Debug -> AppleDebug` and `ApplePanic` to `false`, and set `Misc -> Debug -> Target` to `0`.
- **USB Port Limit (`XhciPortLimit`)**: On macOS 11.3+, `Kernel -> Quirks -> XhciPortLimit` should be set to `false` once you confirm your active ports in [`UTBMap.kext`](EFI/OC/Kexts/UTBMap.kext/Contents/Info.plist) are trimmed to 15 or fewer ports.
- **CFG Lock (Optional)**: If you want native power management without `AppleXcpmCfgLock`, you can unlock MSR `0xE2` in Dell's UEFI NVRAM via `modGRUBShell.efi` (`setup_var 0x5A4 0x00` on most OptiPlex 5060 BIOS versions — verify offset with `ifrextract` before modifying).

---

## Credits & Resources

- [Acidanthera](https://github.com/acidanthera) for OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC, and IntelMausi.
- [Dortania's OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/) for Coffee Lake desktop configuration reference.
- [USBToolBox](https://github.com/USBToolBox/kext) for cross-platform USB port mapping.
