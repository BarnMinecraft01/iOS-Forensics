# iOS-Forensics

Working notes and workflows for forensic acquisition and **security/exploit
research** on mobile & embedded devices. Initial focus: an **iPod touch 6th gen
(iPod7,1) on iOS 12.4.8** fielded as a medical **patient programmer/controller**.

> **Scope & ethics:** research on devices you own/are authorized to test. Do not
> operate against hardware actively connected to a patient; keep units RF-isolated;
> receive-only on regulated medical RF bands unless authorized to transmit.

## The one thing that matters for the iPod: `checkm8`
The A8 chip is vulnerable to the **unpatchable BootROM exploit `checkm8`**, so
iOS 12.4.8 is no obstacle — you get repeatable code execution and clean full-file-
system extraction without modifying the on-disk OS.

## Docs
| # | Doc | Covers |
|---|-----|--------|
| 01 | [iPod touch 6G imaging](docs/01-ipod-touch-6g-imaging.md) | Step-by-step logical → checkm8 ramdisk → jailbreak acquisition, keychain, commands |
| 02 | [Jailbreak options (iOS 12.x)](docs/02-jailbreak-ios-12.md) | checkra1n / ipwndfu / SSHRD vs unc0ver, decision rule for A8 |
| 03 | [Device matrix](docs/03-device-matrix.md) | Tool fit for checkm8 iOS, **A12+ iOS**, **Android**, **embedded/medical** |
| 04 | [Forensic tool catalog](docs/04-forensic-tools.md) | Free vs commercial, with recommendations for your device mix |
| 05 | [Exploit-research workflow](docs/05-exploit-research.md) | Frida/objection, Ghidra, LLDB, RF/network, MVT |

## Quick start (iPod touch 6G, no passcode)
```bash
ideviceinfo                                   # confirm iPod7,1 / 12.4.8
git clone https://github.com/verygenericname/SSHRD_Script && cd SSHRD_Script
./sshrd.sh 12.4.8 && ./sshrd.sh boot          # DFU -> ramdisk (on-disk OS untouched)
./sshrd.sh ssh                                # root@localhost / alpine
# mount data volume, stream full-FS out, hash it, then parse with iLEAPP + MVT
```

## Templates
- [Acquisition log](templates/acquisition-log.md)
