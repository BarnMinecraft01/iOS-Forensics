# Device Matrix — Tool Fit by Platform

You indicated future work spans **newer iOS (A12+)**, **other checkm8 iOS (A7–A11)**,
**Android**, and **embedded/medical hardware**. Each needs a different stack.

## Quick chooser

| Platform | Code-exec entry | Acquisition (free) | Acquisition (commercial) | Research tooling |
|----------|-----------------|--------------------|--------------------------|------------------|
| **checkm8 iOS (A7–A11)** incl. iPod6G | checkm8 (unpatchable) | SSHRD_Script, ipwndfu, checkra1n + tar | Elcomsoft EIFT | Frida, Ghidra, LLDB |
| **A12+ iOS** | Software vuln / signed agent | idevicebackup2 (logical only) | EIFT agent, Cellebrite Premium, GrayKey | Corellium, Frida (on JB) |
| **Android** | Bootloader/EDL/MTK | ADB, mtkclient, EDL dd, ALEAPP | Cellebrite, Magnet AXIOM, Oxygen | Frida, objection, jadx |
| **Embedded/medical** | JTAG/UART/SWD/flash | flashrom, OpenOCD, binwalk | JTAGulator, chip-off rigs | Ghidra, SDR, logic analyzer |

## A12+ iOS (no checkm8!)

The A12/A12+ **fixed the checkm8 UAF** — the free BootROM path is gone. Options:

- **Full-FS + keychain:** needs a **software exploit jailbreak** or a **signed
  extraction agent**:
  - Jailbreaks: **Taurine** / **unc0ver** (≤14.3), **Dopamine** (iOS 15–16, arm64e
    via `kfd`), **Fugu15/Fugu16** (research-grade).
  - Agent-based (no jailbreak): **Elcomsoft EIFT agent** (sideloaded, dev-account
    signed), **Cellebrite Premium**, **GrayKey** (LE only).
- **No exploit available for that build:** you are limited to **logical**
  (`idevicebackup2 --full`) + iCloud/`.plist` artifacts.
- **Research without hardware:** **Corellium** virtualizes modern iOS — the single
  best tool for A12+ dynamic analysis and exploit dev.

## Other checkm8 iOS (A7–A11)

Identical workflow to the iPod: `SSHRD_Script` / `ipwndfu` for clean full-FS,
`checkra1n` for a jailbroken shell, EIFT for the commercial one-click path.
Covers iPhone 5s→X, many iPads, iPod5/6, Apple TV 4.

## Android

- **Logical:** `adb backup` (deprecated/limited), `adb pull` of accessible dirs,
  content-provider dumps.
- **Physical:**
  - **Qualcomm:** **EDL (9008) mode** + firehose loaders → full `dd` images.
  - **MediaTek:** **`mtkclient`** (BROM exploit) → read/write full flash.
  - Bootloader-unlockable: flash **TWRP**, then `dd` partitions over ADB.
- **Parsing:** **ALEAPP** (Android Logs Events And Protobuf Parser), **Autopsy**.
- **App research:** **jadx**/**apktool** (static), **Frida + objection** (dynamic),
  **Burp/mitmproxy** (network).
- **Commercial:** Cellebrite, Magnet AXIOM, Oxygen — broadest device/bypass support.

## Embedded / medical hardware (the stimulator, base station, dongles)

Treat as generic embedded RE + hardware forensics:

- **Interfaces:** UART (console), **JTAG/SWD** (halt/dump via **OpenOCD** + a
  probe like a J-Link/ST-Link/FT2232), SPI/I²C flash.
- **Flash extraction:** **`flashrom`** (SPI NOR), chip-off + programmer for eMMC,
  **`mtkclient`**/**EDL** if it's a Qualcomm/MTK SoC inside.
- **Recon aids:** **JTAGulator** / **Bus Pirate** / **Shikra** / logic analyzer
  (Saleae/sigrok) to find pinouts and protocols.
- **Firmware RE:** **`binwalk`** to carve/unpack, **Ghidra** to disassemble, **`qiling`**
  to emulate.
- **RF telemetry:** medical device links often use **MedRadio/MICS (~402–405 MHz)**,
  proprietary sub-GHz, or **BLE**. Analyze with **HackRF/RTL-SDR + GNU Radio**;
  BLE with **nRF Sniffer**/**Ubertooth**. **Transmitting on medical bands is
  regulated — capture/receive only unless properly authorized.**

> ⚠️ **Medical safety:** never power or stimulate hardware that could still drive a
> patient-facing output. Bench units, dummy loads, and RF isolation only.
