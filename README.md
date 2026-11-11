# ASR BMCU Helper firmware

Signed firmware for the **ASR BMCU Helper** (ESP32) and the **ASR BMCU Bridge** (the BMCU's own firmware). Helpers on a home Wi-Fi check this repository when they start and when **Check for updates** is pressed, and install updates from it. The source code is not published here.

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

Each line is the word `helper` or `bridge`, a space, and the version (1 to 23 characters: letters, digits, `.`, `-`, `_`). Upper or lower case and Windows or Unix line endings are fine.

## How a Helper updates

1. **The check.** About 20 seconds after power-on (on the home Wi-Fi, with the BMCU connected), and whenever **Check for updates** is pressed, the Helper reads `version.txt`. That's all it downloads while it runs.
2. **Only a newer Helper counts as an update.** `1.0.0-dev.15` < `1.0.0-dev.16` < `1.0.0` < `1.0.1`. A release is newer than any `-dev.N` build of the same number. A different bridge in `version.txt` is also an update.
3. **The update start-up, as the 5th Wheel.** The files are downloaded in a special start-up that runs only Wi-Fi and the OLED. Nothing else runs: no web pages, no slot control and no BMCU link. That leaves plenty of memory for the secure connection. It takes about a minute, and the AMS is off the printer meanwhile.
   - **After power-on:** if the check finds an update and the AMS is idle, the Helper restarts into the update start-up by itself. If the AMS is busy, it waits until it is idle.
   - **After Check for updates:** the Settings and Update pages show **Update available** and a **Restart to update** button. Nothing happens until it is pressed, and it only works while the AMS is idle.
4. **Every file is checked against the ASR signing key while it downloads.** The check covers the marker, the target (`helper` or `bridge`), the length and the signature. The version inside the signature must also match `version.txt`. A file that fails is discarded and the running firmware stays.
5. **If the update start-up fails** (Wi-Fi, download or check), the Helper restarts into the version it had and the pages say why. It then doesn't restart to update by itself again until the next power-on; **Check for updates** and **Restart to update** try again.
6. **Bridge rule.** If `version.txt` names a different bridge from the one the BMCU runs, the update start-up does not install the new Helper:
   - it downloads `bridge.bin` into the Helper's flash storage;
   - it downloads the new Helper into the spare program slot;
   - the Helper comes back on its current version and shows **BMCU update ready**.

   The user flashes the BMCU from the stored file (Flash BMCU page). Once the BMCU is back on the printer and reports the new bridge, the Helper checks the waiting firmware again and starts it, without a second download.
7. **Bridge-only releases** (same Helper, new bridge) store `bridge.bin` the same way and show **BMCU update ready**.

Any signed Helper `.bin` can also be installed by hand on the Helper's **/update** page, including an older one (a deliberate roll-back).

## First install

A new ESP32 is flashed once over USB (Arduino IDE, the board setting above). After that, updates come from here. The User Manual describes the first install.

## Third-party software in the Helper firmware

- ESP32 Arduino core (Espressif): LGPL-2.1.
- WiFiManager (tzapu): MIT.
- Adafruit SSD1306 and Adafruit GFX: BSD.
- The BMCU flasher is a port of BMCU-Flasher by Paweł Jarczak: MIT. Its notice is shown on the Helper's Flash BMCU page.

---
Document history: 1.0 (2026-11-11), written with ASR BMCU Helper 1.0.0-dev.10. 1.1 (2026-11-11): the update start-up and Restart to update (Helper 1.0.0-dev.15).
