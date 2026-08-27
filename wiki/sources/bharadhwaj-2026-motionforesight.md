---
type: source
source_type: paper
title: "MotionForesight: Re-purposing Video Models for Future 3D Scene-Flow Prediction"
authors: [Bharadhwaj, Homanga; Jangir, Yash]
year: 2026
venue: "arXiv preprint (2026)"
url: https://motionforesight.github.io
raw_path: papers/motionforesight_paper.pdf
status: ingested
tags: [point-tracking, 3d-point-tracking, manipulation, world-model, scene-flow, video-model, forecasting, web-video, 4d-reconstruction]
sources: []
related:
  - "[[motionforesight]]"
  - "[[objectforesight]]"
  - "[[soraki-2026-objectforesight]]"
  - "[[trackcraft3r]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[bharadhwaj-2024-track2act]]"
  - "[[lee-2026-mu0]]"
  - "[[kuang-2026-dex4d]]"
  - "[[homanga-bharadhwaj]]"
  - "[[johns-hopkins]]"
  - "[[depth-anything-3]]"
  - "[[3d-point-tracking]]"
  - "[[4d-reconstruction]]"
created: 2026-07-16
updated: 2026-08-27
---

# MotionForesight

## TL;DR

**Forecast future 3D scene flow** — reference-anchored dense 3D tracks
of points on a manipulated object — from a short monocular human video
prefix, **without any language or action labels**. The key move: take
[[trackcraft3r]] (a *retrospective* feed-forward dense 3D tracker built
on the Wan2.1 video DiT) and **convert it into a forecaster by hiding
the future frames** — replace future RGB/pointmap latents with learned
mask latents, keep the frame-0 query + temporal RoPE, and train only a
fresh rank-32 LoRA + I/O projections + prediction head + mask latents
while freezing the entire video backbone, tracking LoRA, VAEs, and
track decoder. Trained on just **40K SSv2 human videos**, it beats
MolmoMotion (trained on 1.16M videos *with* language) and a
video-generation-then-track baseline. From **Homanga Bharadhwaj**
([[bharadhwaj-2024-track2act]] author, now at **Johns Hopkins**) + Yash
Jangir.

## Why it matters

Squarely in the [[point-tracks-as-manipulation-interface]] thread (E),
but occupies a **new coordinate**: it is a **forecasting / plan-only**
method — it predicts *what will happen* to the object from passive
observation and stops there. No robot policy, no action conditioning,
no deployment (all flagged as future work). This makes it the
**upstream plan-generation half** of the interface, isolated from the
control half.

Three things make it notable:

1. **Repurposing a video-model tracker as a forecaster is nearly free.**
   Rather than train a forecaster from scratch or generate future pixels
   then re-track them, MotionForesight redirects an existing
   retrospective tracker's learned "video-latent → 3D track" interface
   toward prediction, changing only *which frames are observed*. The
   trainable footprint is a rank-32 LoRA + a few projections + mask
   tokens. This is the same **frozen-backbone + tiny-adapter** philosophy
   as [[mu0]] (freeze the trace-expert WM, train a small action expert),
   applied one level earlier.
2. **Geometry beats pixels for motion.** A video-generation-then-track
   baseline (Wan-VACE → the same 3D track pipeline) is decisively worse
   (Table 1: ADE 11.20 vs 4.47) — visually plausible future pixels do
   not preserve point correspondence or metric 3D motion. Predicting the
   actionable geometric variable *directly* also skips the cost of
   rendering RGB then reconstructing.
3. **Data-efficient anticipation.** 40K passive SSv2 clips, no language,
   beats language-conditioned MolmoMotion trained on 1.16M videos. Strong
   geometric grounding converts observed context into metric future
   motion better than a much larger language-paired corpus.

## Key claims

- **Best on every metric vs. both baselines**, with *no* auxiliary input
  (Table 1). SSv2 unseen (150 clips): ADE **4.47** / FDE **6.23** /
  PWT@5cm **76**. OOD phone videos (50): ADE **9.31** / FDE **14.88** /
  PWT **54**. MolmoMotion-with-language: 5.93 / 9.38 / 68 (SSv2). Video
  generation + tracks (no lang): 11.20 / 12.58 / 40.
- **Long-context (Table 2):** observe first 50% of clip, forecast the
  rest. MotionForesight ADE **10.20** vs MolmoMotion 11.90 (no lang) /
  12.57 (with lang) on OOD phone videos.
- **Data scaling (Table 3):** 1K → 10K → 40K SSv2. 40K best on all three
  SSv2 metrics (4.47 / 6.23 / 76) and highest OOD PWT; but OOD ADE/FDE
  are **non-monotonic** (10K slightly better) — expected for a
  multimodal task scored against a single recorded future.
- **Motion-conditional diagnostics (Table 4):** MotionForesight (40K)
  leads all five learned metrics — TVO 0.231, VVO 0.175, MoveF1 0.618,
  MoveIoU 0.448, DQS 0.326 — with TVO/DQS improving monotonically 1K→40K.
  Motion ratio **r̄ = 0.72** → the model *under-predicts* motion
  magnitude ("too timid"); calibration is the main weakness.
- **Three contributions:** (1) formalize future 3D scene-flow prediction
  from casual RGB interaction video as **reference-anchored
  tracking-pointmap forecasting**; (2) a data-curation pipeline turning
  human videos into pseudo-GT future tracks; (3) a **minimal
  modification** to TrackCraft3R that flips it from track *extraction* to
  track *prediction*.

## Methods

### Task setup

- Given `T1` observed RGB frames, predict future 3D tracks of object
  points for the next `T2` steps. Default **T1 = 7, T2 = 15, T = 22**.
- Query points `Q` sampled from the object mask in reference frame
  `a = 0`. Predict `X̂_t(q) ∈ R³` for `t = T1..T1+T2-1`.
- Everything expressed in the **last-observed camera frame** (`t = T1-1`)
  to cancel apparent camera motion.
- **No language / action input** to the model. Text used only offline to
  name the object for masking.

### Data curation: RGB video → pseudo-GT 3D tracks (§3.1)

Offline on ~40K [[something-something-v2|SSv2]] clips:

1. Pick an intermediate **anchor frame** where the object is clear; use
   the dataset object name to init **SAM** there; propagate mask
   backward + forward.
2. Densely sample query points inside the object mask.
3. **[[depth-anything-3|DepthAnything3]]** per-frame depth + recovered
   camera → temporally aligned pointmaps.
4. Run **[[trackcraft3r]]** on the *complete* clip → reference-anchored
   3D trajectories + validity masks, transformed to last-observed frame.

Future frames feed the pipeline **only offline** to make labels; the
forecasting model never sees them.

### Turning tracking into forecasting (§3.2–3.4)

TrackCraft3R uses two time-aligned latent streams:

```
context stream  c_t = [E_rgb(I_t) ; E_pm(P_t)]      (varies per frame)
query stream    r_t = [E_rgb(I_0) ; E_pm(P_0)]      (frame-0, repeated ∀t)
```

Repeating the query turns every output slot into the same question:
*where is each reference-frame point at time t?* The forecasting change:

```
observed  t < T1:  c̃_t = [E_rgb(I_t) ; E_pm(P_t)]     (real latents)
future    t ≥ T1:  c̃_t = [m_rgb ; m_pm]               (learned mask latents)
```

`m_rgb`, `m_pm` are **shared across all future slots**, broadcast
spatially — they signal "unobserved," not a specific appearance.
**Temporal RoPE** gives each slot a distinct time index so near- vs
long-horizon slots differ despite identical mask content. The DiT emits
a residual-track latent `ẑ_t^Δ`; a **frozen** track decoder maps it to a
3D residual:

```
X̂_t(q) = X_0(q) + D_track(ẑ_t^Δ)(q)
```

Pointmap normalization uses **only the observed prefix** (future geometry
unavailable at inference). TrackCraft3R runs the DiT as a single-step
latent regressor (fixed regression timestep), not an iterative diffusion
sampler — MotionForesight inherits this deterministic single-pass
interface.

### Frozen vs. trained

- **Frozen (blue):** Wan2.1 video DiT, original TrackCraft3R tracking
  LoRA (**rank 1024**), RGB + pointmap VAE encoders/decoders, track
  decoder.
- **Trained (green):** fresh **rank-32 forecasting LoRA**, I/O
  projections, prediction head, and the mask latents `m_rgb`, `m_pm`.

### Objective (§3.5)

Decoded coordinate-space loss, validity-masked, future-weighted:

```
L_dec = λ_obs · (Σ_{t<T1}  Σ_q v_t^q ‖X̂_t(q) − X_t(q)‖² / (Σ v + ε))
      + λ_fut · (Σ_{t≥T1} Σ_q v_t^q ‖X̂_t(q) − X_t(q)‖² / (Σ v + ε))
```

Defaults **λ_obs = 0.25, λ_fut = 1.0** — the observed term stabilizes the
inherited tracking interface; the future term does the real forecasting.
Gradients backprop *through* the frozen decoder (its weights are not
updated). Supports both dense (mask-sampled) and sparse (native-tracked)
supervision via a common query-point interface.

## Results

### Table 1 — SSv2 + OOD phone (same T1/T2 for all)

| Method | Input | SSv2 ADE↓ | FDE↓ | PWT↑ | OOD ADE↓ | FDE↓ | PWT↑ |
|---|---|---|---|---|---|---|---|
| **MotionForesight** | None | **4.47** | **6.23** | **76** | **9.31** | 14.88 | **54** |
| MolmoMotion (no lang) | null action | 5.66 | 8.90 | 70 | 9.50 | 16.05 | 53 |
| Video gen + tracks | None | 11.20 | 12.58 | 40 | 13.82 | 17.65 | 32 |
| MolmoMotion (+ lang) | action desc | 5.93 | 9.38 | 68 | 9.94 | 17.16 | 51 |
| Video gen + tracks (+ lang) | action desc | 11.99 | 13.57 | 44 | 13.63 | **16.71** | 29 |

ADE/FDE in cm; PWT = PWT@5cm. Rows below the rule use privileged
language the model does not.

### Table 3 — data scaling

| Train videos | SSv2 ADE / FDE / PWT | OOD ADE / FDE / PWT |
|---|---|---|
| 1K | 4.81 / 6.57 / 74 | 9.48 / 14.74 / 53 |
| 10K | 4.72 / 6.38 / 73 | **8.97** / **14.63** / 52 |
| 40K | **4.47** / **6.23** / **76** | 9.31 / 14.88 / **54** |

### Table 4 — motion-conditional (SSv2, 108 moving clips)

| Method | TVO↑ | VVO↑ | MoveF1↑ | MoveIoU↑ | DQS↑ | r̄→1 |
|---|---|---|---|---|---|---|
| **MotionForesight (40K)** | **0.231** | **0.175** | 0.618 | **0.448** | **0.326** | 0.72 |
| MotionForesight (10K) | 0.167 | 0.127 | 0.582 | 0.388 | 0.237 | 0.57 |
| MolmoMotion (lang) | 0.122 | 0.089 | 0.586 | 0.269 | 0.225 | 1.46 |
| Video gen + tracks (lang) | 0.157 | 0.137 | 0.613 | 0.415 | 0.256 | 0.90 |

New metrics defined in the paper: **TVO** (displacement-vector overlap),
**VVO** (velocity-vector overlap), **MoveF1** / **MoveIoU** (does it move
the right points, by the right amount), **DQS** = geomean(TVO, MoveF1),
**r̄** (motion-magnitude calibration; <1 timid, >1 overshoot).

## Limitations / open questions

- **Deterministic, single-future** prediction on a **multimodal** task —
  a short prefix admits several plausible futures, yet both standard
  (ADE/FDE/PWT) and motion-conditional metrics score against one recorded
  trajectory. Authors call for probabilistic / multi-hypothesis
  forecasting + coverage-aware evaluation.
- **Pseudo-GT supervision** — depth, camera, segmentation, and tracking
  errors propagate into training *and* eval; inference is sensitive to
  noisy observed pointmaps.
- **Under-predicts motion magnitude** (r̄ = 0.72) — the model is timid;
  magnitude calibration is an open direction.
- **40K SSv2 only** — no egocentric/head-mounted video, large camera
  motion, long-horizon, or non-manipulation dynamics tested. OOD is
  phone captures of tabletop-style scenes.
- **No downstream policy yet.** Connecting predicted 3D tracks to
  planning/control is explicitly deferred — the paper is the *plan* half
  of the interface, unproven on a robot.
- Appendix A/B (metric definitions, hyperparameters) not fully extracted
  this pass — flagged in the method page.

## Connections

- **Direct author lineage — [[bharadhwaj-2024-track2act]]:** same first
  author, [[homanga-bharadhwaj]], who has moved **CMU → Johns Hopkins**
  (new "Brains, Bots, and Behavior Lab", [[johns-hopkins]]). Track2Act
  predicts **2D** point tracks from web video for a robot policy;
  MotionForesight is the **3D + forecasting + video-prior-repurposing**
  evolution, and stops at the plan (no policy).
- **Closest sibling — [[lee-2026-mu0]]:** both take a **video model →
  future 3D tracks** and both use the **frozen-backbone + small-adapter**
  recipe with **no action labels**. Differences: µ0 predicts *semantic
  B-spline keypoints* under language/task conditioning and feeds frozen
  WM features to a downstream action expert; MotionForesight predicts
  *dense reference-anchored scene flow* from passive observation with no
  conditioning and no policy. µ0 is a WM-for-control; MotionForesight is
  a pure anticipatory forecaster.
- **vs. [[pointworld]]:** PointWorld is *action-conditioned* dynamics
  ("what happens given this action?"); MotionForesight is *unconditioned*
  anticipation ("what will happen from what I've seen?"). Opposite ends
  of the conditioning axis.
- **Backbone — [[trackcraft3r]]** (Nam et al. 2026, arXiv:2605.12587):
  the retrospective dense 3D tracker on Wan2.1 that MotionForesight
  repurposes. Not yet ingested — priority, since it is load-bearing here.
- **Concurrent baseline — MolmoMotion** (Zhang et al. 2026,
  arXiv:2606.18558, Krishna group): forecasts **8** sparse 3D points
  under **language** from a 1.16M-video corpus (MolmoMotion-1M). Shares
  the point-trajectory output but opposite supervision regime
  (language-grounded sparse vs. passive dense). Not yet ingested.
- **Sibling from same author — [[objectforesight|ObjectForesight]]**
  ([[soraki-2026-objectforesight]]; Soraki, Bharadhwaj, Farhadi, Mottaghi
  2026, arXiv:2601.05237): predicts future **rigid 6-DoF object pose**
  trajectories from human video. MotionForesight argues scene flow is
  strictly more general (handles articulated + deformable + local nonrigid
  motion that pose cannot); conversely ObjectForesight's **diffusion** DiT
  handles the multimodal one-to-many future that MotionForesight's
  deterministic model cannot. Now ingested — the pose-vs-flow contrast.
- **Anti-baseline — Gen2Act** (Bharadhwaj et al. 2025): the
  generate-video-then-act paradigm MotionForesight's "video gen + tracks"
  baseline stands in for, and argues against on both cost and metric
  fidelity.
- **Perception substrate — [[depth-anything-3]]:** monocular depth in the
  pseudo-GT pipeline (shared with the wider 4D-reconstruction wave).
- **Concept / comparison:** [[point-tracks-as-manipulation-interface]],
  [[cmp-point-track-manipulation]]. Also touches [[4d-reconstruction]] /
  [[3d-point-tracking]] — it is a *forecasting* extension of dense 3D
  tracking, adjacent to the feed-forward 4D wave but predictive rather
  than retrospective.

## Citation

```
Bharadhwaj, H., & Jangir, Y. (2026). MotionForesight: Re-purposing Video
Models for Future 3D Scene-Flow Prediction. Brains, Bots, and Behavior
Lab, Johns Hopkins University. Project: motionforesight.github.io
```
