---
type: comparison
title: Comparison of TAP (Point Tracking) Methods
status: growing
tags: [point-tracking, comparison, 3d-point-tracking, online-tracking]
sources:
  - "[[harley-2022-pips]]"
  - "[[doersch-2023-tapir]]"
  - "[[zholus-2025-tapnext]]"
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
  - "[[zhang-2025-tapip3d]]"
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[3d-point-tracking]]"
  - "[[synthetic-to-real-gap]]"
created: 2026-05-24
updated: 2026-06-26
---

# Comparison of TAP Methods

Synthesis across the 9 sources currently in the wiki. Caveat: numbers are
**not directly comparable** across papers — different evaluation protocols,
backbones, training data scales, and resolutions. Treat as a navigation
table, not a leaderboard.

## High-level table

| Method | Year | Latency | Dim | Training | Joint? | Backbone | Defining trick |
|---|---|---|---|---|---|---|---|
| [[pips]] | 2022 | window (T=8) | 2D | synthetic (FlyingThings++) | no | CNN + MLP-Mixer | 8-frame iterative refinement over corr pyramids; chained at test time |
| [[tapir]] | 2023 | any-length offline | 2D | synthetic (Panning MOVi-E) | no | TSM-ResNet + depthwise conv | global per-frame init + iterative depthwise refinement + self-supervised uncertainty |
| [[cotracker]] | 2024 | window | 2D | synthetic | yes | transformer | cross-track attention + proxy tokens |
| [[cotracker3]] | 2024 | window (online + offline) | 2D | synthetic + pseudo-labeled real (15K) | yes | transformer | random-teacher distillation |
| [[tapnext]] | 2025 | **frame** | 2D | synthetic (500K × 48) | implicit (ViT) | SSM + ViT (TRecViT) | masked-token decoding + classification coord head |
| [[track-on2]] | 2025 | **frame** | 2D | synthetic-only | no | DINOv3 + memory transformer | classification-first + expandable memory |
| [[tapnext-plus-plus]] | 2026 | **frame** | 2D | synthetic (1024-frame Kubric) | implicit (ViT) | SSM + ViT (same as TAPNext) | long-sequence training + roll aug + AJRD metric |
| [[tapip3d]] | 2025 | window | 3D | synthetic + sensor RGB-D | yes (3D N2N) | iterative refinement transformer | world-coord lifting + 3D N2N attention |
| [[spatialtracker-v2]] | 2025 | window | 3D | 17-dataset mix (synth + real) | yes (SyncFormer) | dual-branch 2D/3D transformer + BA | end-to-end joint depth + pose + tracking |

## Headline benchmark numbers (cross-paper, take with salt)

### TAP-Vid DAVIS (queried-first / strided, AJ ↑)

| Method | AJ (queried-first) | AJ (strided) |
|---|---|---|
| [[pips]] (FlyingThings++, 8-frame chained) | 33.0 | 42.0 |
| [[tapir]] (Panning MOVi-E) | 56.2 | 61.3 |
| [[tapir]] high-res (1080p) | — | 65.7 |
| CoTracker3 (online, +15k real) | ~63 | — |
| TAPNext-B (synth-only) | 62.4 | — |
| BootsTAPNext-B | 65.2 | 68.9 |
| TAPNext++ | ≥ BootsTAPNext-B | — |
| Track-On2 (DINOv3, synth-only) | beats TAPTRv3 + TAPNext-B in AJ + δ_avg | — |

### PointOdyssey (~2400 frames, long-horizon)

The clearest performance separator:

- **Window/video methods** (CoTracker3, LocoTrack): OOM or sustained drift.
- **Frame-by-frame causal methods** ([[track-on2]], [[tapnext-plus-plus]]):
  stable. TAPNext++ is the strongest on AJRD (re-detection) after roll
  augmentation.

### TAPVid-3D (AJ ↑ / APD3D ↑)

| Method | AJ | APD3D |
|---|---|---|
| BootsTAPIR (Type I baseline) | ~9–12 | ~14–19 |
| DELTA | ~13 | ~20 |
| [[tapip3d]] (GT depth) | new SOTA per paper | new SOTA |
| [[spatialtracker-v2]] | **21.2** | **31.0** |

No head-to-head [[tapip3d]] vs [[spatialtracker-v2]] under a unified
protocol exists yet — open eval gap.

### Latency (online tracking, 256 query pts, H100 unless noted)

| Method | FPS | Latency (ms) |
|---|---|---|
| TAPNext-B | 197 | **5.05** |
| TAPNext++ (1024 pts!) | 348 | <10 |
| Track-On2 | >30 (different setup) | low |
| [[tapir]] (JAX/JIT, 50 pts) | ~150 | ~7 (per inference, batched across frames) |
| CoTracker3 (online) | 102 | 80 |
| LocoTrack-B | 452 | 2210 (windowed) |
| [[pips]] (chained, V100) | ~3 | ~30000 (sequential chunks; the bottleneck TAPIR fixed) |

TAPNext / TAPNext++ are 15-40× lower latency than the next-best
quality-competitive methods.

## Where each method wins

- **Best raw 2D AJ + lowest latency, synthetic-only:** [[tapnext-plus-plus]]
  (TAPNext architecture, 1024-frame training).
- **Best 2D AJ with real-data fine-tuning, low complexity:** [[cotracker3]]
  (1000× less data than BootsTAPIR).
- **Best long-sequence / re-detection robustness:** [[tapnext-plus-plus]]
  (especially with roll aug) and [[track-on2]] (memory-based).
- **Best 3D tracking quality (modular pipeline):** [[tapip3d]] when
  reliable depth is available.
- **Best 3D tracking quality + speed (end-to-end):** [[spatialtracker-v2]].
- **Best dense / quasi-dense 2D tracking (70K points):** [[cotracker]] /
  [[cotracker3]] via proxy tokens.

## Recurring axes (linked concept pages)

- [[online-vs-offline-tracking]] — the latency framing.
- [[joint-point-tracking]] — independent vs. shared-info tracks.
- [[3d-point-tracking]] — modular vs. end-to-end paradigms.
- [[synthetic-to-real-gap]] — pseudo-labels vs. better-synthetic camps.
- [[pseudo-labeling-point-tracking]] — recipes.

## Open eval gaps

1. **No direct [[tapip3d]] vs [[spatialtracker-v2]] head-to-head** — both
   claim SOTA on TAPVid-3D under different conditions (GT depth + GT
   pose, vs end-to-end with estimated everything).
2. **No causal-3D tracker** — combining [[tapnext]]'s per-frame design
   with 3D awareness is an open slot.
3. **AJRD is new** — most prior methods haven't been re-evaluated under
   it; only [[jung-2026-tapnext-plus-plus]] reports it.
4. **Training-budget normalization** — comparing methods at matched
   train-compute / data-scale would change a lot of the rankings.

## When new tracking papers arrive

Update this table. If a new method significantly shifts the layout
(e.g. SOTA on multiple axes at once), promote findings to [[overview]].
