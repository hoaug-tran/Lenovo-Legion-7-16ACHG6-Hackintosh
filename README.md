<p align="center">
  <img width="150px" src="https://rallybr.com.br/logo-hacklegion.png" align="center" alt="Hackintosh Logo" />
</p>

<h1 align="center">Lenovo Legion 7 (16ACHg6) Hackintosh</h1>
<h3 align="center">Ryzen 9 5900HX | Vega 8 (NootedRed) | macOS Ventura 13.6</h3>

<p align="center">
  <img src="https://img.shields.io/badge/BIOS-GKCN65WW-blue?style=flat-square&logo=lenovo&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenCore-1.0.6-8A2BE2?style=flat-square&logo=acidanthera&logoColor=white" />
  <img src="https://img.shields.io/badge/macOS-Ventura-success?style=flat-square&logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/macOS-Sonoma-success?style=flat-square&logo=apple&logoColor=white" />
  <img src="https://img.shields.io/badge/macOS-Sequoia-success?style=flat-square&logo=apple&logoColor=white" />
</p>

<hr />

## Disclaimer & Warnings

> [!IMPORTANT]
> **Reference Only**
> This repository documents my personal configuration. **Do not copy-paste this EFI directly.** You should build your own EFI following the Dortania Guide.
>
> **SMBIOS Required:** If you decide to use this EFI, you **must** generate your own Serial Number, Board Serial, and SmUUID using [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS). The values in this config are generic and invalid.
>
> **macOS Version:** This setup is validated for **macOS Ventura (13.6)**. Newer versions (Sonoma/Sequoia) have not been tested thoroughly by me, though they likely work with minor adjustments to Wi-Fi kexts, AMD Kernel Patches, ...

> [!WARNING]
> **Data Safety:** Modifying BIOS and EFI settings involves risk. Always backup your data before proceeding.

---

## Hardware Specifications & Status

| Component | Specification | Status | Notes |
| :--- | :--- | :--- | :--- |
| **Processor** | AMD Ryzen 9 5900HX | **Working** | Power management via firmware. |
| **Graphics** | AMD Radeon Vega 8 | **Working** | Full acceleration via NootedRed. |
| **Discrete GPU** | NVIDIA RTX 3080 | **Disabled** | Unsupported. Disabled via software (boot-arg). |
| **Display** | 16" WQXGA (2560x1600) | **Working** | Max 120Hz (Panel is 165Hz). |
| **External Video** | HDMI / USB-C DP | **Not Working** | Hardwired to the disabled NVIDIA GPU. |
| **Audio** | Realtek ALC287 | **Working** | Speakers and Mic functioning. |
| **Wi-Fi** | Intel Wi-Fi 6 AX210 | **Working** | Native speeds via AirportItlwm. |
| **Bluetooth** | Intel Bluetooth | **Working** | |
| **Ethernet** | Realtek RTL8111 | **Working** | |
| **Sleep/Wake** | S3 Sleep | **Partial** | Wake via Power Button / Lid Open only. |
| **Trackpad** | I2C HID | **Working** | Full multi-touch gestures. |
| **Storage** | SUNEAST Gen4 G55 | **Working** | |
| **Storage (OEM)** | SK Hynix PC711 | **Incompatible** | Must be disabled (See Critical Notes). |

---

## Repository Structure

This repository may contain two EFI versions:
1. **EFI Standard:** Stable OpenCore version.
2. **EFI MOD (Dev):** [OpenCore Mod](https://github.com/wjz304/OpenCore_NO_ACPI_Build) version. This development version includes experimental fixes for cosmetic issues like the "Blue Screen" glitch and addresses dual-boot problems with Windows or other OSes.

---

## Critical Configuration Notes

### 1. NVIDIA RTX 3080 & Fan Spin Issue
Do **NOT** use `SSDT-dGPU-Off` or `SSDT-NoHybGfx`.
Using these ACPI patches will cause the fans to spin at maximum speed when booting macOS, followed by an automatic shutdown after 1 minute.
* **Solution:** I use the boot argument `-wegnoegpu`. This disables the discrete GPU safely at the kernel level.

### 2. Incompatible SK Hynix SSD
The stock SK Hynix PC711 SSD causes immediate Kernel Panics in macOS.
* **Solution:** `SSDT-dSSD.aml` is included to hide this specific drive.
* **Action:** If you do not have this drive, **delete** this SSDT, or your compatible drive will be hidden.

### 3. Display Refresh Rate (165Hz vs 120Hz)
The display is limited to **120Hz** in macOS.
* **Reason:** The 165Hz timing requires high bandwidth and compression (DSC) that the Vega 8 driver (Nootedred problems maybe) does not support over eDP.
* **Fix:** Copy the provided display override folder to `/Library/Displays/Contents/Resources/Overrides/` to ensure HiDPI and 120Hz options are available.

---

## Fixing Sleep

This section details the troubleshooting process to achieve stable sleep on the Ryzen 5900HX.

**Phase 1: The Wake Freeze**
Initially, sleep worked, but waking the laptop caused a complete system freeze.
* **Fix:** Switched from `ForgedInvariant` to **`CpuTscSync.kext`**. This resolved the Time Stamp Counter synchronization issue on AMD mobile CPUs.

**Phase 2: The Instant Wake**
After fixing the freeze, the laptop would sleep but immediately wake up due to electrical noise from the internal USB hub (GPE 0x19). Software filters (`GPRW`, `USBWakeFixup`) failed to consistently block this noise.

**Phase 3: The Stable Solution (SSDT-SLEEP)**
I implemented `SSDT-SLEEP` to force the USB controller off (`_STA=0`) during sleep.
* **Result:** Stable sleep, no battery drain, no heat.
* **Limitation:** You cannot wake the device with the keyboard or mouse.
* **How to Wake:** Use the **Power Button** or **Open the Lid**.

**Helper SSDTs:**
* `SSDT-PTSWAK`: Forces the dGPU off immediately after wake.
* `SSDT-EXT3-WSC`: Fixes the "Black Screen" issue upon wake.

> [!NOTE]
> **Hackintool USB Note:** When using `SSDT-SLEEP`, the USB tab in Hackintool may show ports as disconnected or empty. This is expected behavior because the controller is disabled during sleep logic. Do not worry, your physical USB ports still function correctly.

---

## USB Configuration Strategy

I rely on **`USBPorts.kext`** for a clean, driver-less map.

* **Coverage:** All physical Type-A, Type-C, and internal devices (Bluetooth, Camera) are mapped correctly.
* **Topology:** On this Ryzen platform, USB 3.0 and 2.0 ports are routed through internal Hubs (e.g., USB 3.2 Hub / USB 2.1 Hub). This is normal architecture.
  
---

## Configuration Details

### Kexts
| Name | Description |
| :--- | :--- |
| **NootedRed** | Graphics acceleration for Vega 8. |
| **CpuTscSync** | Critical fix for wake freezing. |
| **AirportItlwm** | Wi-Fi support. |
| **IntelBluetoothFirmware** | Bluetooth firmware. |
| **AppleALC** | Audio driver. |
| **VoodooI2C / HID** | Trackpad support. |
| **VoodooPS2Controller** | Keyboard support. |
| **RealtekRTL8111** | Ethernet support. |
| **USBPorts** | Custom USB Map. |
| **Innie** | PCI device naming fix. |
| **RestrictEvents** | AMD patches. |
| **SMCAMDProcessor** | CPU temperature monitoring. |
| **SMCBatteryManager** | Battery status. |

### ACPI (SSDTs)
| Name | Description |
| :--- | :--- |
| **SSDT-PTSWAK** | Manages Sleep/Wake lifecycle and dGPU state. |
| **SSDT-EXT3-WSC** | Fixes Black Screen on wake. |
| **SSDT-SLEEP** | Disables USB during sleep (Fixes Instant Wake). |
| **SSDT-dSSD** | Hides incompatible SK Hynix SSD. |
| **SSDT-PLUG-ALT** | CPU Power Management. |
| **SSDT-PNLF** | Backlight Control. |
| **SSDT-ALS0** | Fake Ambient Light Sensor. |
| **SSDT-EC** | Embedded Controller fix. |
| **SSDT-USBX** | USB Power injection. |

---
### Recommended Apps & Tools

Here are some essential tools to improve your Hackintosh experience on the Legion 7:

| App | Description | Link |
| :--- | :--- | :--- |
| **Karabiner-Elements** | **Essential.** Use this to remap Function Keys (F1-F12) to work correctly like a real Mac (Brightness, Volume, etc.). | [Website](https://karabiner-elements.pqrs.org/) |
| **Maccy** | The best lightweight, open-source clipboard manager. Beautiful UI/UX and very fast. | [GitHub](https://github.com/p0deje/Maccy) |
| **Macs Fan Control** | Monitor CPU/GPU temperatures and control fans (Sensor support depends on `SMCProcessorAMD` kext). | [Website](https://crystalidea.com/macs-fan-control) |
| **RunCat** | A cute utility that visualizes system usage (CPU/RAM/Network) with a running cat (or other animations) in the menu bar. | [Website](https://kyome.io/runcat/index.html?lang=en) |

---
## Credits

* **Apple** for macOS.
* **Acidanthera** for OpenCore and Kexts.
* **ChefKissInc** for NootedRed.
* **CorpNewt** for GenSMBIOS, SSDTTime, and ProperTree.
* **Seey6** for CpuTscSync.
* **Ghost-Cavendish** for the research on Ryzen 5000 sleep issues.
* Special thanks to **kalkmann** for the README layout and Logo inspiration.

---
<p align="center">
  <i>If this project helped you, please give it a star! ⭐️</i>
</p>
<p align="center">
  <a href="https://star-history.com/#hoaug-tran/Lenovo-Legion-7-16ACHG6-Hackintosh&Date">
    <img src="https://api.star-history.com/svg?repos=hoaug-tran/Lenovo-Legion-7-16ACHG6-Hackintosh&type=date&legend=top-left&refresh=fix_ugly" alt="Star History Chart">
  </a>
</p>