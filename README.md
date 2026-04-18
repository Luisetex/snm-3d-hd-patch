# Saikyo no Mahjong 3D: HD Patch

A resolution upscale patch for **Saikyo no Mahjong 3D**. Doubles the game's internal render resolution from **1024×768** to **2048×1536** (2× native), with all gameplay and interaction paths scaling correctly.

## What's patched

- **3D scene** renders natively at 2048×1536: table, tiles, dice, wall, shadows, all sharper.
- **2D UI** (player portraits, score displays, action buttons, menus, score screen) scales uniformly to 2×.
- **Mouse input** and **UI hit-testing** work correctly at the new resolution.
- **Hand-tile picking** works correctly: tiles highlight on hover and can be clicked to discard, same as at 1×.

Nothing else in the game is changed. Saves, assets, input bindings, and game logic are all untouched. Seven byte-level edits totalling ~90 bytes.

## Before / after

[Online Comparsion Screenshots](https://twinlens.app/compare?share=f8330a714f31)

## How to install

**You need:**

1. A clean, unmodified `Saikyo no Mahjong 3D.exe` from your own copy of the game. Verify it matches the checksums below before patching.
2. The `2xmod.xdelta` patch file from this repository .
3. An xdelta patcher, either of these work:
   - [RomHacking.net online patcher](https://www.romhacking.net/patch/), browser-based, no install needed.
   - [Delta Patcher](https://www.romhacking.net/utilities/704/), desktop tool, Windows.

**Steps:**

1. Confirm your original exe matches the expected checksums (see below).
2. Open your patcher of choice.
3. Point it at your original `Saikyo no Mahjong 3D.exe` as the source file.
4. Point it at the `.xdelta` file from the release.
5. Apply. Save the output as something like `Saikyo no Mahjong 3D_HD.exe`.
6. *(Optional)*. Install [dgVoodoo2](https://dege.freeweb.hu/dgVoodoo2/) to further improve the results. You can use [these settings](/dgvoodoo2/settings.png) as a starting point.
7. Run the patched exe. Done.

If the checksums don't match and the patcher refuses to apply, you have a different build of the game than the one this patch was made from, but most probably will work with all the versions floating around Internet.

## Original file checksums

Verify these match your `Saikyo no Mahjong 3D.exe` before patching:

| Hash | Value |
|------|-------|
| CRC32 | `473528bb` |
| MD5   | `49eacc6811b4d3a8b098b48d12872bf9` |
| SHA-1 | `778da662d2799770622850f450bf3d44f69ce472` |

You can check this directly in RomHacking.net's online patcher.
