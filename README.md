# tinySA Firmware Updater

**by ZS6ORB · v1.1.1**

A console tool (**Windows and Linux**) that updates the firmware on a **tinySA**
or **tinySA Ultra** spectrum analyser. It talks to the unit over USB, checks
whether a newer firmware build is available, downloads it, and flashes it over
USB DFU — with a clear on-screen summary at every step.

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
              v1.1.1    for tinySA  &  tinySA Ultra
```

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
6. **Fetches `dfu-util-static.exe`** automatically if it isn't in the firmware folder.
7. **Renames** the download to `tinySA4.bin` / `tinySA.bin` for flashing, while
   keeping the original versioned `.bin` as an on-disk record.
8. **Flashes** with `dfu-util-static.exe -a 0 -s 0x08000000:leave -D <bin>`,
   streaming the live output.
9. **Prints a summary** of every check and **stays open** until you press `[Q]`.

---

## Requirements

- Windows 10/11 (64-bit).
- The tinySA connected by USB.
- The **STMicroelectronics Virtual COM Port driver** so the unit shows as a COM
  port. A copy is bundled in the **`Driver\`** folder next to the .exe — if
  Windows doesn't show a COM port, install it yourself by running
  **`Driver\dpinst_amd64.exe`** (right-click → Run as administrator).
- Internet access to `http://dfu.tinydevices.org` (HTTP only — the server does
  not use HTTPS).

No .NET installation is needed — the build is self-contained and single-file.
The firmware folder defaults to a `Firmware` folder next to the executable and is
created automatically if it doesn't exist. The driver above is **Windows-only**;
on Linux see the [Linux](#linux) section.

---

## Quick start

1. Connect the tinySA to the PC **in normal mode** (powered on as usual).
2. Double-click **`TinySAUpdater.exe`**.
3. It detects the unit, checks the version, and tells you whether an update is
   needed.
4. If an update is available, it downloads it and asks you to put the unit in
   **DFU mode** (see below), then flashes.
5. Read the summary, then press **`[Q]`** to close.

---

## Linux

A Linux build is provided as **`Linux/TinySAUpdater`** (a self-contained `linux-x64`
binary — no .NET install needed). Linux differs from Windows in two ways:

- **No driver needed** — the kernel's built-in `cdc-acm` driver handles the tinySA.
  The unit appears as `/dev/ttyACM0`.
- **dfu-util comes from your package manager** — the tool calls the system
  `dfu-util` (it does not bundle or download it):

  ```bash
  sudo apt install dfu-util        # Debian/Ubuntu  (or dnf/pacman/brew)
  chmod +x TinySAUpdater
  ./TinySAUpdater
  ```

If you get a permission error reading the port, add yourself to the `dialout`
group (`sudo usermod -aG dialout $USER`, then log out/in). Flashing over DFU may
need `sudo` or a udev rule for the STM DFU device (VID `0483` PID `df11`).
Serial ports are given as `--port /dev/ttyACMx`.

---

## Command-line usage

```
TinySAUpdater [folder] [options]
```

| Option | Meaning |
|--------|---------|
| `[folder]`        | Firmware folder (default: a `Firmware` folder next to the .exe) |
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
| "No tinySA found on a COM port" | Connect the unit in normal mode; install the bundled STM VCP driver by running `Driver\dpinst_amd64.exe` as administrator; or pass `--port COMx`. |
| Unit is already in DFU mode (blank screen) | Run with `--no-check --ultra` (or `--basic`) to flash without the version check. |
| Wrong COM port chosen | Pass `--port COMx` explicitly. |
| `dfu-util exited with code …` | The unit was probably not in DFU mode — repeat the DFU-mode steps and try again. |
| Network error | The server is **HTTP only**; check connectivity/firewall to `http://dfu.tinydevices.org`. |

---

## Version history

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
  (`--no-update-check` to skip).
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

---

This is a personal tool by **Wayne Bevan (ZS6ORB)**, shared as-is. Comments,
suggestions or feature ideas? Drop me a mail: <wayne.bevan@yahoo.co.uk> — 73!
