# Imaging the iPod touch 6G (iOS 12.4.8) — Patient Programmer/Controller

> **Context:** Device fielded as a medical *patient programmer / controller*. Purpose
> of this workflow: **security & exploit research** (studying the controller app and
> iOS internals), **no user passcode set**. Toolchain: free-first, with commercial
> callouts where they clearly win.

## 0. Why this device is easy: `checkm8`

The iPod touch 6G uses the **Apple A8** SoC. A8 is in the range covered by
**`checkm8`** (axi0mX), a **BootROM exploit that Apple cannot patch in software**.
Consequences:

- The iOS version (12.4.8) is **irrelevant** to gaining code execution — the entry
  point is in read-only silicon.
- You can boot a **custom RAM disk** and extract the file system **without modifying
  the on-disk OS / user partition** — the most forensically clean path.
- It is **repeatable** and survives OS updates/reboots.

## 1. Pre-acquisition (do this first)

1. **Photograph & log** the unit, serial, screen state. Start an acquisition log
   (see `templates/acquisition-log.md`) and keep SHA-256 of every image you produce.
2. **Isolate RF**: keep it in a Faraday bag / airplane mode until acquisition to
   prevent remote wipe or telemetry to the medical base station.
3. **Enumerate before touching anything** (no jailbreak needed):
   ```bash
   ideviceinfo                      # UDID, iOS build, model (iPod7,1 = 6G)
   ideviceinfo -k PasswordProtected # confirm the "no passcode" assumption
   ideviceinfo -k HostAttached
   # Supervision / MDM state (medical units are often supervised/kiosk):
   ideviceinfo -k IsSupervised 2>/dev/null || true
   ideviceprovision list            # provisioning profiles (the medical app is signed)
   ideviceinstaller -l              # installed apps -> find the controller app bundle ID
   ```
4. Note whether it is in **Single App / kiosk mode** or MDM-managed — this shapes
   what a logical backup exposes.

## 2. Tier 1 — Logical (no jailbreak, quickest, non-invasive)

Good baseline; misses a lot but touches nothing.

```bash
idevicepair pair
# Full iTunes-style backup (encrypted backups expose MORE data, so set one):
idevicebackup2 backup encryption on -i        # set a known backup password, e.g. "research"
idevicebackup2 backup --full ./ipod6g-backup/
idevicebackup2 info -s ./ipod6g-backup/        # verify
```

Parse it:
```bash
pip install mvt ileapp
mvt-ios decrypt-backup -p research -d ./decrypted ./ipod6g-backup/<UDID>
mvt-ios check-backup   --output ./mvt-out ./decrypted        # IOC / anomaly scan
ileapp -t itunes -i ./ipod6g-backup/<UDID> -o ./ileapp-out   # readable report
```

Media/AFC without a backup:
```bash
ifuse ./mnt            # or: afcclient ls /
```

## 3. Tier 2 — checkm8 SSH ramdisk (RECOMMENDED, clean full-FS)

Boots a RAM disk over the BootROM exploit. **The device's own OS/user partition is
not modified** — you SSH into a ramdisk and copy the mounted data volume.

```bash
git clone https://github.com/verygenericname/SSHRD_Script && cd SSHRD_Script
./restart_usbmuxd.sh
./sshrd.sh 12.4.8         # builds ramdisk for this iOS; requires the matching IPSW/keys
./sshrd.sh boot           # put iPod in DFU when prompted -> boots ramdisk
# In another terminal:
./sshrd.sh ssh            # -> root@localhost (password: alpine)
```

Inside the ramdisk, mount and image the data partition:
```bash
# identify the data slice (usually disk0s1s2 -> /dev/disk0s1s2), then:
mount_apfs /dev/disk1s1 /mnt
# stream a full tar image OUT over the SSH tunnel to the workstation:
tar -czf - -C /mnt . | ssh -p 2222 workstation "cat > /evidence/ipod6g_fullfs.tar.gz"
```
Then on the workstation:
```bash
sha256sum /evidence/ipod6g_fullfs.tar.gz | tee ipod6g_fullfs.sha256
ileapp -t fs -i /evidence/ipod6g_fullfs.tar.gz -o ./ileapp-fs-out
```

**Manual alternative with `ipwndfu`** (axi0mX reference tooling):
```bash
git clone https://github.com/axi0mX/ipwndfu && cd ipwndfu
sudo python3 ipwndfu -p          # pwned DFU via checkm8
# then load an SSH ramdisk payload and proceed as above
```

## 4. Tier 3 — checkra1n jailbreak (max convenience, modifies running OS)

Use when you want a persistent research shell and on-device tooling (Frida, etc.).
Slightly less clean than the ramdisk because it modifies the booted OS.

```bash
# Linux CLI (also has a GUI, and a macOS app):
sudo checkra1n -c            # enter DFU when prompted; A8/iPod7,1 + 12.4.8 fully supported
# after jailbreak, install OpenSSH from the checkra1n loader / Cydia, then:
iproxy 2222 22 &
ssh root@localhost -p 2222   # password: alpine  (CHANGE IT)
tar -czf - /private/var | ssh -p 2222 workstation "cat > /evidence/ipod6g_var.tar.gz"
```

## 5. Keychain (no passcode set)

Even with **no passcode**, some keychain items use device-only protection classes
(`kSecAttrAccessibleAlwaysThisDeviceOnly`, etc.) that are still recoverable on a
checkm8 device because you have the device keys:

```bash
# On a checkra1n-jailbroken device, via objection (Frida-based):
objection -g "<controller-app-bundle-id>" explore
#   ios keychain dump
# or a dedicated keychain_dumper build over SSH.
```
The medical controller app's **pairing keys / device tokens** typically live here or
in its app-group container — a prime research target.

## 6. Commercial shortcut

**Elcomsoft iOS Forensic Toolkit (EIFT)** automates the entire checkm8 ramdisk
→ full-FS + keychain extraction for this exact device class, with less manual
plumbing. If you have (or can get) a license, it's the fastest reliable path;
otherwise the free ramdisk workflow above gives equivalent data.

## 7. Post-acquisition targets for the controller app

- App container: `/private/var/mobile/Containers/Data/Application/<GUID>/`
  (Documents, Library/Preferences plists, Caches, and any SQLite telemetry DBs).
- App bundle: `/private/var/containers/Bundle/Application/<GUID>/<App>.app` — pull
  the Mach-O for static analysis (see `05-exploit-research.md`).
- Unified logs, `com.apple.mobile.installation.plist`, MDM profiles.
