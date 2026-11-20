---
name: android-tv-makeover
description: End-to-end makeover of an Android TV or Google TV over Wi-Fi - find it on the network, back it up, remove bloat and ads, measure why it is slow, replace the home screen with a fast Apple-TV-style launcher, brand it (name, boot splash, wallpaper, fonts, sounds) and close up safely. Use this whenever someone says their smart TV is slow, laggy or full of ads, wants to debloat or "clean" a TV, wants it to look like an Apple TV, mentions cleanyourtv, ADB on a TV, a TV model number (TCL, Hitachi, Xiaomi, Hisense, Philips, Sony, Onn, Chromecast...), or wants a custom/branded TV home screen - even if they only ask to "plan" it. It coordinates the android-tv-debloat, android-tv-launcher, android-tv-launcher-design and android-tv-ui-sounds skills.
---

# Android TV makeover

Paths: `<skill>` is the folder that contains this SKILL.md; `<repo>` is the folder two levels above it (the one
holding `templates/`, `tvs/` and `scripts/`). Run commands from `<repo>` or use absolute paths.

Turns a slow, ad-filled Android TV into a clean, fast, branded one from a computer on the same Wi-Fi, using
[cleanyourtv](https://github.com/Gathin23/cleanyourtv) (MIT) for the ADB tools, debloat lists and launcher.
Nothing is flashed or rooted and nothing is uninstalled: every change is a `pm disable-user` or an app install,
so one script puts the TV back.

## What works and what does not

- Works: Android TV and Google TV sets and boxes (TCL, Hitachi/Philips/Sanyo built by TCL or MTC, Xiaomi,
  Hisense Android TV models, Sony, Onn, Chromecast with Google TV, Nvidia Shield...).
- Does not work: Samsung Tizen, LG webOS, Roku, Fire TV OS 8+ (Vega). Say so plainly and stop.
- Cannot change without root (do not offer to root): the manufacturer's boot logo and the Android boot
  animation (they live on read-only, verified partitions). The launcher can show its own splash right after.

## The phases and which skill does each

| Phase | Skill | Result |
|---|---|---|
| 1. Find, connect, back up, diagnose | `android-tv-debloat` | Snapshot, baseline numbers, the real cause of the slowness |
| 2. Debloat in batches | `android-tv-debloat` | Telemetry, ads and unused apps disabled, verified after each batch |
| 3. New home screen | `android-tv-launcher` | cleanyourtv launcher installed as Home, dock and inputs configured |
| 4. Custom look (optional) | `android-tv-launcher-design` | A template (`templates/`: DreamOS, starter) or a new design from Figma, verified to the pixel |
| 5. Sounds (optional) | `android-tv-ui-sounds` | Startup, focus-move and select sounds |
| 6. Close up | this skill | Network ADB off, before/after report, restore instructions |

Read the next skill when you reach its phase; each one is self-contained.

## Before touching anything

1. Clone cleanyourtv next to the work (`git clone https://github.com/Gathin23/cleanyourtv`), create its
   venv (`python3 -m venv tools/venv && tools/venv/bin/pip install -r tools/requirements.txt`) and look for
   the user's model among the 125 profiles in `<repo>/tvs` with
   `<repo>/scripts/find-tv-profile.sh` (also read lists for the same chip/firmware), and run
   `<repo>/scripts/doctor.sh` to check the computer has what the launcher build needs.
2. Identify the platform: search the model number, and read what the TV announces on the network
   (`android-tv-debloat/scripts/find_tv.sh`): a Cast name like `BeyondTV-...` means TCL's Android TV, which
   matters more than the brand on the box.
3. Plan with the user in their language, in plain words, and set honest expectations: debloating alone often
   gives no measurable speed gain. The usual real wins are a lighter launcher (more free RAM), switching off
   a voice assistant that burns CPU at idle, fixing Wi-Fi, and keeping heavy apps fully compiled.
4. Ask the user to enable Developer options and USB debugging (Settings > Device Preferences > About >
   press OK on Build 7 times > back > Developer options), and to tick "Always allow from this computer" on
   the authorisation prompt. A plain "Allow" is one-time: the next connection waits for the prompt again.

## Ground rules (why they matter)

- Only reversible changes, in small batches, with a screenshot and a check after each. A batch can take
  something load-bearing with it, and with ten packages at once you cannot tell which.
- Ask before anything that depends on taste or habits: which apps they use, the dock, voice assistant,
  casting, USB media, screensaver, firmware updates. Use checklist questions (a multiple-choice
  question tool if your agent has one) with recommendations, and offer to skip.
- Confirm before reboots and say what the user will see on the TV; tell them when a prompt will appear.
- Report what you measured, not what you hoped. If the numbers are noisy or the gain is small, say so.
- Never paste the user's IP, Wi-Fi name, account names or profile pictures into anything shared (an
  example list, a PR, a skill).

## Close up (phase 6)

1. Show a before/after table: idle free memory, swap used, idle CPU, the slow thing they complained about.
2. Give them the restore command (`TV_HOST=<ip> tools/restore.sh <list>`) and say where the backup is.
3. Offer to contribute their debloat list to cleanyourtv's `examples/` (it helps the next owner of that model).
4. Close network ADB: on firmware with `ro.adb.secure=0` anyone on the LAN gets a shell, and port 5555 is
   plaintext either way. Ask them to switch off USB debugging and Network/ADB debugging, then check from the
   computer that `nc -vz <tv ip> 5555` fails after a reboot. Only skip this if they say they will keep
   working on the TV soon, and remind them.
