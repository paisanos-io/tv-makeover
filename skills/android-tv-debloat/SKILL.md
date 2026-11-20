---
name: android-tv-debloat
description: Connect to an Android TV / Google TV over Wi-Fi ADB, back it up, diagnose why it is slow (memory, idle CPU, Wi-Fi band, GPU), and disable telemetry, ads and unused apps in small verified batches with cleanyourtv's reversible scripts. Use it whenever a user wants to clean up, speed up or remove ads/bloatware from a TV, asks why their TV or its YouTube/Netflix is slow, needs to connect to a TV with ADB (including "No route to host" on macOS or a stuck authorisation prompt), or wants to restore what was disabled.
---

# Android TV: connect, diagnose, debloat

Paths: `<skill>` is the folder that contains this SKILL.md; `<repo>` is the folder two levels above it (the one
holding `templates/`, `tvs/` and `scripts/`). Run commands from `<repo>` or use absolute paths.

Scripts live in `<skill>/scripts`. They take `TV_HOST=<tv ip>` and `CYT=<path to the cleanyourtv
clone>` (default: the current directory) and use cleanyourtv's `tools/tv.py`, a pure-Python ADB client.
Firmware quirks and lessons from real units are in `references/quirks.md`; read it when something behaves
oddly. Profiles for 125 tested TVs and boxes are in `<repo>/tvs` (`INDEX.md`); find the closest ones to
this TV with `<repo>/scripts/find-tv-profile.sh <brand model chip android>` and read their headers
before deciding anything.

## 1. Find and connect

- `scripts/find_tv.sh` lists mDNS names (`_androidtvremote2`, `_googlecast`, `_adb-tls-connect`), the ARP
  table and which hosts answer on 5555. Match the mDNS `bt=` address to an ARP MAC (usually the last byte
  differs by one) to pin the TV's IP.
- Port 5555 closed: USB debugging is off, or the TV only offers Wireless debugging (Google TV): pair with the
  code shown under Developer options > Wireless debugging (`adb pair`, `adb connect`, `adb tcpip 5555`).
- Use `tools/tv.py`, not the `adb` binary: on macOS, Local Network privacy makes `adb connect` fail with
  "No route to host" even though the port is open.
- `tv.py --probe` hanging for 120 s and then a timeout in `connect()` means the TV is showing the "Allow USB
  debugging?" prompt. Tell the user to look at the TV and tick "Always allow", then retry once.
- If the TV vanishes (ping fails, "Host is down") but the user says it is on: it probably rejoined a different
  SSID after a reboot. Routers often isolate their 2.4 GHz and 5 GHz SSIDs from each other. Ask which network
  the TV is on (Settings > Network) and compare with the computer's (`ipconfig getsummary en0` on macOS).

## 2. Back up and diagnose (read-only)

Run `scripts/snapshot.sh <backup_dir>` (put it in cleanyourtv's git-ignored `backup/<tv>/`). It saves
getprop, the package lists, all settings, Home and its claimants, TV inputs, Wi-Fi owners, and prints a summary:
model, board, Android, firmware, GPU, display, memory, storage, Wi-Fi band/RSSI/link, and the top CPU users.

Then find the actual cause of "slow" before promising anything:

- Baseline after a reboot, the same way you will measure at the end: `REBOOT=1 scripts/metrics.sh before <dir>`
  (waits 150 s after boot). Compare MemAvailable, swap used and idle CPU, not a snapshot taken mid-use.
- A process eating CPU at idle (often `com.google.android.katniss:interactor`, the voice assistant, around a
  third of a core on TCL Realtek sets) is a bigger win than any debloat.
- Video that stutters: have the user open YouTube > settings > "Stats for nerds" while playing and take a
  screenshot (`scripts/shot.sh`): connection speed, buffer health and dropped frames tell network from decoder.
  Also read the Wi-Fi line of the snapshot: many TVs are 2.4 GHz-only even on an SSID named "5GHz".
- Menus that lag inside apps on a 1.1 GHz quad Cortex-A55 are mostly the CPU. Compiling the user's main apps
  fully helps a little: `cmd package compile -m speed -f <pkg>` (one per call; a big app can take longer
  than tv.py's 120 s read timeout, so check `dumpsys package dexopt` for `status=speed` instead of retrying).
- Check `/data` free space: under ~10% free, installs fail and the TV crawls.

## 3. Ask, then debloat in batches

Sort the packages (snapshot.txt) into telemetry, ads/content, background extras, preloaded apps and streaming
apps. For anything ambiguous, read `dumpsys package <pkg>` (activities, services, receivers) before deciding;
vendor names rarely say what a package does. Lean on the closest `examples/` lists for the same chip.

Ask with checklist questions (multiple choice, with a recommendation) before touching preferences: apps they actually use (group them), the four dock
apps in order, voice assistant (recommend off if it burns CPU), Chromecast, USB media player, screensaver,
firmware updates (default keep), ad-blocking Private DNS (optional, does not block YouTube ads).

Never disable, whatever the name suggests: the Live TV app and TV input services (a disabled input driver
leaves HDMI showing "No Signal" until a full restore and cold boot), the source/input menu, remote control and
Bluetooth pairing services, the keyboard, Play Store and Play Services, Settings, and
`com.google.android.tungsten.setupwraith` when `dumpsys wifi` shows it owns the saved Wi-Fi (disabling it
deletes that network and drops the TV off Wi-Fi). `tools/disable.sh` guards several of these.

Apply with one growing list file, `examples/<your-model>/debloat.txt` in the cleanyourtv clone, starting with a
header comment (model, board, Android, firmware, fingerprint, what you kept and why). When done, copy it to
`<repo>/tvs/<model>/` with a `profile.json` (see tvs/README.md) so templates can use it:

1. Add a batch (telemetry first, then ads/content, then unused apps, then voice, then extras).
2. `TV_HOST=<ip> tools/disable.sh examples/<model>/debloat.txt`. Re-running the whole list is fine: the
   recovery record keeps the first state and adds new packages to it.
3. Verify: press Home, `scripts/shot.sh`, open the main streaming apps (a black/0-byte screenshot over Netflix
   is the secure layer; check `mResumedActivity` instead), and for voice/remote batches ask the user to press
   the real buttons.
4. Leave packages that carry an installer (TCL `com.tcl.guard`, `appmarket2`) until after the launcher install.

Undo anything: `TV_HOST=<ip> tools/restore.sh examples/<model>/debloat.txt`.

## 4. Small, safe speed-ups

- Animation scales to 0.5 (`settings put global window_animation_scale 0.5`, same for
  `transition_animation_scale` and `animator_duration_scale`); restore.sh resets them.
- Full compile of the dock apps (above).
- Screensaver: replace a heavy photo screensaver (Backdrop, ~120 MB) with the launcher's clock
  (`settings put secure screensaver_components com.gathin.tvhome/.ClockDream`) once the launcher is installed.

Measure again exactly like the baseline (`REBOOT=1 scripts/metrics.sh after <dir>`) and report both.

## 5. Extreme performance mode (very old or weak TVs)

For sets with 1 GB of RAM or less, Mali-400/450-class GPUs or Android 7-9 that still lag after the steps above,
offer this, explain each part, and apply only what the owner agrees to:

1. The `extreme` launcher template: cleanyourtv's layout with no animations, glow, wallpaper, splash or sounds
   (`<repo>/scripts/apply-template.sh extreme --tv <profile> --dock ... --install`).
2. `<repo>/scripts/extreme-performance.sh apply <pkg,pkg,...>`: system animation scales to 0, the named apps
   barred from running in the background (they still open normally) and moved to the "rare" standby bucket,
   caches trimmed. Name heavy streaming apps the household only uses in the foreground; never music players,
   casting, the launcher, keyboards or TV/input services. Run `status` first and `--dry-run` to show the plan.
   It records every value it changes; `undo` restores them exactly (keep the state file it writes).
3. Voice assistant off and the photo screensaver replaced by the clock (above), if not done already.
4. Measure with `metrics.sh` before and after, and tell the owner what each part bought.

