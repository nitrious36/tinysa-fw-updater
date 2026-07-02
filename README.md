# tinySA Firmware Updater

**by ZS6ORB · v1.1.6**

A console tool for **Windows, Linux and macOS** (x64 and ARM64) that updates the
firmware on a **tinySA** or **tinySA Ultra** spectrum analyser. It talks to the
unit over USB, checks whether a newer firmware build is available, downloads it,
and flashes it over USB DFU — with a clear on-screen summary at every step.

```
     _____ _               ____    _
    |_   _(_)_ __  _   _  / ___|  / \
      | | | | '_ \| | | | \___ \ / _ \
      | | | | | | | |_| |  ___) / ___ \
      |_| |_|_| |_|\__, | |____/_/   \_\
                   |___/
        ▁▂▃▅▇▆▄▂▃▇█▇▃▂▁▂▄▆▇▅▃▂▁   tinySA™

   ╔══════════════════════════════════════════════════════╗
   ║            F I R M W A R E   U P D A T E R            ║
   ║                     by ZS6ORB                        ║
   ╚══════════════════════════════════════════════════════╝
              v1.1.6    for tinySA  &  tinySA Ultra
```

---

## Download

Grab the self-contained build for your machine from the **[`binaries/`](binaries/)**
folder — the .NET runtime is bundled, so **no .NET install is required**:

| OS | x64 | ARM64 |
|----|-----|-------|
| **Windows** | [`binaries/windows/x64/TinySAUpdater.exe`](binaries/windows/x64/TinySAUpdater.exe) | [`binaries/windows/arm64/TinySAUpdater.exe`](binaries/windows/arm64/TinySAUpdater.exe) |
| **Linux** | [`binaries/linux/x64/TinySAUpdater`](binaries/linux/x64/TinySAUpdater) | [`binaries/linux/arm64/TinySAUpdater`](binaries/linux/arm64/TinySAUpdater) |
| **macOS** | [`binaries/macos/x64/TinySAUpdater`](binaries/macos/x64/TinySAUpdater) | [`binaries/macos/arm64/TinySAUpdater`](binaries/macos/arm64/TinySAUpdater) |

Prefer a packaged download? Each platform also has a zip on the
[latest release](https://github.com/nitrious36/tinysa-fw-updater/releases/latest).
See [`binaries/README.md`](binaries/README.md) for per-platform notes.

---

## What it does

1. **Checks internet connectivity** and that the firmware server is reachable,
   before doing anything else. If it's offline (or you pass `--offline`), it
   falls back to flashing already-downloaded firmware from the `Firmware\` folder.
2. **Detects the unit** over its USB serial (COM) port by USB VID/PID (`0483`/`5740`)
   and reads the running firmware with the `version` command.
3. **Auto-detects the model** from that version string:
   `tinySA4_…` → tinySA Ultra, `tinySA_…` → tinySA (basic).
4. **Compares versions** — the installed build vs. the latest `.bin` on the
   official server. It compares the **version string**, not file dates.
5. **Downloads only when newer.** If the unit is already current, it stops and
   says so (use `--force` to re-flash anyway).
6. **Fetches `dfu-util`** automatically on Windows (bundled `dfu-util-static.exe`);
   on Linux/macOS it uses the system `dfu-util` from your package manager.
7. **Renames** the download to `tinySA4.bin` / `tinySA.bin` for flashing, while
   keeping the original versioned `.bin` as an on-disk record.
8. **Flashes** with `dfu-util -a 0 -s 0x08000000:leave -D <bin>`,
   streaming the live output.
9. **Prints a summary** of every check and **stays open** until you press `[Q]`.

---

## Requirements

- **Windows** 10/11 (x64 or ARM64), a modern 64-bit **Linux** distro (x64/ARM64),
  or **macOS** 11+ (Intel or Apple Silicon).
- The tinySA connected by USB.
- **Windows only:** the **STMicroelectronics Virtual COM Port driver** so the unit
  shows as a COM port. A copy is bundled in the **`Driver\`** folder at the repo
  root (and inside the Windows download zips) — if Windows doesn't show a COM port,
  install it by running **`Driver\dpinst_amd64.exe`** (right-click → Run as
  administrator). On Linux/macOS no driver is needed (see below).
- Internet access to `dfu.tinydevices.org` (the tool tries HTTPS first and falls
  back to plain HTTP).

No .NET installation is needed — every build is self-contained and single-file.
The firmware folder defaults to a `Firmware` folder next to the executable and is
created automatically if it doesn't exist.

---

## Quick start (Windows)

Windows SmartScreen may warn that this is an unrecognised app (it's an unsigned personal tool). Click More info → Run anyway. Verify the download with the SHA-256 in the release notes.

1. Connect the tinySA to the PC **in normal mode** (powered on as usual).
2. Double-click **`binaries\windows\x64\TinySAUpdater.exe`** (or the `arm64` build
   on an ARM PC).
3. It detects the unit, checks the version, and tells you whether an update is
   needed.
4. If an update is available, it downloads it and asks you to put the unit in
   **DFU mode** (see below), then flashes.
5. Read the summary, then press **`[Q]`** to close.

---

## Linux

Linux builds are under **`binaries/linux/x64/`** and **`binaries/linux/arm64/`**
(self-contained — no .NET install needed). Linux differs from Windows in two ways:

- **No driver needed** — the kernel's built-in `cdc-acm` driver handles the tinySA.
  The unit appears as `/dev/ttyACM0`.
- **dfu-util comes from your package manager** — the tool calls the system
  `dfu-util` (it does not bundle or download it):

  ```bash
  sudo apt install dfu-util        # Debian/Ubuntu  (or dnf/pacman)
  chmod +x TinySAUpdater
  ./TinySAUpdater
  ```

If you get a permission error reading the port, add yourself to the `dialout`
group (`sudo usermod -aG dialout $USER`, then log out/in). Flashing over DFU may
need `sudo` or a udev rule for the STM DFU device (VID `0483` PID `df11`).
Serial ports are given as `--port /dev/ttyACMx`.

---

## macOS

macOS builds are under **`binaries/macos/x64/`** (Intel) and
**`binaries/macos/arm64/`** (Apple Silicon) — self-contained, no .NET install
needed. As on Linux, no driver is required and the tool uses the **system
`dfu-util`**:

```bash
brew install dfu-util
chmod +x TinySAUpdater
xattr -d com.apple.quarantine TinySAUpdater   # clear Gatekeeper quarantine (unsigned)
./TinySAUpdater
```

The tinySA appears as `/dev/tty.usbmodem…`; pass it with `--port` if auto-detect
misses it. The binary is unsigned — if Gatekeeper still blocks it, allow it under
**System Settings → Privacy & Security**.

---

## Command-line usage

```
TinySAUpdater [folder] [options]
```

| Option | Meaning |
|--------|---------|
| `[folder]`        | Firmware folder (default: a `Firmware` folder next to the executable) |
| `-u`, `--ultra`   | Force tinySA Ultra (skip auto-detect / menu) |
| `-b`, `--basic`   | Force tinySA basic (skip auto-detect / menu) |
| `--port COMx`     | Use this serial port instead of auto-detecting |
| `--no-check`      | Skip the serial version check (flash even if the unit is already in DFU mode or has no driver) |
| `--offline`       | Skip the internet check and flash already-downloaded firmware from the `Firmware\` folder |
| `--no-update-check` | Skip the startup check for a newer version of this updater on GitHub |
| `-f`, `--force`   | Flash even if the unit already runs the latest build |
| `--no-flash`      | Download / rename only, do not run dfu-util |
| `-y`, `--yes`     | Do not prompt before flashing |
| `-v`, `--version` | Print the version and exit |
| `-h`, `--help`    | Show help |

---

## How it decides whether to update

| Situation | What you see |
|-----------|--------------|
| **Unit already current** | `✔ Your tinySA Ultra is already on the latest firmware (…). Nothing to do.` and it exits without downloading or flashing. Summary: `✔ Status  Already on the latest firmware`. |
| **Unit firmware is older** | `! Update available:  <installed>  ->  <latest>` (yellow). It downloads the new `.bin`, shows the **ENTER DFU MODE** box, and prompts `…press ENTER to flash (or Q + ENTER to skip)`. Summary: `! Status  update available: <installed> -> <latest>`. There is no separate menu — the flash prompt is the decision point. |
| **No unit detected** | It stops with `✗ No tinySA found on a USB serial (COM) port.` and driver/`--port`/`--no-check` guidance. |
| **`--no-check` used** | The serial check is skipped, so you pick the model with `--ultra`/`--basic` or the on-screen menu, and it proceeds to download + flash. |

> The model-selection menu (`[1] Ultra / [2] basic / [Q] Quit`) only appears in
> `--no-check` mode when no model flag is given. In normal operation the model is
> auto-detected from the connected unit.

---

## Entering DFU mode

DFU (Device Firmware Update) mode is needed for flashing:

1. Keep the tinySA connected to the PC via USB.
2. Power the unit **OFF**.
3. **Hold the jog button down.**
4. While holding it, switch the unit **ON**. The screen stays **blank** — it
   looks dead, but it is now in DFU mode.
5. Press **ENTER** in the tool to flash.

In DFU mode the unit changes USB identity (PID `0xDF11`) and no longer presents a
COM port — which is why the version check is done **before** you enter DFU mode.

---

## After the update

1. The unit reboots automatically.
2. Connect an **SMA-to-SMA** cable between the two ports.
3. Menu → run **SELF TEST**.
4. Menu → run **CALIBRATION** (below 6 GHz).
5. Re-enable **ULTRA** mode in the menu, password **4321** (Ultra models only).

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| "No tinySA found on a COM port" | Connect the unit in normal mode; on Windows install the bundled STM VCP driver by running `Driver\dpinst_amd64.exe` as administrator; or pass `--port COMx`. |
| Unit is already in DFU mode (blank screen) | Run with `--no-check --ultra` (or `--basic`) to flash without the version check. |
| Wrong port chosen | Pass `--port COMx` (Windows) or `--port /dev/tty…` (Linux/macOS) explicitly. |
| `dfu-util could not open the DFU device` / `Cannot open DFU device 0483:df11` | The unit IS in DFU mode but Windows has no compatible (WinUSB) driver on the "STM32 BOOTLOADER" device — Windows normally fetches ST's driver from Windows Update by itself, but corporate PCs often block that. With the unit still in DFU mode: run the bundled **`Driver\zadig-2.9.exe`** (or [zadig.akeo.ie](https://zadig.akeo.ie)) → Options → List All Devices → select **STM32 BOOTLOADER** (0483 DF11) → choose **WinUSB** → Install/Replace Driver, then re-run the updater. (Alternative: Device Manager → "STM32 BOOTLOADER" → Update driver → Search automatically.) |
| `dfu-util exited with code …` (other) | The unit was probably not in DFU mode — repeat the DFU-mode steps and try again. |
| `dfu-util` not found (Linux/macOS) | Install it: `sudo apt install dfu-util` (Linux) or `brew install dfu-util` (macOS). |
| Network error | Check connectivity/firewall to `dfu.tinydevices.org` (HTTPS first, plain-HTTP fallback — allow both). |

---

## Version history

- **1.1.6** — Fixes from a fourth code review. **No more silent downgrades from the
  server:** the online "latest" is now picked by the firmware's **build number**
  (the `224` in `…v1.4-224-g…`), with the listing date only as a tie-break, and the
  update decision compares build numbers too — a re-uploaded *older* `.bin` on the
  server, or a unit running a build *newer* than the server's, is no longer offered
  as an "update". A **pre-existing `dfu-util-static.exe` is now validated** (real
  Windows executable) and re-downloaded if corrupt, instead of crashing the flash
  step. `--port` no longer swallows a following option (e.g. `--port --yes`) as its
  value. The **flash prompt only proceeds on ENTER / Y / YES** — `N`, `No`, `Quit`
  or a typo now skips (previously anything except `Q` flashed). Downloads use a
  **stall timeout** instead of a 120-second total cap, so a slow-but-moving link is
  no longer cut off mid-transfer, and a stalled transfer is reported as a network
  error (not "Unexpected error"); empty (0-byte) downloads are rejected. Help text:
  corrected the default firmware-folder description.
- **1.1.5** — Reliability fixes from a code review: **port auto-detect now probes
  every candidate** until one answers as a tinySA (previously only the first port was
  tried, so a radio/GPS on macOS or a stale COM mapping on Windows could hide the
  unit); **offline mode picks the newest firmware by build number**, not text order
  (a `…-89` file no longer beats `…-224`); a wedged `dfu-util` is terminated after
  3 minutes instead of hanging; EOF (Ctrl+Z/Ctrl+D) at the flash prompt now cancels
  instead of flashing; a bad `[folder]` argument no longer crashes silently; `--port`
  with no value is reported; and the self-update check no longer nags on a
  non-numeric release tag.
- **1.1.4** — Fix: the dfu-util output capture added in 1.1.2 (for the driver
  diagnostics) was not thread-safe — stdout and stderr arrive on different
  threads and could, very rarely, garble the captured text and miss the
  driver-fix detection. Appends are now locked.
- **1.1.3** — macOS: the tinySA is now auto-detected (`/dev/cu.usbmodem*` — previously
  only Linux port names were scanned, so macOS always needed `--port`). Security:
  server downloads (firmware and `dfu-util-static.exe`) now try **HTTPS first** with
  plain-HTTP fallback, and the downloaded `dfu-util` is validated as a real Windows
  executable before it is ever run. Safety: when input is non-interactive (piped /
  scripted), the flash step no longer proceeds without `--yes`.
- **1.1.2** — Better flash-failure diagnostics: when dfu-util finds the unit in DFU
  mode (`0483:df11`) but cannot **open** it — a missing WinUSB driver on Windows,
  permissions on Linux/macOS — the tool now says so and shows the exact fix,
  instead of wrongly claiming the unit is not in DFU mode. **Zadig 2.9** is now
  bundled in the `Driver\` folder for offline driver installs (thanks to a field
  report from a corporate PC that blocks Windows Update drivers). Distribution:
  self-contained binaries for **Windows, Linux and macOS** (x64 + ARM64) under
  `binaries/<os>/<arch>/`.
- **1.1.1** — Fixes: (1) flashing failed with "Could not find file `tinySA.bin`"
  when the versioned firmware was already downloaded — the generic flash file
  (`tinySA.bin` / `tinySA4.bin`) is now always (re)created before flashing;
  (2) the `--no-check` model menu no longer loops forever when input is
  non-interactive — it exits asking for `--ultra`/`--basic`; (3) downloads now
  write to a temporary `.part` file and are size-checked, so an interrupted
  download can't leave a truncated `.bin` that a later run treats as complete.
  New: on startup (in the `[1/5]` step) the app checks the project's GitHub
  Releases and reports whether this is the latest version, or shows an "update
  available" notice with the download link when a newer release exists
  (`--no-update-check` to skip). Distribution: prebuilt self-contained binaries
  for **Windows, Linux and macOS** (x64 + ARM64) are provided under
  `binaries/<os>/<arch>/`.
- **1.1.0** — Internet connectivity preflight; offline flash of already-downloaded
  firmware; serial version check + model auto-detection; version-string comparison
  (not dates); downloads only when newer; end-of-run summary; window stays open
  with a `[Q]` quit prompt; `by ZS6ORB` banner and `--version` flag;
  cross-platform Windows + Linux build.
- **1.0.0** — Initial: download latest `.bin`, fetch `dfu-util`, rename, flash;
  model menu.

---

## Credits

- **ZS6ORB** — author of this updater.
- Firmware, `dfu-util-static.exe`, and `tinySA.py` are by the tinySA project
  (Erik Kaashoek) — <https://tinysa.org>.
- **Zadig** (bundled driver installer) is by Pete Batard, GPLv3 —
  <https://zadig.akeo.ie>.

---

This is a personal tool by **Wayne Bevan (ZS6ORB)**, shared as-is. Comments,
suggestions or feature ideas? Drop me a mail: <wayne.bevan@yahoo.co.uk> — 73!
