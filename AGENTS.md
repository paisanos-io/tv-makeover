# Instructions for coding agents

This repo makes over Android TV / Google TV sets from a computer on the same Wi-Fi: diagnose, debloat, install a
fast launcher from a template, implement designs, add UI sounds. It works with any agent that can run shell
commands (Claude Code, Codex, others). Everything it does to a TV is reversible.

## Where things are

- `skills/<name>/SKILL.md`: step-by-step playbooks. Read the one that matches the task before acting:
  - `android-tv-makeover`: the whole flow and the close-up (start here if unsure)
  - `android-tv-debloat`: find and connect to the TV, back up, diagnose, debloat, extreme performance mode
  - `android-tv-launcher`: build and install the launcher from a template
  - `android-tv-launcher-design`: implement a design (Figma, mockup, brief) as a template, verify, measure
  - `android-tv-ui-sounds`: create, prepare and wire startup / move / select sounds
- `templates/`: looks for the Home screen (`dreamos`, `starter`, `extreme`); see templates/README.md.
- `tvs/`: 125 tested TV profiles; `tvs/INDEX.md`; `scripts/find-tv-profile.sh` finds the closest one.
- `scripts/`: `doctor.sh`, `apply-template.sh`, `extreme-performance.sh`, `find-tv-profile.sh`, ...

In the skills, `<repo>` means this folder and `<skill>` the skill's own folder.

## Rules

- Only reversible changes, in small batches, verified with a screenshot or a check after each.
- Ask the TV's owner before anything that depends on their habits (apps, dock, voice, casting, sounds) and before
  reboots; say what they will see on the TV.
- Report measured numbers, before and after, honestly.
- Never put the owner's IP, Wi-Fi name or account details into files meant to be shared (tvs/, templates/).
- When done, have the owner switch off network ADB (port 5555 is open to the whole LAN while it is on).
