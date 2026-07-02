# Binaries

Pre-built, self-contained tinySA Firmware Updater executables (v1.1.2). Each binary
bundles the .NET runtime, so **no .NET installation is required** — download the one
for your machine and run it.

| OS | Architecture | File |
|----|--------------|------|
| Windows | x64 (Intel/AMD) | [`windows/x64/TinySAUpdater.exe`](windows/x64/TinySAUpdater.exe) |
| Windows | ARM64 | [`windows/arm64/TinySAUpdater.exe`](windows/arm64/TinySAUpdater.exe) |
| Linux | x64 | [`linux/x64/TinySAUpdater`](linux/x64/TinySAUpdater) |
| Linux | ARM64 (e.g. Raspberry Pi 64-bit) | [`linux/arm64/TinySAUpdater`](linux/arm64/TinySAUpdater) |
| macOS | x64 (Intel) | [`macos/x64/TinySAUpdater`](macos/x64/TinySAUpdater) |
| macOS | ARM64 (Apple Silicon) | [`macos/arm64/TinySAUpdater`](macos/arm64/TinySAUpdater) |

## Running

**Windows** — double-click `TinySAUpdater.exe` or run it from a terminal. The
Windows driver is in the `Driver\` folder at the repo root; `dfu-util` is bundled
and fetched automatically when needed.

**Linux / macOS** — the updater uses the *system* `dfu-util` (it does not bundle it):

```sh
# Linux:  sudo apt install dfu-util   (or dnf / pacman)
# macOS:  brew install dfu-util
chmod +x TinySAUpdater      # if the executable bit was lost on download
./TinySAUpdater
```

On macOS the binary is unsigned; if Gatekeeper blocks it, clear the quarantine
attribute with `xattr -d com.apple.quarantine TinySAUpdater` (or allow it under
System Settings → Privacy & Security).

See the top-level `README.md` / `readme.txt` for full usage, flags, and the DFU
flashing walkthrough.
