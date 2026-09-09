# App Research — St. Jude Medical / Abbott "Patient Ctrl" (Patient Controller, Model 3875)

## Device identification

| Field | Value |
|-------|-------|
| App name | Patient Ctrl (Patient Controller) |
| Vendor | St. Jude Medical (now **Abbott**) |
| Model | **3875** (Patient Controller) — clinician counterpart is **3874** |
| Version | 3.9.Rev1.US |
| Build | 3.9.1.1.278 |
| GTIN | 05415067023681 |
| Host | iPod touch 6G, iOS 12.4.8 (locked-down appliance, but this unit is unmanaged/unused) |

**What it is:** the *patient*-facing controller app for Abbott/St. Jude
**neurostimulation** systems (spinal cord stimulation and deep-brain stimulation —
Proclaim, Infinity, Prodigy, Protégé families). It talks to an **implantable pulse
generator (IPG)** over **Bluetooth Low Energy (BLE)** to let the patient adjust
therapy within clinician-set limits. The Model 3874 *Clinician* Programmer is the
higher-privilege sibling.

**Research scope here:** with no IPG paired and no patient data, the value is the
**app binary + its BLE protocol + pairing/authentication design** — a classic
medical-device security research target. There is nothing evidentiary to preserve.

## Responsible-research guardrails (read first)

- This is **patient-safety-critical**. Do all work on the bench, on your own unit,
  with **no implanted or patient-connected IPG** in range.
- If you find a real vulnerability, use **coordinated disclosure**: Abbott PSIRT,
  and report to **FDA** and **CISA** (ICS Medical Advisories). Prior Abbott/St. Jude
  advisories (e.g., the cardiac Merlin@home line, ICSMA-17-241-01) show they run a
  disclosure process — check CISA's ICS-Medical database for current contacts and any
  existing neurostim advisories before starting.
- Do **not** transmit crafted BLE traffic at a real device you don't own/control.

## 1. Get the decrypted app off the device

The iPod is checkm8 — jailbreak with checkra1n (see `docs/01`), then:

```bash
# decrypt + pull the .ipa (App Store binaries are FairPlay-encrypted at rest):
frida-ios-dump -H localhost -p 2222 "Patient Ctrl"     # or the exact bundle id
# find the bundle id first if unsure:
ideviceinstaller -l | grep -i patient
```

Also grab, from the full-FS dump:
- Bundle: `/private/var/containers/Bundle/Application/<GUID>/Patient Ctrl.app/`
- Data container: `/private/var/mobile/Containers/Data/Application/<GUID>/`
  (Preferences plists, Caches, any bundled config/limits, logs).

## 2. Static analysis (Ghidra / class-dump)

```bash
unzip Patient_Ctrl.ipa && cd "Payload/Patient Ctrl.app"
codesign -d --entitlements :- "Patient Ctrl"       # bluetooth + background modes?
otool -L "Patient Ctrl"                             # linked frameworks
class-dump -H "Patient Ctrl" -o /tmp/headers        # ObjC class/method surface
strings -a "Patient Ctrl" | grep -iE 'uuid|servic|charact|pair|bond|key|crypt|token|IPG|stim|abbott|jude'
```

Load the Mach-O in **Ghidra** and focus on:
- **CoreBluetooth** usage — hardcoded **service/characteristic UUIDs**, the GATT map.
- **Pairing/bonding & auth** — is the link authenticated/encrypted, or just BLE
  Just-Works? Any app-layer challenge/response or shared secret?
- **Crypto & secrets** — embedded keys, certs, key-derivation, SSL pinning.
- **Therapy limits** enforcement — are stimulation bounds enforced app-side (client
  trust) or by the IPG? Client-side-only limits are the interesting finding.
- **Jailbreak / integrity detection** — note it so you can bypass it dynamically.

## 3. Dynamic analysis with Frida (live BLE protocol)

Hook CoreBluetooth to log exactly what the app scans for, connects to, and
reads/writes — this reconstructs the protocol without needing to reverse every byte
statically:

```js
// frida -U -f <bundle-id> -l cb.js   (or attach with -n)
for (const sel of [
  "- centralManager:didDiscoverPeripheral:advertisementData:RSSI:",
  "- peripheral:didDiscoverServices:",
  "- peripheral:didDiscoverCharacteristicsForService:error:",
  "- peripheral:didUpdateValueForCharacteristic:error:"]) {
  try { Interceptor.attach(ObjC.classes /* resolve via ApiResolver */); } catch(e){}
}
// Practical route: use `objection -g <bundle-id> explore` then
//   ios hooking watch class CBPeripheral
//   ios hooking watch class CBCentralManager
// and dump NSData args (writeValue:forCharacteristic:type:) to see command frames.
```

Also via **objection**: `ios keychain dump`, `ios plist cat`, and
`ios jailbreak disable` (defeats the app's JB detection so it will run).

## 4. BLE at the radio layer (when you add a test IPG/base)

Without an IPG you can still capture the app's advertising/scan behavior. With a
**bench IPG or programmer base** you control:
- **nRF52840 dongle + nRF Sniffer**, **Ubertooth One**, or Android **BLE HCI snoop
  log** to capture the connection, then dissect in **Wireshark**.
- Confirm: connection encryption/pairing method, whether therapy commands are
  replayable, and whether the IPG rejects out-of-range values independently of the app.

## 5. Where to write findings

- Protocol map (services/characteristics, frame formats) → `research/3875-ble-protocol.md`
- Static-analysis notes (auth, crypto, limits) → `research/3875-static-notes.md`
- Keep any disclosure correspondence out of the repo.
