# H96 MAX V58 — Modded Stock Firmware

Modified factory firmware for the **H96 MAX V58** (Rockchip RK3588, Android 12 / Android TV).

Built on top of the stock OEM image. The vendor launcher is replaced with the real
**Android TV** launcher, the Google components the stock build was missing are filled
in, and a few long-standing bugs in that build are fixed. Everything else — kernel,
bootloader, recovery, partition layout — is untouched stock.

> Flashing replaces the `super`, `boot` and bootloader partitions listed below.
> Your `/data` (apps and settings) is normally preserved, but back up anything
> important first. Full stock restore instructions are at the bottom.

---

## What's changed

### 1. MindTheGapps injected into the system image

The stock ROM already ships a full Google-certified GApps set (GSF in `system_ext`;
GMS Core, Play Store, sync adapters, WebView and YouTube in `product`). Copying
MindTheGapps on top would have produced duplicate packages and a broken boot. So only
the components the stock image was actually missing were added:

| Added | Package |
|---|---|
| AndroidMediaShell | `com.google.android.apps.mediashell` |
| TVLauncher (Google TV launcher) | `com.google.android.tvlauncher` |
| TVRecommendations | `com.google.android.tvrecommendations` |
| AtvAxel | `com.google.android.tv.axel` |
| AtvRemoteService | `com.google.android.tv.remote.service` |
| SssAuthbridgePrebuilt | `com.google.android.sss.authbridge` |
| Backdrop | `com.google.android.backdrop` |
| talkback | `com.google.android.marvin.talkback` |
| Katniss (upgraded from the stock 2.2) | `com.google.android.katniss` 6.4 |
| ATVOverlay (RRO) + permission XMLs | |

**GMS Core was also replaced**: stock `GmsCore` 21.48.15 → MindTheGapps
`PrebuiltGmsCorePano` 22.15.14. This is not optional — the Google TV launcher
declares `google_play_services_version 220602000`, which 21.48.15 cannot satisfy.

### 2. Android TV launcher replaces the vendor launcher

The stock `com.droidlogic.mboxlauncher` was removed so the Android TV launcher becomes
the only HOME app.

### 3. Apps drawer crash fixed

On stock, tapping **Apps** in the launcher crashed straight back to the home screen:

```
SecurityException: Permission Denial: android.intent.action.CLOSE_SYSTEM_DIALOGS
  broadcast from com.google.android.tvlauncher requires
  android.permission.BROADCAST_CLOSE_SYSTEM_DIALOGS
```

The launcher's manifest never requested that permission, so whitelisting alone did
nothing. Fixed by patching the permission into the APK's manifest and adding it to the
priv-app whitelist.

### 4. Settings gear opens the TV settings app

The launcher's top-right setting was changed to "com.android.tv.settings".

### 5. Replace boot animation with atv11_bootanimation

https://github.com/khurramrizvi/atv11_bootanimation

> the **first boot logo** (the H96 splash shown before Android starts).
> It lives inside `boot.img`'s Rockchip resource section and is protected by a
> U-Boot image hash that we could not reproduce, so it is left as stock.

---

## Download & flash

Two equivalent packages — pick whichever matches your tooling.

### Option A — RKDevTool (Windows, one click)

`rkdevtool/` — **`update.img`**, split into parts because GitHub caps release assets
at 2 GiB.

1. Rejoin the parts:

   ```powershell
   # Windows
   cmd /c copy /b update.img.001+update.img.002 update.img
   ```
   ```bash
   # Linux / macOS
   cat update.img.001 update.img.002 > update.img
   ```

2. Open **RKDevTool**, go to the **Upgrade Firmware** tab
3. Click **Firmware** and select `update.img`
4. Put the box into **Maskrom mode**: with the box unplugged, hold the recessed reset
   button (rear panel, between the two WiFi antenna posts), connect the USB-A cable,
   then power, and release after ~2 seconds. RKDevTool should report *Found One MASKROM Device*.
5. Click **Upgrade** and wait (~3 minutes). The box reboots by itself.

### Option B — rkdeveloptool (Linux / macOS / WSL)

Relate to original tutorial.

---

## Credits

1. https://github.com/AndroidOS-PRO/MindTheGapps_Android_TV
2. https://github.com/khurramrizvi/atv11_bootanimation
3. https://github.com/Scrumpper/H96-MAX-V58-rk3588-Custom-Firmware-Suite



------------------------------------------------------------------------------------
# Scrumpper's H96 Max V58 Firmware Suite

One-stop installer for custom firmware on **H96 Max V58** (Rockchip RK3588) TV box.
Run one script, pick build, flash it.

```
   ██╗  ██╗ █████╗  ██████╗    ██╗   ██╗███████╗ █████╗
   ██║  ██║██╔══██╗██╔════╝    ██║   ██║██╔════╝██╔══██╗
   ███████║╚██████║███████╗    ██║   ██║███████╗╚█████╔╝
   ██╔══██║ ╚═══██║██╔═══██╗    ╚██╗ ██╔╝╚════██║██╔══██╗
   ██║  ██║ █████╔╝╚██████╔╝     ╚████╔╝ ███████║╚█████╔╝
   ╚═╝  ╚═╝ ╚════╝  ╚═════╝       ╚═══╝  ╚══════╝ ╚════╝
```

## Use it

```bash
python3 install.py
```

Pick build from menu → confirm → it flashes.

## Layout

REQUIRED: Extract all .zip files into one folder as listed below.

Each build lives in **its own sub-folder** next to `install.py`, named after build.
Menu auto-detects whatever folders are present, so you can add or remove builds freely:

```
Scrumpper's H96 Max V58 Firmware Suite/
├── install.py                              ← run this
├── WHICH BUILD SHOULD I PICK.md
├── README.md
├── H96 MAX V58 Stripped Stock Android/     ← Android 12, Google/Play (recommended)
├── LineageOS 23 (Android 16)/              ← newest Android
└── Unofficial Armbian v6.2/                ← Linux, console base, desktop on demand
```

Each build folder is **self-contained**: it holds its own image(s) + `flash.py` (plus
`MiniLoaderAll.bin` and any partition files it needs). Menu runs that folder's flasher.
Folder names are matched loosely (case/punctuation don't matter); any folder with `flash*.py`
or image will still be offered.

## Requirements

- **Python 3.6+**
- **rkdeveloptool** on your PATH (flashers talk to box over USB)
  - Arch: `yay -S rkdeveloptool` · Debian/Ubuntu: build from `github.com/rockchip-linux/rkdeveloptool`
- **USB-A ↔ USB-A** cable to box, and box in **Maskrom** mode (flashers walk you
  through it. Power off, hold pinhole reset button (rear panel, in gap between two WiFi antenna posts), plug USB, connect power, release).
- Linux or macOS. (Windows: use flashers from WSL, or Rockchip's RKDevTool GUI.)

## ⚠️ Flashing wipes box

Every build fully erases and replaces what's on box. That's expected. All of them are
recoverable. RK3588 Maskrom lives in ROM, so paperclip on reset button always gets
you back to flashable state.

---


