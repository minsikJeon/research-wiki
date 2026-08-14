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

## Summary
**Problem**
	 Existing 3d trackers are trained on synthetic data with gt tracks, or finetune 3d static reconstruction models to 4D. They lacks the prior of real-world motion.
**idea**
	 Repurpose Video DiT into a 3D Tracker.
	 As video DiT is frame-anchored model (generates per-frame content), convert it into ref-anchored model.
**key details**
	1. forward track tokens (rgb + pointmap of 1st (=ref) frame) + geometry tokens (rgb + pointmap of each frame) through video DiT
	2. use temporal attention, to distinguish track tokens at each timestep.
	3. Intuition here: for point in frame-0, let them find correspondence in frame-j, where we can directly retrieve 3d coordinate of them also. During finding corrrespondecne, attend more to nearby temporal frames via temporal RoPE.
**What I learned**
	 video DiT learns how to denoise noisy video, by learning what patches should each patch attend to in order to denoise itself. To this end, it learns how to correlate pixels (=spatiotemporal priors) given a video, which is useful for tracking/reconstructions (probably)
**remaining questions**
	 geometry comes from external model
	 w/o video DiT initialization, still better than Any4D
		 due to pointmap input?
	 **What does Video DiT lack? (gap btw video DiT vs tracker)** 
		- accuracy: no need pixel-level accuracy for correspondence
		- repeated texture: can attend to visually similar, but physically different points.
		- occlusion: no need to reason where the occluded point is

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
