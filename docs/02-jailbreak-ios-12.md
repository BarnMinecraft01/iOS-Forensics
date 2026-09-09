# Jailbreak Options — iOS 12.x (focus: A8 / iOS 12.4.8)

## Decision rule for A8 (iPod touch 6G)

Use **`checkm8`**. It is a BootROM exploit → **unpatchable**, version-independent,
and works on 12.4.8 exactly as on any other 12.x build. Everything else is a
distant second.

| Tool | Exploit basis | A8 + 12.4.8 | Persistence | Best for |
|------|---------------|-------------|-------------|----------|
| **checkra1n** | checkm8 (BootROM) | ✅ Best | Semi-tethered (re-run on boot) | Full jailbreak, research shell, Frida |
| **ipwndfu** | checkm8 (raw) | ✅ | Ramdisk (nothing persisted) | Clean acquisition, custom payloads |
| **SSHRD_Script** | checkm8 (ramdisk) | ✅ | Ramdisk | Forensic full-FS extraction |
| **Chimera** | checkm8 variant | ✅ | Semi-tethered | Alt package manager (Sileo) |
| **unc0ver** | `sock_puppet` (software) | ⚠️ Unreliable | Semi-untethered | **Avoid** — bug patched at 12.4.1 |

**Bottom line:** `checkra1n` for a working jailbroken research device; `ipwndfu` /
`SSHRD_Script` when you want to extract data without jailbreaking the on-disk OS.

## checkm8 mechanics (why it matters for research)

- Triggered in **DFU mode**; exploits a use-after-free in the USB stack of the
  BootROM before signature checks fully lock down.
- Gives you a **pwned DFU** state → you can send unsigned iBSS/iBEC/kernel/ramdisk.
- Because it precedes the OS, it's the foundation for **kernel debugging** and
  clean forensic imaging alike.

## Semi-tethered reality

checkm8-based jailbreaks are **semi-tethered**: after a power cycle the device boots
stock until you re-run the exploit. For research, script the re-jailbreak and keep
the device on stable power / in a controlled state during a session.

## Legal / safety notes

- This is your device and a **research** context — fine. Do **not** run jailbreak or
  RF steps against a device that is **actively connected to a patient** or an
  implanted stimulator. Use bench units.
- Keep the unit RF-isolated during work so it can't command the medical hardware.
