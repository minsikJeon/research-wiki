---
type: method
title: MotionForesight
status: growing
tags: [point-tracking, 3d-point-tracking, manipulation, world-model, scene-flow, video-model, forecasting, lora, frozen-backbone]
sources:
  - "[[bharadhwaj-2026-motionforesight]]"
related:
  - "[[trackcraft3r]]"
  - "[[objectforesight]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[track2act]]"
  - "[[mu0]]"
  - "[[dex4d]]"
  - "[[pointworld]]"
  - "[[depth-anything-3]]"
  - "[[3d-point-tracking]]"
  - "[[4d-reconstruction]]"
  - "[[homanga-bharadhwaj]]"
created: 2026-07-16
updated: 2026-08-27
---

# MotionForesight

## One-line summary

Repurpose a **retrospective** dense 3D tracker ([[trackcraft3r]], on the
Wan2.1 video DiT) into a **future 3D scene-flow forecaster** by hiding
future frames — replace future RGB/pointmap latents with learned mask
latents, keep the frame-0 query + temporal RoPE, and train only a small
adapter. Predicts reference-anchored dense 3D tracks of a manipulated
object from a short human-video prefix. No language, no action, no policy.

## Inputs / outputs

- **In:** `T1` observed RGB frames `I_{0:T1-1}` + their estimated
  pointmaps `P_{0:T1-1}`; query points `Q` from the object mask in
  reference frame `a = 0`. Default **T1 = 7**.
- **Out:** future 3D positions `X̂_t(q) ∈ R³` for each query point `q`,
  for `t = T1 .. T1+T2-1`. Default **T2 = 15** (so `T = 22`). Expressed
  in the **last-observed camera frame** (`t = T1-1`).
- **Not used:** language, action labels, future RGB/geometry.

## How it works

1. **Observed context (§3.2).** For each observed frame, encode RGB and
   pointmap separately and concatenate: `c_t = [E_rgb(I_t); E_pm(P_t)]`,
   `t < T1`. Pointmap norm stats use only the observed prefix.
2. **Repeated query stream (§3.3).** `r_t = [E_rgb(I_0); E_pm(P_0)]`,
   repeated at *every* timestamp — turns each output slot into "where is
   each frame-0 point at time t?" This is TrackCraft3R's native
   reference-anchored interface.
3. **Mask the future (§3.4).** For `t ≥ T1` there is no observation:
   `c̃_t = [m_rgb; m_pm]`, learned mask latents **shared across all
   future slots**, broadcast spatially. **Temporal RoPE** distinguishes
   the slots by time index despite identical mask content.
4. **Predict + decode.** The frozen video DiT (+ fresh LoRA) emits a
   residual-track latent `ẑ_t^Δ`; the **frozen** track decoder gives
   `X̂_t(q) = X_0(q) + D_track(ẑ_t^Δ)(q)`. Single deterministic pass
   (fixed regression timestep, no iterative diffusion sampling).

## What is trained

| Frozen | Trained |
|---|---|
| Wan2.1 video DiT | fresh **rank-32** forecasting LoRA |
| TrackCraft3R tracking LoRA (**rank 1024**) | I/O projections |
| RGB + pointmap VAE enc/dec | prediction head |
| track decoder | mask latents `m_rgb`, `m_pm` |

Tiny trainable footprint — the whole point. Same frozen-backbone +
small-adapter philosophy as [[mu0]], applied to the *tracker* rather
than to a trace-expert WM.

## Loss

Decoded coordinate-space, validity-masked, future-weighted:

```
L_dec = 0.25 · L_observed + 1.0 · L_future
```

where each term is `Σ v·‖X̂ − X‖² / (Σ v + ε)` over its time range
(`v_t^q` = per-point validity). Observed term stabilizes the inherited
tracking interface; future term does the forecasting. Backprops through
the frozen decoder (weights not updated). Handles dense (mask-sampled)
and sparse (native-tracked) supervision via one query-point interface.

## Data pipeline (offline pseudo-GT)

40K [[something-something-v2|SSv2]] clips → **SAM** (object-name-cued
anchor frame, propagate mask fwd/back) → dense query points →
**[[depth-anything-3|DepthAnything3]]** depth + camera → pointmaps →
**[[trackcraft3r]]** on the *complete* clip → reference-anchored 3D
tracks + validity, in last-observed frame. Future frames used **only**
to make labels; never fed to the forecasting model.

## Where it's been applied

- [[bharadhwaj-2026-motionforesight]] — trained on 40K SSv2; evaluated on
  150 held-out SSv2 + 50 OOD phone videos. Beats MolmoMotion (1.16M
  videos + language) and a video-gen-then-track baseline on ADE/FDE/PWT +
  five motion-conditional metrics. Best at 40K training scale.

## Known limitations

- **Deterministic single future** on a multimodal task — no uncertainty /
  multi-hypothesis output.
- **Pseudo-GT dependence** — depth/camera/segmentation/tracking error
  propagates; sensitive to noisy observed pointmaps at inference.
- **Under-predicts motion magnitude** (r̄ = 0.72) — timid trajectories.
- **No robot deployment** — plan/forecast only; connecting to control is
  future work.
- Trained only on 40K SSv2; egocentric / large-motion / long-horizon
  regimes untested.

## Related methods

- **Backbone:** [[trackcraft3r]] — the retrospective dense 3D tracker
  (Wan2.1 video DiT) that this method inverts into a forecaster.
- **Author lineage:** [[track2act]] (same author [[homanga-bharadhwaj]])
  — 2D tracks from web video for a policy; MotionForesight is the 3D,
  forecasting, video-prior-repurposing successor without the policy.
- **Closest sibling:** [[mu0]] — video model → future 3D tracks + frozen
  backbone + no action labels; but µ0 predicts semantic B-spline
  keypoints under language and feeds an action expert. MotionForesight
  predicts dense scene flow from passive observation and stops at plan.
- **Conditioning-axis opposite:** [[pointworld]] — *action-conditioned*
  dynamics WM vs. MotionForesight's *unconditioned* anticipation.
- **Sibling (same author):** [[objectforesight]] — forecasts rigid **6-DoF
  pose** instead of dense scene flow, via a **diffusion** DiT trained from
  scratch (vs MotionForesight's frozen-video-DiT + LoRA). Flow ⊃ pose
  (MF's argument) but ObjectForesight's diffusion is **multimodal** where
  MotionForesight is deterministic — each answers the other's weakness.
- **Thread:** [[point-tracks-as-manipulation-interface]] — the
  forecasting/plan-only member (no action policy). See
  [[cmp-point-track-manipulation]].
- **Not-yet-ingested neighbors:** MolmoMotion (language + 8-point sparse,
  1.16M videos), Gen2Act (video-gen-then-act — the baseline it argues
  against).
