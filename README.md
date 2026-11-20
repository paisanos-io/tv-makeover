# tv-makeover

Turn a slow, ad-filled Android TV or Google TV into a clean, fast, branded one, from a computer on the same
Wi-Fi, with an AI coding agent doing the work: Claude Code, Codex, or any agent that can run shell commands.

It finds the TV on your network, backs it up, measures why it is slow, disables telemetry, ads and unused apps in
small verified batches, replaces the home screen with a light Apple-TV-style launcher built from a **template**,
can implement your own design from Figma to the pixel, and adds startup and navigation sounds. Nothing is rooted,
flashed or uninstalled: every change can be undone with one command.

Built on [cleanyourtv](https://github.com/Gathin23/cleanyourtv) (MIT), which provides the ADB tools, the debloat
lists for 123 TVs and the launcher.

## What you need

- A computer on the same Wi-Fi as the TV with Python 3, git and ffmpeg (for sounds).
- To build the launcher: JDK 17 or newer and the Android SDK (platform 34). `scripts/doctor.sh` checks it all.
- On the TV: developer mode and USB debugging switched on (see the tutorial below; it takes two minutes).
- Works with Android TV and Google TV (TCL, Hitachi, Philips, Sony, Xiaomi, Hisense Android models, Onn,
  Chromecast with Google TV, Nvidia Shield...). Not with Samsung Tizen, LG webOS, Roku or recent Fire TV OS.

## Tutorial: switch on debugging on your TV

You do this once, with the TV remote. Nothing changes on the TV until the agent asks you to accept it.

**1. Unlock Developer options.** Open Settings (the gear icon, or the gear button on the remote) and go to the
"About" screen. Its place depends on the TV:

| Your TV | Where "About" is |
|---|---|
| Android TV (TCL, Hitachi, Philips, Sony, Xiaomi, Sharp, Nvidia Shield...) | Settings > Device Preferences > About |
| Google TV (newer Sony, TCL, Hisense, Chromecast with Google TV, Onn...) | Settings > System > About |
| Older Sony Bravia | Settings > Device Preferences > About (or Settings > About) |
| Xiaomi Mi Box / Mi TV with PatchWall | Settings > Device Preferences > About |

On that screen, highlight **Build** (on Google TV: **Android TV OS build**) and press **OK seven times**. A message
counts down and then says "You are now a developer!". If the TV asks for a PIN, it is the TV's parental-control
or account PIN.

**2. Switch on USB debugging.** Go back one screen. **Developer options** is now listed there (Device Preferences
on Android TV, System on Google TV). Open it and switch on **USB debugging**. If you also see **Network debugging**
or **ADB debugging**, switch that on too. Despite the name, this works over Wi-Fi: no cable needed.

**3. Find the TV's IP address** (the agent can usually find it on its own): Settings > Network & Internet >
your Wi-Fi network; the IP address is near the bottom, e.g. `192.168.1.23`. Make sure the computer is on the
same Wi-Fi network: some routers keep their 2.4 GHz and 5 GHz networks apart.

**4. Accept the computer.** When the agent connects, the TV shows "Allow USB debugging?". Tick
**"Always allow from this computer"**, then press **Allow**. A plain Allow only lasts one connection, and the
prompt would keep coming back.

**If port 5555 stays closed (some Google TV 12+ models):** in Developer options open **Wireless debugging**,
switch it on, choose **Pair device with pairing code**, and tell the agent the code and the two port numbers
shown. Wireless debugging switches itself off whenever the TV's Wi-Fi drops.

**When you are done:** Developer options > switch **USB debugging** (and Network/Wireless debugging) **off**, and
optionally "Revoke USB debugging authorisations". The agent will remind you: while debugging is on, anyone on
your Wi-Fi network could connect to the TV.

## Install for your agent

Clone it (`git clone https://github.com/paisanos-io/tv-makeover`), then:

| Agent | How |
|---|---|
| Claude Code | `/plugin marketplace add paisanos-io/tv-makeover` then `/plugin install tv-makeover@tv-makeover-skills`, or copy `skills/*` into `~/.claude/skills/` |
| Codex | Open the repo folder in Codex (it reads `AGENTS.md`), or copy `skills/*` into `~/.codex/skills/` |
| Any other agent | Open the repo and point the agent at `AGENTS.md`; the skills are plain `SKILL.md` files |

Inside the skills, `<repo>` means the cloned repo and `<skill>` the skill's folder, so the scripts resolve the same
way for every agent.

## Use it

Ask in your own words, for example:

- "My TCL TV is slow and full of ads, make it look like an Apple TV"
- "Connect to my TV over Wi-Fi and tell me why YouTube stutters"
- "Put the DreamOS template on my Xiaomi with Netflix, YouTube, Disney+ and Max in the dock"
- "My TV is a 2017 box with 1 GB of RAM, make it as fast as possible"
- "Make the TV's home screen exactly like this Figma frame: <link>"
- "These three mp3s are the startup, move and select sounds"

| Skill | Does |
|---|---|
| `android-tv-makeover` | The whole flow, start to finish, including closing network ADB at the end |
| `android-tv-debloat` | Find and connect to the TV, back up, diagnose, debloat in batches, extreme performance mode |
| `android-tv-launcher` | Build, configure and install the launcher from a template; name, splash, dock, inputs |
| `android-tv-launcher-design` | Turn a design (Figma, mockup or brief) into a template, verify it to the pixel, measure its GPU cost |
| `android-tv-ui-sounds` | Generate, prepare and wire startup, focus-move and select sounds |

## What is in the repo

```text
AGENTS.md    instructions for any coding agent (CLAUDE.md imports it)
skills/      the five skills, with their scripts and references
templates/   looks for the home screen: dreamos, starter, extreme
tvs/         125 tested TV profiles (123 from cleanyourtv + Hitachi CDH-LE504KSMART20 + XGIMI Elfin Flip Plus) and INDEX.md
scripts/     doctor, apply-template, extreme-performance, find-tv-profile, sync-tv-profiles, ...
```

### Templates and TV profiles

A **template** is how the home screen looks; a **TV profile** is what a given model needs (what is safe to
disable, its Live TV app, tiles to hide, firmware quirks). Any template works with any profile:

```bash
scripts/doctor.sh                                    # does this computer have what it needs?
scripts/find-tv-profile.sh tcl beyondtv android 11   # the closest tested TVs to yours
TV_HOST=192.168.x.x scripts/apply-template.sh dreamos --tv <profile> \
  --dock com.netflix.ninja,com.google.android.youtube.tv,com.disney.disneyplus,com.wbd.stream --install
```

| Template | Look |
|---|---|
| `dreamos` | Photo wallpaper that blurs over the app grid, glass dock and settings button, Inter Tight, DreamOS boot splash and sounds |
| `starter` | cleanyourtv's stock look with every branding switch added but off: the base for your own design |
| `extreme` | For very old or weak TVs: the stock layout with every animation, effect, wallpaper and sound off |

Templates are drawn for a 1920x1080 frame and render the same on TVs that report a different screen density
(`design_width_dp`, verified by forcing 240 dpi on a 320 dpi TV). How to make one: `templates/README.md`.
How to add your TV: `tvs/README.md`.

### Extreme performance mode

For TVs with 1 GB of RAM or less, Mali-400/450-class GPUs or Android 7-9: the `extreme` template plus
`scripts/extreme-performance.sh`, which turns system animations off, keeps the heavy apps you choose from running
in the background (they still open normally) and trims caches. It records every value it changes, and `undo`
puts them back exactly. Run `status` and `--dry-run` first.

## Safety

Packages are disabled with `pm disable-user`, never uninstalled, in small batches with a screenshot and a check
after each. cleanyourtv's `tools/restore.sh` puts the TV back as it was. At the end the skills ask you to switch
network ADB off: while it is on, port 5555 is open to every device on your Wi-Fi.

## Measured on a real TV

Hitachi CDH-LE504KSMART20 (TCL BeyondTV, Android 11, Realtek RTD2851, Mali-470, 2 GB): the voice assistant was
using a third of a CPU core at idle (switching it off took idle CPU from 84% to 95% free); free memory after boot
went from ~530-600 MB to ~640-690 MB; the DreamOS home runs at 5-9 ms of CPU and ~13-16 ms of GPU per frame.
Debloating alone did not make apps faster; the launcher, voice and animation changes did.

## Credits and licences

- [cleanyourtv](https://github.com/Gathin23/cleanyourtv) (MIT, (c) 2026 Gathin): ADB tools, the TV lists in `tvs/`
  and the launcher. The `launcher.patch` files in `templates/` are derived from its code, under the same licence.
- Settings icon in DreamOS: [Lucide](https://lucide.dev) (ISC). Inter Tight: SIL Open Font License 1.1 (licence
  file next to it in `templates/dreamos/assets/`).
- DreamOS's wallpaper and sounds are the owner's artwork. The starter's wallpaper and sounds were generated for
  this repo (MIT).
- Everything else: MIT (`LICENSE`).
