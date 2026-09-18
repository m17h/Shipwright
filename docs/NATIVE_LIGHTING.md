# Native Lighting Development

This document tracks the staged native-rendering work on the
`graphics/native-lighting` branch. Native graphics features must remain
optional, preserve the original renderer as a fallback, and never change the
game's 20 Hz simulation behavior.

## Current Foundation

The first milestone is a DirectX 11 post-processing and depth-inspection pass.
When `gSettings.NativePostProcessing` is enabled, the Fast3D interpreter renders
the game into an off-screen color/depth framebuffer even when MSAA and output
resolution would otherwise allow direct backbuffer rendering. A full-screen
triangle then copies the resolved color into a second framebuffer, which the
existing ImGui compositor presents.

`gSettings.NativePostProcessingDebugView` selects one of three views:

- `0`: scene color, an intentionally color-neutral identity pass;
- `1`: raw nonlinear DirectX device depth; and
- `2`: linear view-space depth, mapped to white at
  `gSettings.NativePostProcessingDepthRange` game units (default `2000`).

The interpreter recovers the actual near and far planes from perspective
projection matrices loaded during the frame and retains the projection with the
widest valid depth span. This rejects short-range interface projections while
avoiding hard-coded camera planes. Fixed-point N64 matrix precision can make the
recovered far plane differ slightly from the source value; the current default
`12800` far plane is observed as approximately `12860` after conversion.

The pass establishes:

- a toggleable post-processing boundary;
- shader-readable scene color;
- shader-readable scene depth and projection-derived view-depth reconstruction;
- an MSAA-resolve path before post-processing;
- separate single-sample and 2x-through-8x MSAA depth readers (the closest
  sample is used for multisampled depth);
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
- The scene-color, raw-device-depth, and linear-view-depth modes each launched
  under DirectX 11 at 640x480 and 1x MSAA, remained responsive, and logged their
  active mode without errors.
- The linear-depth mode also launched at 4x MSAA, exercising the dedicated
  multisampled depth reader and the color resolve-before-process path.
- The recovered scene projection logged near `10.00` and far `12860.24`; the
  configured debug display range was `2000.00`.
- No error, critical, exception, or crash entry appeared in the test log.
- The lighting runtime's ignored configuration currently enables the native
  pass, leaves the debug view on scene color, and sets a 2000-unit linear-depth
  display range for continued development testing.

This verifies compilation, shader creation, mode dispatch, projection recovery,
and basic stability. It does not verify pixel equality, visual depth correctness,
or long-session stability. The Windows Computer Use runtime required for a
window capture was unavailable in this session, so visual equivalence and depth
orientation still need captured comparisons.

## Next Milestones

1. Capture matched scene-color, raw-depth, and linear-depth images and verify
   orientation, depth gradients, gamma, HUD, and pixel-center sampling.
2. Capture matched post-processing-disabled and scene-color images to establish
   pixel equivalence.
3. Exercise window resize, fullscreen, advanced resolution, and MSAA values 1,
   2, 4, and 8; 1x and 4x currently have launch-level coverage.
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
