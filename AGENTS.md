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
- Remote tracking branch: `origin/develop`
- Executable: `x64\Release\soh.exe`
- Purpose: the dependable version used for normal play.
- The 3DS-style asset overhaul lives in this folder's ignored runtime `mods`
  directory.
- ReShade belongs only in this stable runtime unless explicitly testing native
  lighting interactions. ReShade 6.8.0 is installed here for DirectX 11 as of
  2026-09-18 and was verified to load successfully.

### Native-lighting development version

- Folder: `C:\Users\puzzl\Documents\AI Projects\Shipwright-Lighting`
- Branch: `graphics/native-lighting`
- Remote tracking branch: `origin/graphics/native-lighting`
- Purpose: isolated renderer, shader, shadow, lighting, fog, and post-processing
  development.
- Build and run a separate executable from this folder.
- Do not install ReShade here by default. Evaluate native rendering with ReShade
  disabled so external post-processing cannot hide regressions.
- Keep the stable worktree playable while experimental work is incomplete.
- Verified independent Release runtime: `x64\Release\soh.exe`.
- The lighting runtime has its own copied configuration and save, plus the same
  39 selected 3DS-style mod archives for comparable visual testing.
- A DirectX 11 smoke test passed on 2026-09-18 and closed normally.

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

## Verified Windows Build Procedure

The installed CMake is bundled with Visual Studio Build Tools rather than added
to the system `PATH`:

`C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe`

From the selected worktree root, configure and build Release with:

```powershell
$cmake = 'C:\Program Files (x86)\Microsoft Visual Studio\2022\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe'
& $cmake -S . -B 'build/x64' -G 'Visual Studio 17 2022' -T v143 -A x64 -DCMAKE_BUILD_TYPE:STRING=Release
& $cmake --build .\build\x64 --config Release --target GenerateSohOtr
& $cmake --build .\build\x64 --config Release --parallel
```

The Release executable is written to `x64\Release\soh.exe`. `GenerateSohOtr`
writes `soh.o2r`; ensure the runtime directory also contains the valid `oot.o2r`
generated from the verified ROM. Build output and generated archives are ignored.

Desktop shortcuts currently identify the two runtimes:

- `Ocarina of Time - Stable`
- `Ocarina of Time - Lighting Dev`

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

Current stable-runtime installation:

- Signed standard ReShade 6.8.0 build, loaded through ignored `d3d11.dll`.
- Active runtime preset: ignored `x64\Release\ReShadePreset.ini` (ReShade's
  first-run default name).
- Version-controlled enhanced preset source: `docs\reshade\OoT-Stable.ini`.
- Lightweight fallback preset: `docs\reshade\OoT-Clean-120.ini`.
- Shader sources: the official indexed `crosire/reshade-shaders` slim package
  at commit `6db142b4b1a05c764222e5b0bd9a644b7ccfe1dc` and Marty McFly's qUINT package
  at commit `98fed77b26669202027f575a6d8f590426c21ebd`.
- Enhanced enabled effects, in order: qUINT MXAO, qUINT SSR, qUINT Debanding,
  qUINT Lightroom, qUINT Bloom, and qUINT DELC Sharpen. The MXAO build enables
  indirect lighting, smooth reconstructed normals, and two AO scales through
  ReShade preprocessor definitions. Depth of field remains disabled for normal
  gameplay because it obscures the scene and HUD.
- ReShade 6 preset activation keys must be at the top level of the preset. Do
  not put `Techniques` or `TechniqueSorting` under a `[GENERAL]` section; ReShade
  will leave the real top-level `Techniques=` empty and no effects will run.
- DirectX 11 depth was visually calibrated on 2026-09-18 with DisplayDepth. The
  buffer is upright, non-reversed, non-logarithmic, and shows correct scene
  normals; keep all three corresponding depth flags at `0`.
- Press `Home` while the stable game is running to open the ReShade overlay.
- Stable runtime interpolation is explicitly set to 120 FPS with
  `gSettings.InterpolationFPS=120` and `gSettings.MatchRefreshRate=0`. VSync
  remains enabled; the current physical NVIDIA display reports 239 Hz, so it
  does not clamp the 120 FPS target. This changes visual frame presentation,
  while the game simulation remains at its original 20 Hz.
- ReShade's FPS counter reports presented/rendered frames, not the 20 Hz game
  logic tick. A steady 20 FPS there means interpolation is not active. With the
  enhanced six-effect preset active, Shipwright's built-in statistics window
  reported 119.0 FPS / 9.194 ms on 2026-09-18.
- The 2026-09-18 test on the RTX 4070 Ti SUPER loaded ReShade 6.8.0.2158 and
  compiled all six enhanced effects successfully. The generated search paths
  must end in one `**`; the installer's `**\**` form failed on this version.

- Do not commit or redistribute the ReShade injector or third-party shader
  binaries.
- Keep the project-authored preset and documentation committed, and copy
  `docs\reshade\OoT-Stable.ini` to the ignored
  `x64\Release\ReShadePreset.ini` after changes.
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
