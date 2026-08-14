---
type: method
title: TrackCraft3R
status: stub
tags: [3d-point-tracking, point-tracking, video-model, diffusion-transformer, dense-tracking, feed-forward]
sources: []
related:
  - "[[motionforesight]]"
  - "[[bharadhwaj-2026-motionforesight]]"
  - "[[3d-point-tracking]]"
  - "[[4d-reconstruction]]"
created: 2026-07-16
updated: 2026-07-16
---

# TrackCraft3R

Feed-forward **dense 3D point tracker** built on a video diffusion
transformer (**Wan2.1**), by Nam, Koo, Son, Jung, An, Hur, Kim
(arXiv:2605.12587, 2026). **Retrospective**: observes the whole clip,
encodes RGB frames + reconstructed pointmaps into geometry latents,
repeats a frame-0 query latent across time, and predicts
**reference-anchored tracking pointmaps** — the 3D position of each
reference-frame point at every timestamp.

> **Stub — not yet ingested (no PDF in `raw/`).** Documented because it
> is the load-bearing backbone of [[motionforesight]]. Priority ingest.

## Why it's in the wiki

[[motionforesight]] repurposes TrackCraft3R's learned "video latent →
reference-anchored 3D track" interface: by hiding future frames (mask
latents) it flips the *retrospective* tracker into a *prospective*
forecaster while freezing the backbone. TrackCraft3R runs the video DiT
as a **single-step latent regressor** at a fixed regression timestep
(not an iterative diffusion sampler) — a deterministic single pass that
MotionForesight inherits.

## Key structure (as used by MotionForesight)

- **Two time-aligned latent streams:** context stream `c_t` (per-frame
  RGB+pointmap) + query stream `r_t` (frame-0 RGB+pointmap, repeated ∀t).
- **Output:** residual-track latent `ẑ_t^Δ` → frozen track decoder →
  `X̂_t(q) = X_0(q) + D_track(ẑ_t^Δ)(q)`.
- Trained with a large (**rank-1024**) tracking LoRA on the Wan2.1 DiT.

## Related

- **Consumer:** [[motionforesight]] (forecasting repurpose).
- **Family:** feed-forward [[3d-point-tracking]] / dense
  [[4d-reconstruction]] on video-model backbones.
