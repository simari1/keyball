# Copilot instructions for Keyball repository (focused on firmware builds)

Purpose: targeted, repo-specific guidance so Copilot sessions can help contributors build, inspect, and modify Keyball firmware—especially the keyball44 board—without hunting for scattered docs.

---

## Quick build / test / single-target commands (keyball44 examples)

Prerequisites:

- Repository layout expected:
  - <repo root>/qmk_firmware/keyboards/keyball/...
- Firmware is built on demand via GitHub Actions, so local QMK tooling is not required for end users.

Build firmware via GitHub Actions (on-demand):

- Use the "Build a firmware on demand" workflow in the Actions UI and select keyboard: keyball44, keymap: myvia
- Or GH CLI (example):
  - gh workflow run "Build a firmware on demand" --ref main -f keyboard=keyball44 -f keymap=myvia
- Download the produced `.hex` artifact from the workflow run.
- Flash the downloaded `.hex` to the Pro Micro using REMAP or Pro Micro Web Updater:
  - REMAP: https://remap-keys.app/
  - Pro Micro Web Updater: https://sekigon-gonnoc.github.io/promicro-web-updater/index.html

CI details (how the repo builds in Actions):

- build-firmware.yml runs inside ghcr.io/qmk/qmk_cli container.
- CI uses a helper action to check out qmk_firmware and then creates a symlink: ln -s $(pwd)/qmk_firmware/keyboards/keyball **qmk**/keyboards/keyball
- CI runs: qmk compile -kb keyball/<keyboard> -km <keymap>, and uploads the produced .hex as an artifact.

---

## High-level architecture (big picture)

- Root: documentation, hardware photos, and per-board build guides.
- qmk_firmware/keyboards/keyball/: the canonical firmware source for all Keyball boards.
  - keyball44/ contains: keyball44.c, keyball44.h, config.h, info.json, rules.mk and keymaps/ (default, test, myvia, develop).
  - lib/ and drivers/ contain reusable C code shared across boards (oledkit, keyball helpers, pmw3360 driver, fonts, duplexmatrix).
  - keymaps/\*/ folders contain `keymap.c` and `config.h` for each keymap variant. `via.json` signals VIA compatibility.
- Docs: per-board build guides under keyball{N}/doc/rev1/ and qmk_firmware/keyboards/keyball/readme.md contains build/setup steps.

---

## Key conventions and project-specific patterns

- Keymap naming: `via`, `test`, `default`, `develop`, `myvia`. Use `test` for quick CI-like verification builds.
- VIA: presence of `via.json` in a keyboard folder means the keymap supports VIA remapping.
- Excluding boards from CI: adding a file named `.noci` in a keyboard directory prevents that keyboard from CI matrix builds.
- Build targets: follow QMK `keyboard:keymap` target naming. Example: `keyball/keyball44:myvia`.
- Shared code: place cross-board helpers in `lib/` or `drivers/` under `qmk_firmware/keyboards/keyball/` so CI picks them up via the symlink strategy.
- Keycodes: refer to `lib/keyball/keycodes.md` for project-specific/custom keycodes before editing keymap C files.
- CI symlink path: CI exposes the repo code inside the qmk workspace at `__qmk__/keyboards/keyball`—mirrors the build workflow.

---

## Files to consult when responding or changing code

- qmk_firmware/keyboards/keyball/readme.md — canonical build instructions and recommended QMK version.
- qmk_firmware/keyboards/keyball/keyball44/\* — source and keymaps for keyball44 (primary example board).
- .github/workflows/build-firmware.yml — exact CI steps and container image used.
- qmk*firmware/keyboards/keyball/lib/* and drivers/\_ — shared code to reuse or modify cautiously.

---

## Communication Style

- Be friendly and constructive in Japanese style with light ギャル語, positive tone, and emoji
- Use です・ます調 for technical accuracy
- Provide specific, actionable feedback
- Suggest improvements with code examples when possible
- Prioritize critical issues while noting minor style improvements

---

Guidance for Copilot edits:

- Prefer small, focused changes to C sources or keymap.c files. For new features that affect multiple boards, propose changes in lib/ and drivers/ with an example keymap change for keyball44.
- When suggesting build/test steps, reference the qmk_firmware symlink + `make SKIP_GIT=...` or `qmk compile` examples shown above.

Summary: created repo-focused instructions with concrete keyball44 examples for building single firmwares, the CI flow, high-level layout, and conventions to follow when editing firmware.
