# Blender 5.2.3 — Vulkan/EEVEE shadow fix build (CI)

Minimal recipe for building a patched Blender on GitHub Actions (Linux x64) and
downloading the resulting tarball as an artifact. The repository contains only
this workflow and the patch; Blender sources and precompiled libraries are
fetched inside the job.

## What gets built

- Blender `v5.2.3` (release branch commit `e13b728c688`), plus
  `patches/0001-vulkan-eevee-shadow-fixes-5.2.3.patch`, which backports:

| Upstream commit | Fix |
| --- | --- |
| `05caad5ac73` | Vulkan: missing layout transitions of layered attachments (`add_image_barrier()` merged barriers by image handle only, so layers/mips of layered attachments kept stale contents) |
| `c394b602b67` | EEVEE Shadow: clear the render map to invalid pages (garbage page indices were sampled by the surface shadow shader) |
| `14ea6ab71f7` | EEVEE Shadow: do not reset the `SHADOW_IS_USED` tag on every shadow update loop |

## How to run

1. Actions → **Build Blender (Linux x64)** → *Run workflow*.
2. Inputs:
   - `blender_ref` — `v5.2.3` (default)
   - `apply_patch` — `true` (default)
   - `lib_commit` — `ecbd06cf6d2a4aa6b00a61ffb479fc81b17aba08` (lib submodule commit for 5.2.3)
3. Wait for the job (~1–2 h on a 4-core runner), then download the
   `blender-v5.2.3-linux-x64` artifact.

## Notes

- The precompiled `lib/linux_x64` tree (≈2.4 GB, Git LFS) is cached between
  runs, so only the first run pays for the download.
- Build options: Release, no Cycles/USD/Alembic/OpenVDB/Embree/OIDN/MaterialX/
  Freestyle/XR, audio disabled — enough for EEVEE/Vulkan testing.
- The install step produces a portable layout (`blender`, `5.2/python`,
  `lib/*.so`), packaged as `tar.gz`.
- A smoke render (`-b --gpu-backend vulkan -E BLENDER_EEVEE`, Mesa lavapipe) is
  attempted; it is allowed to fail on GPU-less runners.
- Runtime dependencies for the tarball: a Linux x64 desktop with Vulkan 1.2+
  and Mesa 25+ (or a vendor driver).
