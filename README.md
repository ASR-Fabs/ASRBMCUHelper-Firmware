# ASR BMCU Helper firmware

Signed firmware for the **ASR BMCU Helper** (ESP32) and the **ASR BMCU Bridge** (the BMCU's own firmware). Helpers on a home Wi-Fi check this repository when they start, and when **Check for updates** is pressed, and install updates from it. The source code is not published here.

Copyright (c) 2026 ASR-LLC. All rights reserved.

## Files

| File | What it is |
|---|---|
| `version.txt` | The current release: which Helper and which bridge go together. |
| `latest.bin` | The ASR BMCU Helper firmware, signed. |
| `bridge.bin` | The ASR BMCU Bridge firmware for that release, signed. |

`version.txt` has two lines:

```
helper 1.0.0
bridge 0.1.3
```

Each line is the word `helper` or `bridge`, a space, and the version (1 to 23 characters: letters, digits, `.`, `-`, `_`). Upper or lower case and Windows or Unix line endings are fine. The 5th Wheel's one-line form is not accepted here: both lines are needed.

## How a Helper updates

1. **The check.** It runs once after start-up (on the home Wi-Fi, with the BMCU connected) and whenever **Check for updates** is pressed on the Settings or Update page. The Helper reads `version.txt`.
2. **Only a newer Helper is installed automatically.** `1.0.0-dev.10` < `1.0.0-dev.11` < `1.0.0` < `1.0.1`. A release is newer than any `-dev.N` build of the same number.
3. **Only while the AMS is idle.** Nothing may be loading, unloading or printing, and the BMCU may not be flashing. If the AMS is busy, the update waits and tries again once it is idle. If the AMS becomes busy during a download, the download stops; nothing changes, and it tries again later.
4. **Every file is checked against the ASR signing key while it downloads.** The check covers the marker, the target (`helper` or `bridge`), the length and the signature. The version inside the signature must also match `version.txt`. Anything that fails is discarded, and the running firmware stays as it is.
5. **Bridge rule.** If `version.txt` names a different bridge from the one the BMCU runs, the Helper does not restart into the new version straight away:
   - it downloads `bridge.bin` into its flash storage;
   - it downloads the new Helper into its spare program slot;
   - it shows **BMCU update ready** on the app, the Settings page and the OLED.

   The user flashes the BMCU from the stored file (Flash BMCU page). Once the BMCU is back on the printer and reports the new bridge, the Helper checks the waiting firmware again and starts it, without a second download.
6. **Bridge-only releases** (same Helper, new bridge) just store `bridge.bin` and show **BMCU update ready**.

Any signed Helper `.bin` can also be installed by hand on the Helper's **/update** page, including an older one (a deliberate roll-back).

## Publishing a release

You need: the Arduino IDE, the Helper sketch, the signing tool (`asr_sign.html`, project asr_tools) and the private signing key file (`asr_signing_private_0f98efc30d3f1f68.pem`). **Never put the private key in this repository.**

1. **Set the versions.**
   - `HELPER_VERSION_NUM` in `asr_helper.ino`.
   - `ASR_BENCH 0` in `asr_build.h` for a release build.
   - If the bridge changed: `BRIDGE_VERSION` in the bridge's `bridge.h`. Keep the bridge version to 7 characters or fewer (for example `0.1.10`): the BMCU reports it in a 23-character field after "ASR BMCU Bridge ".
2. **Export the Helper.**
   - Tools > Board > **ESP32 Dev Module**.
   - Partition Scheme **Minimal SPIFFS (1.9MB APP with OTA/190KB SPIFFS)**.
   - Sketch > **Export Compiled Binary**.
   - Use `asr_helper.ino.bin`, not the bootloader, partitions or merged `.bin`.
3. **Sign the Helper.**
   - Open `asr_sign.html` and go to **Sign**.
   - Target **ASR BMCU Helper**; the version exactly as in step 1.
   - Rename the result (`asr_helper_<version>_signed.bin`) to `latest.bin`.
4. **Sign the bridge** (every release, even if it didn't change). Build the bridge `.bin`, then sign it with target **ASR BMCU Bridge** and its version. Rename the result to `bridge.bin`.
5. **Check both files** with the tool's **Check** section and your public key file.
6. **Upload in this order:**
   1. `latest.bin` and `bridge.bin` first;
   2. `version.txt` last.

   That way a Helper that checks in between never sees a `version.txt` that points at files that aren't there yet. (If it does, it refuses the mismatch and tries again at its next check.)
7. **Test on a unit:** Settings > **Check for updates**.

GitHub's raw file service may keep serving the old files for a few minutes after a push. If a unit still says "Up to date", wait five minutes and check again.

This repository must stay **public**: the Helpers download without logging in.

## First install

A new ESP32 is flashed once over USB (Arduino IDE, the board setting above). After that, updates come from here. The User Manual describes the first install.

## Third-party software in the Helper firmware

- ESP32 Arduino core (Espressif): LGPL-2.1.
- WiFiManager (tzapu): MIT.
- Adafruit SSD1306 and Adafruit GFX: BSD.
- The BMCU flasher is a port of BMCU-Flasher by Paweł Jarczak: MIT. Its notice is shown on the Helper's Flash BMCU page.

---
Document history: 1.0 (2026-11-11), written with ASR BMCU Helper 1.0.0-dev.10.
