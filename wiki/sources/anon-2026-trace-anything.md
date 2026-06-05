---
type: source
source_type: paper
title: "Trace Anything: Representing Any Video in 4D via Trajectory Fields"
authors:
  - Liu, Xinhang
  - Xiao, Yuxi
  - Chen, Donny Y.
  - Feng, Jiashi
  - Tai, Yu-Wing
  - Tang, Chi-Keung
  - Kang, Bingyi
year: 2025
venue: arXiv (cs.CV) — ICLR 2026 submission
url: https://arxiv.org/abs/2510.13802
raw_path: papers/attachment.pdf
status: ingested
tags: [4d-reconstruction, trajectory-fields, video-representation, feed-forward, point-tracking, dense-tracking]
sources: []
related:
  - "[[trace-anything]]"
  - "[[trajectory-fields]]"
  - "[[4d-reconstruction]]"
  - "[[point-tracking]]"
  - "[[3d-point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# Trace Anything: Representing Any Video in 4D via Trajectory Fields

> ⚠️ **Raw file rename suggested:** the raw PDF is currently
> `raw/papers/attachment.pdf` — a generic name. Recommend renaming to
> `liu-2025-trace-anything.pdf` to match the now-known authorship. The
> wiki intentionally never modifies `raw/`; user action required.
> The wiki slug `anon-2026-trace-anything` is also outdated — the paper
> is no longer anonymous (see citation below). Keeping the slug for now
> to avoid breaking links; rename in a future refactor.

## TL;DR

Trace Anything proposes a new **4D video representation**: the
**Trajectory Field** — a dense per-pixel-per-frame mapping to a
**parametric (spline) 3D trajectory**. A single feed-forward transformer
predicts the field in one pass, *without* depth, flow, or tracking
estimators, and *without* per-scene optimization. Beats existing methods
on a new trajectory-field benchmark, **competitive on TAP-Vid**, with
**significant efficiency gains**. Enables goal-conditioned manipulation,
motion forecasting, and spatio-temporal fusion. ICLR 2026 anonymous.

## Why it matters

This paper sits **at the intersection** of the two main threads in the
wiki:

- **From the point-tracking side:** pushes [[point-tracking]] from
  sparse trajectories (TAP family) to **dense per-pixel** trajectory
  prediction.
- **From the 4D reconstruction side:** instead of per-frame pointmaps
  (V-DPM, Any4D), uses **continuous parametric trajectories** as the
  atomic primitive of video.

The conceptual claim: pixels naturally trace 3D curves over time, so a
4D representation should model **the trajectory directly**, not derive
it from per-frame snapshots + post-hoc matching. This is a clean
unification — if it holds up to scrutiny.

Three explicit downstream applications demonstrate the representation's
breadth: motion forecasting (extrapolate the spline), spatio-temporal
fusion (sample at arbitrary time), goal-conditioned manipulation
(plan in trajectory space).

## Key claims

- **Trajectory Field:** dense mapping from each `(pixel, frame)` to a
  **parametric 3D trajectory** (spline / Bézier-style).
- **Single feed-forward pass** over all input frames — no depth, flow,
  tracking sub-models; no global alignment.
- **Generalizes across input types:** monocular videos, image pairs,
  unordered photo sets capturing dynamic scenes (note: "any video"
  caveat).
- **Per-frame control point maps** parameterize each pixel's trajectory.
- **New synthetic data + benchmark.** Blender-based platform; 10K+ training
  videos × 120 frames; 200 curated benchmark videos. Released.
- **Beats SOTA on the trajectory-field benchmark; competitive on
  established TAP benchmarks** at higher efficiency.
- **New capabilities:** goal-conditioned manipulation, motion
  forecasting, spatio-temporal fusion.

## Methods

- **Input:** N RGB frames (any number; can include image pairs / photo
  sets).
- **Network:** feed-forward transformer; for each input frame, outputs
  a stack of **control point maps** that together define a spline
  trajectory per pixel.
- **Representation:** spline-based parametric 3D trajectory; degree and
  control point count are model hyperparameters.
- **Training data platform:** Blender-based with photo-realistic
  rendering; provides 2D/3D trajectories, depth, semantics, optical
  flow, camera poses.

## Results (headline)

- SOTA on the new trajectory-field benchmark.
- Competitive on TAP-Vid (point tracking) at much higher efficiency.
- Qualitative: goal-conditioned manipulation demos, motion forecasting,
  spatio-temporal fusion.

## Limitations / open questions

- Spline parameterization may struggle with **high-frequency / abrupt
  motion** (explicitly noted by [[4rc]]'s related-work critique:
  "TraceAnything's Bézier curves often struggle with complex or
  high-frequency dynamics and may compromise geometric accuracy").
- "Any video*" caveat — performance on truly unstructured photo sets
  remains a TBD claim.
- Anonymous submission — final author list / institution unknown;
  reproducibility depends on the promised open release.
- No metric-scale claim (unlike [[any4d]], [[mapanything]]).

## Connections

- Method: [[trace-anything]].
- Concept introduced: [[trajectory-fields]].
- Concepts: [[4d-reconstruction]], [[point-tracking]],
  [[3d-point-tracking]].
- **Closest concurrent / contemporary work:**
  - [[any4d]] — dense + metric, but scene flow only from view 1.
  - [[v-dpm]] — extends VGGT to DPMs; per-frame, not trajectory-based.
  - [[4rc]] — conditional query for any frame/time; explicit critique of
    Trace Anything's Bézier representation.
- **Lineage cited:** DUSt3R, Fast3R, [[vggt]], π³, [[mapanything]],
  MegaSaM, MonST3R, POMATO, Easi3R, St4RTrack, Dynamic Point Maps
  (= predecessor of [[v-dpm]]).
- **Point-tracking comparison:** beats or matches CoTracker / TAPIR /
  TAPNext family at the dense-trajectory granularity.
- **Authorship note:** lead author **Yuxi Xiao** is also the lead of
  [[xiao-2025-spatialtracker-v2]] — Trace Anything is conceptually his
  3D-tracking line extended to dense 4D. **Bingyi Kang** (DA3 lead,
  ByteDance Seed) is also on this paper. Confirms the
  ByteDance/HKUST/Bingyi-Kang cluster as a distinct research line
  (parallel to the CMU and Oxford-VGG lines).
- **Critique from [[anon-2026-point4d]] (Table 1):** TraceAnything's
  long-video numbers are the *worst* among feed-forward 4D baselines
  (EPE 2.059 on PointOdyssey-200, vs Point4D 0.526). The Bézier-spline
  parameterization indeed struggles at long horizons — confirming
  4RC's earlier critique.

## Citation

Liu, X., Xiao, Y., Chen, D. Y., Feng, J., Tai, Y.-W., Tang, C.-K., &
Kang, B. (2025). *Trace Anything: Representing Any Video in 4D via
Trajectory Fields.* arXiv:2510.13802. ICLR 2026 submission.
https://arxiv.org/abs/2510.13802
