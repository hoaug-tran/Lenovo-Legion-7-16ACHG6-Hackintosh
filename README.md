<p align="center">
  <img width="150px" src="https://rallybr.com.br/logo-hacklegion.png" align="center" alt="Hackintosh Logo" />
</p>

<h1 align="center">Lenovo Legion 7 (16ACHg6) Hackintosh</h1>
<h3 align="center">Ryzen 9 5900HX | Vega 8 (NootedRed) | macOS Sequoia 15.7.3</h3>

<p align="center">
  <a href="#-english-version">
    <img src="https://img.shields.io/badge/English-Read_Now-blue?style=for-the-badge&logoColor=white" alt="English">
  </a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/BIOS-GKCN53WW-blue?style=flat-square&logo=lenovo&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenCore-1.0.6-8A2BE2?style=flat-square&logo=acidanthera&logoColor=white" />
  <img src="https://img.shields.io/badge/macOS-Sequoia-success?style=flat-square&logo=apple&logoColor=white" />
</p>

<hr />

### Disclaimer
> _Information provided here is for reference only. Modifying your BIOS or EFI involves risks. Always **backup your data** before proceeding. I am not responsible for any damage to your device._

### Hardware Specifications

| Component | Detail | Status |
| :--- | :--- | :---: |
| **Model** | Lenovo Legion 7-16ACHg6 | ✅ |
| **CPU** | AMD Ryzen™ 9 5900HX (8C/16T) | ✅ |
| **iGPU** | AMD Radeon™ Vega 8 (via **NootedRed**) | ✅ |
| **dGPU** | NVIDIA GeForce RTX 3080 16GB | ⛔️ |
| **RAM** | 64GB DDR4 3200MHz | ✅ |
| **NVMe 1** | SKHynix PC711 (HFS001TDE9X084N) (1TB) | ⛔️ |
| **NVMe 2** | SUNEAST Gen4 G55 (2TB) - MacOS and Window installed | ✅ |
| **Audio** | Realtek ALC287 (Speakers & Mic) | ✅ |
| **Ethernet** | Realtek RTL8111 | ✅ |
| **Wi-Fi/BT** | Intel® Wi-Fi 6 AX210 | ✅ |
| **Display** | 16" WQXGA (2560x1600) IPS 165Hz | ✅ |

### BIOS Settings

* **Switchable Graphics / Hybrid Mode:** `ENABLED` (Required for Vega 8 detection).
* **Secure Boot:** `DISABLED`.
* **Platform Security Processor (PSP):** `DISABLED`.
* **Device Guard:** `DISABLED`.
* **Fast Boot:** Can be `ENABLED` or `DISABLED` (Tested and found no impact).

### Post-Configuration (Mandatory)
Before using this EFI, you **MUST** generate your own SMBIOS information. Using the serials provided in any sample config will lead to iServices issues and potentially an Apple ID ban.

* **Tool:** [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)
* **Model:** `MacBookPro16,2`
* **Action:** Generate `Serial`, `Board Serial`, and `SmUUID` then paste them into your `config.plist` under `PlatformInfo > Generic`.

### Key Features & Important Notes

#### 1. Graphics & External Display
* **Internal Display:** Working perfectly with full acceleration.
* **HDMI & Type-C Output:** ⛔️ **NOT WORKING**. On this model, HDMI and Type-C video ports are hardwired to the RTX 3080. Since the dGPU is unsupported in macOS, external monitors cannot be used.

#### 2. The dGPU (RTX 3080) Issue
> ⚠️ **CRITICAL:** Do NOT use `SSDT-DGPU-OFF.aml` or `SSDT-NoHybGfx.aml`.

Using these SSDTs causes the fans to spin at max speed, and the laptop will shut down after 1 minute.
* **Solution:** I use the boot-arg `-wegnoegpu`. This disables the dGPU properly via software, keeping the system stable and cool.

#### 3. Sleep/Wake Behavior
Sleep works, but with specific limitations due to the USB Controller handling (`SSDT-SLEEP.aml`):
* **Wake Method:** You can **ONLY wake the device using the Power Button**.
* **Limitations:** Waking via USB devices (Mouse/Keyboard) or the Laptop Keyboard is **disabled** to prevent instant wake issues.
* **Requirements:** `SSDT-PTSWAKTTS` + `SSDT-EXT3-WakeScreen` + `SSDT-SLEEP` + Correct USB Map.

#### 4. CPU Power Management
I deliberately **do not use** `AMDRyzenCPUPowerManagement.kext` or `SMCAMDProcessor.kext`.
* **Reason:** Letting the laptop firmware handle power management results in better thermal performance and stability for the Ryzen 5900HX.

#### 5. NVMe SSD Compatibility (SSDT-dSSD)
The stock **SKHynix PC711 (HFS001TDE9X084N)** SSD is not compatible with macOS (causes Kernel Panics), so I used `SSDT-dSSD.aml` to completely hide/disable it from macOS.
* **Important:** If your SSD is compatible (e.g., WD, Samsung) or different from mine, please **DELETE or DISABLE** `SSDT-dSSD.aml` in your config.plist. Otherwise, your macOS installer will not see the drive.
### Kexts Configuration

| Kext | Function |
| :--- | :--- |
| `Lilu` `VirtualSMC` `RestrictEvents` | Core system patching. |
| `NootedRed` | **AMD Vega 8 Graphics Acceleration.** |
| `AirportItlwm` | Native Wi-Fi support. |
| `IntelBluetoothFirmware` `BlueToolFixup` `IntelBTPatcher` | Bluetooth support. |
| `AppleALC` | Audio (ALC287). |
| `VoodooI2C` `VoodooI2CHID` | Touchpad support. |
| `VoodooPS2Controller` | Keyboard support. |
| `RealtekRTL8111` | Ethernet support. |
| `ForgedInvariant` | CPU TSC Sync (AMD Speed fix). |
| `Innie` | Fixes PCI storage device naming. |
| `GenericUSBXHCI` `USBMap` | **Custom USB Mapping (Critical).** |
| `SMCProcessorAMD` `SMCRadeonSensors` | Temperature monitoring (Read-only). |
| `ECEnabler` `BrightnessKeys` `SMCBatteryManager` | Battery & Hotkeys. |
| `AppleMCEReporterDisabler` | Prevents kernel panic on AMD. |

### ACPI (SSDT) Configuration

| SSDT | Function |
| :--- | :--- |
| `SSDT-EC` | Embedded Controller fix. |
| `SSDT-PLUG-ALT` | CPU Power Management definition. |
| `SSDT-PNLF` | Screen Brightness control. |
| `SSDT-USBX` | USB Power properties. |
| `SSDT-XOSI` | OS Spoofing (simulates Windows). |
| `SSDT-dSSD` | **Hides incompatible SK Hynix SSD. Delete if you don't have this drive. (Note: To hide a different unsupported SSD, you must update the PCI path in this SSDT).** |
| `SSDT-ALS0` | Ambient Light Sensor. |
| `SSDT-PTSWAKTTS` | Sleep/Wake patch (Part 1). |
| `SSDT-EXT3-WakeScreen` | Sleep/Wake patch (Part 2). |
| `SSDT-SLEEP` | **Disables USB controller during sleep.** |

## Credits

* **Apple** for macOS.
* **Acidanthera** for OpenCore & Kexts.
* **ChefKissInc** for NootedRed.
* Special thanks to **[kalkmann](https://github.com/kalkmann/Legion-5600H-Hackintosh)** for the README layout, the Logo and inspiration.

<p align="center">
  <i>If this project helped you, please give it a star! ⭐️</i>
</p>
<p align="center">
  <img src="https://api.star-history.com/svg?repos=hoaug-tran/Lenovo-Legion-7-16ACHG6-Hackintosh&type=Date" alt="Star History Chart">
</p>
