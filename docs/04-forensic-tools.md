# Forensic & Exploit-Research Tool Catalog (Free vs Commercial)

## Free / open-source

### iOS
- **libimobiledevice** — `ideviceinfo`, `idevicebackup2`, `ideviceinstaller`,
  `ideviceprovision`, `ifuse`, `iproxy`. Enumeration + logical backups, no jailbreak.
- **checkra1n** — checkm8 jailbreak (A7–A11).
- **ipwndfu** — raw checkm8; pwned DFU, custom payloads, SSH ramdisk.
- **SSHRD_Script** — one-command checkm8 SSH ramdisk for clean full-FS extraction.
- **iLEAPP** — parses iOS backups/full-FS into HTML/CSV reports.
- **MVT (Mobile Verification Toolkit)** — Amnesty Intl; IOC/compromise scanning of
  backups & FS dumps. **Ideal for the "device exploit research" goal.**
- **objection** / **Frida** — runtime app exploration, keychain dump, SSL pinning
  bypass, class/method tracing.

### Android
- **ALEAPP** — Android artifact parser. **`mtkclient`** — MediaTek BROM flash R/W.
- **jadx**, **apktool** — APK static analysis. **Autopsy** — case management/carving.

### Cross-cutting RE
- **Ghidra** (free NSA disassembler/decompiler), **radare2 + Cutter**.
- **qiling**, **unicorn** — emulation. **binwalk** — firmware carving.
- **LLDB / gdb** — debugging. **mitmproxy** — network interception.

### Hardware
- **OpenOCD** (JTAG/SWD), **flashrom** (SPI), **sigrok/PulseView** (logic),
  **GNU Radio + gqrx** (SDR), **Ubertooth**/**nRF Sniffer** (BLE).

## Commercial (where it clearly wins)

| Tool | Sweet spot |
|------|-----------|
| **Elcomsoft iOS Forensic Toolkit** | One-click checkm8 full-FS + keychain (this iPod, all A7–A11); agent-based A12+. Best iOS value for research labs. |
| **Cellebrite UFED / Premium + Physical Analyzer** | Broadest device/bypass coverage, incl. locked A12+ and Android; strong parsing. |
| **Magnet AXIOM** | Excellent artifact analytics/timeline across iOS+Android+cloud. |
| **Oxygen Forensic Detective** | Wide app parsing, cloud, drone/IoT. |
| **GrayKey** | High-end iOS bypass (LE/gov only). |
| **Corellium** | Virtualized iOS/Android for exploit dev & fuzzing without hardware — top pick for A12+ research. |
| **IDA Pro / Binary Ninja** | Heavy static RE (Ghidra is the free substitute). |

## Recommendation for your mix

- **iPod 6G + other A7–A11:** free `SSHRD_Script`/`checkra1n` covers everything;
  add **EIFT** only to save time.
- **A12+ iOS research:** budget for **Corellium** (and EIFT agent for physical units).
- **Android:** free stack (`mtkclient`/EDL + ALEAPP) goes far; **Cellebrite/AXIOM**
  for locked/uncommon devices.
- **Embedded/medical:** all-free hardware stack; spend on a good **JTAG probe**,
  **logic analyzer (Saleae)**, and an **SDR (HackRF)**.
