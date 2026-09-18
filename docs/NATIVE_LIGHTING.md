# Native Lighting Development

This document tracks the staged native-rendering work on the
`graphics/native-lighting` branch. Native graphics features must remain
optional, preserve the original renderer as a fallback, and never change the
game's 20 Hz simulation behavior.

## Current Foundation

The first milestone is a DirectX 11 identity post-processing pass. When
`gSettings.NativePostProcessing` is enabled, the Fast3D interpreter renders the
game into an off-screen color/depth framebuffer even when MSAA and output
resolution would otherwise allow direct backbuffer rendering. A full-screen
triangle then copies the resolved color into a second framebuffer, which the
existing ImGui compositor presents.

The pass is intentionally color-neutral. It establishes:

- a toggleable post-processing boundary;
- shader-readable scene color;
- retained shader-readable scene depth for future effects;
- an MSAA-resolve path before post-processing;
- automatic target recreation at the current game resolution; and
- an unchanged fallback path when the feature is disabled or unsupported.

DirectX 11 is the only implemented backend. OpenGL and Metal report the feature
as unsupported and continue through the original path. The in-game N64 HUD is
part of the processed image, while Shipwright's ImGui menus and floating windows
are composited afterward and remain unaffected.

## Verified 2026-09-18

- Release configuration and full build completed successfully.
- The original disabled path launched under DirectX 11 and entered scene
  `0x51` without a crash.
- The enabled path launched under DirectX 11, remained responsive, and logged
  `Native post-processing identity pass active (640x480)`.
- A separate enabled run with 4x MSAA exercised the resolve-before-process path,
  remained responsive, and logged the active identity pass without errors.
- No error, critical, exception, or crash entry appeared in the test log.
- The lighting runtime's ignored configuration currently enables the identity
  pass for continued development testing.

This verifies execution and basic stability, not pixel equality or long-session
stability. The available Windows capture interface could not enumerate the game
window during this test, so visual equivalence still needs a captured
before/after comparison.

## Next Milestones

1. Add color and linearized-depth debug views.
2. Capture matched before/after images and verify orientation, gamma, HUD, and
   pixel-center sampling.
3. Exercise window resize, fullscreen, advanced resolution, and MSAA values 1,
   2, 4, and 8.
4. Add GPU timing for the native pass and establish 120 FPS and 240 FPS budgets.
5. Add exposure and tonemapping controls with an exact original-output mode.
6. Add restrained multi-resolution bloom.
7. Add depth-based contact shading before attempting directional shadow maps.

## Baseline Scene Matrix

Use the same save, camera position, time of day, resolution, and interpolation
rate for each comparison.

| Scenario | Purpose |
| --- | --- |
| Kokiri Forest daylight | foliage, alpha edges, bright color response |
| Hyrule Field day/night | outdoor range, sky, sun/moon direction |
| Torch-lit interior | local lights, bloom threshold, dark detail |
| Water or reflective surface | transparency and depth interactions |
| Fog-heavy scene | depth reconstruction and atmospheric work |
| Pause menu and HUD-heavy play | UI contamination and compositing order |
| Cutscene | camera interpolation and temporal stability |

Run meaningful captures at the original presentation rate, 120 FPS, and 240
FPS. ReShade must remain disabled for all native-renderer comparisons.
