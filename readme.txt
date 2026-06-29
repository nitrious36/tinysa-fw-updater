============================================================
  tinySA FIRMWARE UPDATER
  by ZS6ORB   -   v1.1.1
  for tinySA  &  tinySA Ultra
============================================================

A console tool (Windows and Linux) that updates the firmware on a
tinySA or tinySA Ultra spectrum analyser. It talks to the unit
over USB, checks whether a newer firmware build is available,
downloads it, and flashes it over USB DFU - with a clear on-screen
summary at every step.


WHAT IT DOES
------------------------------------------------------------
 1. Checks internet connectivity and that the firmware server is
    reachable, before doing anything else. If it is offline (or
    you pass --offline), it falls back to flashing already-
    downloaded firmware from the Firmware folder.
 2. Detects the unit over its USB serial (COM) port by USB
    VID/PID (0483/5740) and reads the running firmware with the
    "version" command.
 3. Auto-detects the model from that version string:
       tinySA4_...  ->  tinySA Ultra
       tinySA_...   ->  tinySA (basic)
 4. Compares VERSION STRINGS (not file dates): the installed
    build vs. the latest .bin on the official server.
 5. Downloads only when newer. If the unit is already current it
    stops and says so (use --force to re-flash anyway).
 6. Fetches dfu-util-static.exe automatically if it is missing
    from the firmware folder.
 7. Renames the download to tinySA4.bin / tinySA.bin for
    flashing, and keeps the original versioned .bin as a record.
 8. Flashes with:
       dfu-util-static.exe -a 0 -s 0x08000000:leave -D <bin>
 9. Prints a summary of every check and stays open until you
    press [Q].


REQUIREMENTS
------------------------------------------------------------
 - Windows 10/11 (64-bit).
 - The tinySA connected by USB.
 - The STMicroelectronics Virtual COM Port driver so the unit
   shows as a COM port. A copy is bundled in the "Driver" folder
   next to the .exe. If Windows shows no COM port, install it
   yourself by running (Run as administrator):
       Driver\dpinst_amd64.exe
 - Internet access to http://dfu.tinydevices.org
   (HTTP only - the server does not use HTTPS).

 No .NET installation is needed - the build is self-contained and
 single-file. The firmware folder defaults to a "Firmware" folder
 next to the executable and is created automatically if it does
 not exist. The driver above is WINDOWS-ONLY; on Linux see the
 LINUX section below.


QUICK START
------------------------------------------------------------
 1. Connect the tinySA to the PC in NORMAL mode (powered on as
    usual).
 2. Double-click TinySAUpdater.exe
 3. It detects the unit, checks the version, and tells you
    whether an update is needed.
 4. If an update is available, it downloads it and asks you to
    put the unit in DFU mode (see below), then flashes.
 5. Read the summary, then press [Q] to close.


LINUX
------------------------------------------------------------
 A Linux build is provided as  Linux/TinySAUpdater  (a self-
 contained linux-x64 binary - no .NET install needed). Two
 differences from Windows:

  - No driver needed. The kernel's built-in cdc-acm driver
    handles the tinySA; it appears as /dev/ttyACM0.
  - dfu-util comes from your package manager (the tool calls the
    system dfu-util; it does not bundle or download it):

        sudo apt install dfu-util      # or dnf / pacman / brew
        chmod +x TinySAUpdater
        ./TinySAUpdater

 If you get a permission error on the port, add yourself to the
 dialout group:  sudo usermod -aG dialout $USER  (then re-login).
 Flashing over DFU may need sudo or a udev rule for the STM DFU
 device (VID 0483 PID df11). Serial ports are given as
 --port /dev/ttyACMx.


COMMAND-LINE USAGE
------------------------------------------------------------
   TinySAUpdater [folder] [options]

   [folder]        Firmware folder (default: a "Firmware" folder
                   next to the .exe)
   -u, --ultra     Force tinySA Ultra (skip auto-detect / menu)
   -b, --basic     Force tinySA basic (skip auto-detect / menu)
       --port COMx Use this serial port instead of auto-detecting
       --no-check  Skip the serial version check (flash even if
                   the unit is already in DFU mode or has no
                   driver)
       --offline   Skip the internet check and flash already-
                   downloaded firmware from the Firmware folder
       --no-update-check
                   Skip the startup check for a newer version of
                   this updater on GitHub
   -f, --force     Flash even if the unit already runs the latest
                   build
       --no-flash  Download / rename only, do not run dfu-util
   -y, --yes       Do not prompt before flashing
   -v, --version   Print the version and exit
   -h, --help      Show help


HOW IT DECIDES WHETHER TO UPDATE
------------------------------------------------------------
 Unit already current:
   "Your tinySA Ultra is already on the latest firmware (...)."
   It exits without downloading or flashing.
   Summary: Status = Already on the latest firmware.

 Unit firmware is OLDER:
   "Update available:  <installed>  ->  <latest>" (yellow).
   It downloads the new .bin, shows the ENTER DFU MODE box, and
   prompts "press ENTER to flash (or Q + ENTER to skip)".
   Summary: Status = update available: <installed> -> <latest>.
   There is NO separate menu - the flash prompt is the decision
   point.

 No unit detected:
   "No tinySA found on a USB serial (COM) port." plus driver /
   --port / --no-check guidance.

 --no-check used:
   The serial check is skipped, so you pick the model with
   --ultra / --basic or the on-screen menu, then it downloads
   and flashes.

 Note: the model-selection menu ([1] Ultra / [2] basic / [Q]
 Quit) only appears in --no-check mode when no model flag is
 given. In normal operation the model is auto-detected from the
 connected unit.


ENTERING DFU MODE
------------------------------------------------------------
 1. Keep the tinySA connected to the PC via USB.
 2. Power the unit OFF.
 3. HOLD the jog button DOWN.
 4. While holding it, switch the unit ON. The screen stays BLANK
    - it looks dead, but it is now in DFU mode.
 5. Press ENTER in the tool to flash.

 In DFU mode the unit changes USB identity (PID 0xDF11) and no
 longer presents a COM port - which is why the version check is
 done BEFORE you enter DFU mode.


AFTER THE UPDATE
------------------------------------------------------------
 1. The unit reboots automatically.
 2. Connect an SMA-to-SMA cable between the two ports.
 3. Menu -> run SELF TEST.
 4. Menu -> run CALIBRATION (below 6 GHz).
 5. Re-enable ULTRA mode in the menu, password 4321 (Ultra
    models only).


TROUBLESHOOTING
------------------------------------------------------------
 "No tinySA found on a COM port"
     Connect the unit in normal mode; install the bundled STM VCP
     driver by running Driver\dpinst_amd64.exe as administrator;
     or pass --port COMx.

 Unit is already in DFU mode (blank screen)
     Run with --no-check --ultra (or --basic) to flash without
     the version check.

 Wrong COM port chosen
     Pass --port COMx explicitly.

 "dfu-util exited with code ..."
     The unit was probably not in DFU mode - repeat the DFU-mode
     steps and try again.

 Network error
     The server is HTTP only; check connectivity / firewall to
     http://dfu.tinydevices.org


VERSION HISTORY
------------------------------------------------------------
 1.1.1  Fixes: (1) flashing failed with "Could not find file
        tinySA.bin" when the versioned firmware was already
        downloaded - the generic flash file (tinySA.bin /
        tinySA4.bin) is now always (re)created before flashing;
        (2) the --no-check model menu no longer loops forever when
        input is non-interactive - it exits asking for --ultra /
        --basic; (3) downloads write to a temporary .part file and
        are size-checked, so an interrupted download can't leave a
        truncated .bin that a later run treats as complete.
        New: on startup (in the [1/5] step) the app checks the
        project's GitHub Releases and reports whether this is the
        latest version, or shows an "update available" notice with
        the download link when a newer release exists
        (--no-update-check to skip).
 1.1.0  Internet connectivity preflight; offline flash of
        already-downloaded firmware; serial version check + model
        auto-detection; version-string comparison (not dates);
        downloads only when newer; end-of-run summary; window
        stays open with a [Q] quit prompt; "by ZS6ORB" banner and
        --version flag; cross-platform Windows + Linux build.
 1.0.0  Initial: download latest .bin, fetch dfu-util, rename,
        flash; model menu.


CREDITS
------------------------------------------------------------
 ZS6ORB - author of this updater.
 Firmware, dfu-util-static.exe and tinySA.py are by the tinySA
 project (Erik Kaashoek) - https://tinysa.org

 This is a personal tool by Wayne Bevan (ZS6ORB), shared as-is.
 Comments, suggestions or feature ideas? Drop me a mail:
 wayne.bevan@yahoo.co.uk   73!
============================================================
