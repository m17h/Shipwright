# Project Operating Guide

Keep this file current. Updating `AGENTS.md` is a required part of any task that
changes the repository layout, worktrees, branches, remotes, build procedure,
runtime files, graphics packages, important settings, architectural decisions,
or verified project state. Do not finish such a task with stale instructions.

## Project Goal

This is the user's independent Ship of Harkinian fork for modifying Ocarina of
Time and running it natively on PC hardware. The long-term target is a polished
high-frame-rate presentation, normally 120 FPS and potentially 240 FPS, without
changing gameplay behavior that depends on the original 20 Hz game logic.

Rendering, animation interpolation, post-processing, and native lighting may run
at the display frame rate. Do not tie gameplay, physics, timers, actor updates,
or other game logic to the render frame rate.

## Repository and Remote Safety

- User fork: `https://github.com/m17h/Shipwright`
- `origin` fetch/push must point to the user fork.
- `upstream` may fetch from `https://github.com/HarbourMasters/Shipwright.git`.
- `upstream` push must remain disabled (`DISABLED`).
- `remote.pushDefault` must remain `origin`.
- Never push to, open a pull request against, or otherwise modify the upstream
  HarbourMasters repository.
- Pull requests, when requested, must target the user's fork.
- Before pushing or creating a pull request, verify remotes and the destination.

## Two-Folder Workflow

Do not make the user manually switch branches in one directory. The two Git
worktrees have distinct purposes:

### Stable/playable version

- Folder: `C:\Users\puzzl\Documents\AI Projects\Shipwright`
- Branch: `develop`
- Executable: `x64\Release\soh.exe`
- Purpose: the dependable version used for normal play.
- The 3DS-style asset overhaul lives in this folder's ignored runtime `mods`
  directory.
- ReShade belongs only in this stable runtime unless explicitly testing native
  lighting interactions. ReShade has not yet been installed as of 2026-09-18.

### Native-lighting development version

- Folder: `C:\Users\puzzl\Documents\AI Projects\Shipwright-Lighting`
- Branch: `graphics/native-lighting`
- Purpose: isolated renderer, shader, shadow, lighting, fog, and post-processing
  development.
- Build and run a separate executable from this folder.
- Do not install ReShade here by default. Evaluate native rendering with ReShade
  disabled so external post-processing cannot hide regressions.
- Keep the stable worktree playable while experimental work is incomplete.

Always verify both the absolute working directory and current branch before
editing, building, committing, or testing.

## ROM and Generated Assets

- Supported ROM currently used: NTSC 1.2 USA.
- Verified ROM SHA-1: `41b3bdc48d98c48529219919015a1af22f5057c2`.
- Stable ROM path: `roms\oot-ntsc-1.2-us.z64`.
- ROMs, generated `.otr`/`.o2r` game assets, save data, logs, build directories,
  ReShade binaries, and downloaded third-party mod archives must not be committed.
- It is acceptable to use a local hard link or copy of the verified ROM in the
  lighting worktree so that its build remains independent.

## Current 3DS-Style Graphics Setup

The stable runtime uses the community-made `Djipi's 3DS Experience + Skilar's
Art Plus Link` version 6.0.2 from:

`https://gamebanana.com/mods/477979`

This recreates the 3DS visual style without requiring an Ocarina of Time 3D ROM;
it is not a literal extraction of Nintendo's 3DS assets. Preserve author credits.
Installation details and checksums are recorded in the ignored runtime file:

`x64\Release\mods\INSTALLED_3DS_GRAPHICS.md`

Current required settings:

- Use Alternate Assets: enabled
- Disable Grotto Fixed Rotation: enabled
- Enable 3D Dropped Items/Projectiles: enabled
- Fix Out of Bounds Textures: disabled per the pack author's crash guidance

The selected install excludes the Original N64 HUD, Majora Chest, Crescent Moon,
generic ARIA variant, and base Link texture archive. Art Plus Link models and
their 3DS texture options replace the base Link texture archive.

## ReShade Policy

ReShade is the temporary external graphics layer while native lighting is under
development. Install it only from `https://reshade.me/` against the stable
DirectX 11 `soh.exe`.

- Do not commit or redistribute the ReShade injector or third-party shader
  binaries.
- A project-authored preset and documentation may be committed when created.
- Favor restrained ambient occlusion, bloom, tonemapping, color grading, and
  sharpening over effects that obscure the image.
- Maintain a 120 FPS-oriented preset and make expensive effects optional for
  240 FPS targets.
- Disable ReShade for native-renderer comparisons and performance baselines.

## Native Lighting Direction

Develop native graphics changes on `graphics/native-lighting`. The intended
incremental direction is:

1. Modern post-processing infrastructure and configurable graphics controls.
2. Directional sun/moon shadow maps using interpolated render transforms.
3. Contact shadows or ambient occlusion.
4. Better torch, fairy, spell, and explosion lighting.
5. Improved fog, atmospheric distance, bloom, and tonemapping.
6. Optional cel-shaded and more natural lighting presets.
7. Later material improvements such as normal and roughness maps where assets
   support them.

Prefer small, toggleable stages with before/after validation. Preserve original
rendering as a fallback. Do not merge experimental renderer work into `develop`
until it builds cleanly, runs cleanly, and has been visually and performance
tested with ReShade disabled.

## Testing and Integration

- Verify the current worktree and branch before every test build.
- Test rendering changes at both the original presentation and high frame rates.
- Confirm gameplay logic remains at 20 Hz.
- Record meaningful performance measurements, visual regressions, crashes, and
  renderer/backend limitations rather than relying only on a short smoke test.
- When native lighting is ready, review and merge it deliberately into `develop`,
  rebuild the stable executable, then remove or disable ReShade.
- Keep unrelated user changes intact and never use destructive Git operations.
