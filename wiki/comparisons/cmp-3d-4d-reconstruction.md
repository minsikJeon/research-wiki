---
type: comparison
title: Comparison of Feed-Forward 3D / 4D Reconstruction Methods
status: growing
tags: [3d-reconstruction, 4d-reconstruction, feed-forward, comparison]
sources:
  - "[[wang-2025-vggt]]"
  - "[[keetha-2025-mapanything]]"
  - "[[lin-2025-depth-anything-3]]"
  - "[[wang-2025-cut3r]]"
  - "[[wu-2025-point3r]]"
  - "[[karhade-2025-any4d]]"
  - "[[sucar-2026-v-dpm]]"
  - "[[luo-2026-4rc]]"
  - "[[anon-2026-trace-anything]]"
  - "[[ma-2026-fsm]]"
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[4d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[trajectory-fields]]"
  - "[[test-time-training]]"
created: 2026-05-24
updated: 2026-07-02
---

# Comparison of Feed-Forward 3D / 4D Reconstruction Methods

Synthesis across the 8 sources in the 3D/4D reconstruction batch. As with
[[cmp-tap-methods]], numbers are **not directly comparable** —
different protocols, training data, view counts. Use as navigation, not
a leaderboard.

## High-level table

| Method | Year | Static/Dyn | Mode | Output | Special trick |
|---|---|---|---|---|---|
| [[dust3r]] (DUSt3R/MASt3R) | 2024 | Static | Pairwise + post-hoc opt | Pointmaps | Original pointmap rep |
| [[vggt]] | 2025 | Static | Batch (all-at-once) | Camera + depth + pointmap + tracks | Alternating attention; over-complete heads |
| [[mapanything]] | 2025 | Static | Batch | Factored (depth + ray + pose + metric scale) | Multi-modal inputs; 12+ task configs; metric |
| [[depth-anything-3]] | 2025 | Static | Batch | Depth + ray maps | Minimalist (single transformer, 2 heads) |
| [[cut3r]] | 2025 | Static + Dyn | **Online recurrent** | Self/world pointmap + ego motion + virtual-view query | Persistent state |
| [[point3r]] | 2025 | Static + Dyn | **Online streaming** | Self/world pointmap + pose | Explicit 3D-indexed pointer memory + 3D hierarchical RoPE |
| [[v-dpm]] | 2026 | Dynamic (4D) | Batch | [[dynamic-point-maps]] | Fine-tune [[vggt]] |
| [[any4d]] | 2025 | Dynamic (4D) | Batch | Factored 4D + dense scene flow + metric | Multi-modal (RGB-D, IMU, Doppler) |
| [[4rc]] | 2026 | Dynamic (4D) | Batch + conditional query | Base geometry + ΔP per (source, target time) | Encode-once, decode anytime/anywhere |
| [[trace-anything]] | 2025 | Dynamic (4D) | Batch | [[trajectory-fields]] (per-pixel splines) | Trajectory as atomic primitive |
| [[point4d]] | 2026 | Dynamic (4D) | Batch + chunk chaining | 3D point query → 3D position at target time | 3D queries (not 2D pixels) + chunk chaining for 200+ frames |
| [[d4rt]] | 2025 | Dynamic (4D) | Batch | 2D-pixel query → 3D position at target time | Canonical query-based 4D decoder; one model → 6 tasks; SRT lineage |
| [[fsm]] (FSM-LVSM) | 2026 | Dynamic (4D) | Batch (streaming-capable) | Novel-view RGB images | [[lacet]] (elastic TTT); O(n) scaling; LVSM-style pixel decoder |
| [[fsm]] (FSM-LRM) | 2026 | Dynamic (4D) | Batch (streaming-capable) | 4D Gaussian Splatting | [[lacet]] (elastic TTT); O(n) scaling; LRM-style GS decoder |

## Headline benchmark numbers (cross-paper, take with salt)

### Multi-view static 3D (camera pose / geometry)

| Method | Pose | Geometry |
|---|---|---|
| VGGT | baseline (2025 SOTA at release) | baseline |
| MapAnything | competitive | competitive |
| Depth Anything 3 | **+35.7%** vs VGGT | **+23.6%** vs VGGT |
| **D4RT** | **best** (Sintel ATE 0.065 vs π³ 0.086 vs VGGT 0.168; Re10K AUC 83.5 vs π³ 78.7 vs VGGT 70.2) | best (Sintel L1 0.768 vs π³ 1.139 vs VGGT 1.582) |

D4RT is **simultaneously** SOTA on static-scene depth/pose **and**
4D tracking — supporting its "one model, six tasks" claim.

### Dynamic 4D — short-window (<100 frames, single chunk)

No unified eval. Each paper claims SOTA on its own / different subsets.
[[4rc]] explicitly compares against [[v-dpm]], [[any4d]],
[[trace-anything]] and claims to beat them, but the protocol details
are mixed.

- [[any4d]]: 15× faster + 2-3× lower error vs next-best 4D.
- [[v-dpm]]: more than halves error vs DPM / MonST3R / St4RTrack
  (older non-feed-forward 4D baselines).
- [[trace-anything]]: SOTA on its own new benchmark; competitive on
  TAP-Vid.
- [[4rc]]: beats prior + concurrent on standard 4D + 3D point tracking
  metrics.
- [[point4d]] (single-chunk Table 2): comparable to V-DPM / 4RC / Any4D
  — no win on single-chunk decoding; the long-video win comes purely
  from chaining.

### Dynamic 4D — long-video (200-frame, 48-frame chunks + chaining)

The first apples-to-apples long-video comparison across all feed-forward
4D methods, from [[anon-2026-point4d]] Table 1. Feed-forward baselines
use reprojection-based chaining (the best reasonable comparison);
iterative trackers use their online modes.

| Method | PointOdyssey EPE↓ | Dyn. Replica EPE↓ | PStudio EPE↓ |
|---|---|---|---|
| TAPIP3D (iter.) | 0.529 | 0.311 | 0.310 |
| SpatialTrackerV2 (iter.) | 0.529 | 0.256 | 0.131 |
| TraceAnything (ff) | **2.059** ← worst | 0.779 | 0.643 |
| Any4D (ff) | 0.939 | 0.582 | 0.471 |
| 4RC (ff) | 0.688 | 0.409 | 0.383 |
| V-DPM (ff) | 0.644 | 0.363 | 0.263 |
| **Point4D (ff)** | **0.526** | **0.256** | 0.246 |

**Key takeaways at long horizon:**

- Point4D wins among feed-forward methods on every dataset. Per-chunk
  degradation is slow (Fig 6 of source).
- [[trace-anything]]'s Bézier-spline parameterization is **empirically
  the worst at long range** (EPE 2.059 vs others <1.0). Confirms 4RC's
  earlier theoretical critique.
- SpatialTrackerV2 (iterative 3D tracker) is competitive on quality but
  is sparse-only due to GPU memory constraints.
- The 2D-query vs 3D-query difference is the load-bearing factor — see
  [[trajectory-chaining]].

### 4D NVS — Stereo4D + NVIDIA benchmarks (from [[ma-2026-fsm]] Table 3)

First direct 4D NVS comparison across feed-forward and optimization-based
methods on a common eval protocol. All at 256×256 for FSM.

| Method | Type | Stereo4D PSNR↑ | LPIPS↓ | SSIM↑ | NVIDIA PSNR↑ | LPIPS↓ | SSIM↑ |
|---|---|---|---|---|---|---|---|
| SoM (opt, ~10min) | Opt | — | — | — | 15.30 | 0.509 | 0.317 |
| MoSca (opt, ~45min) | Opt | — | — | — | 21.45 | 0.265 | 0.712 |
| 4DGT | FF | 24.62 | 0.102 | 0.785 | 14.13 | 0.640 | 0.131 |
| MoVieS | FF | 27.29 | 0.114 | 0.888 | 19.16 | 0.315 | 0.514 |
| FSM-LRM | FF | 27.19 | 0.147 | 0.876 | 20.17 | 0.337 | 0.567 |
| **FSM-LVSM** | FF | **32.16** | **0.043** | **0.931** | **23.90** | **0.105** | **0.747** |

FSM-LVSM is the clear SOTA among feed-forward 4D methods and approaches
the quality of optimization-based methods (MoSca) at orders-of-magnitude
less compute.

## Where each method wins

- **Best static 3D quality, minimal design:** [[depth-anything-3]] —
  beats VGGT on its own benchmark.
- **Best static 3D unification + metric + multi-modal:** [[mapanything]].
- **Best foundation backbone for downstream tasks:** [[vggt]] (because
  its features are most widely picked up).
- **Best online streaming 3D + dynamic:** [[cut3r]] (fixed implicit
  state) and [[point3r]] (explicit 3D-indexed pointer memory — wins at
  long sequence; loses on camera pose).
- **Best metric dense 4D + multi-modal:** [[any4d]] (uniquely supports
  RGB-D, IMU, Doppler).
- **Best smallest-change-to-add-4D-to-VGGT:** [[v-dpm]].
- **Best query flexibility:** [[4rc]] (decode any frame × any time).
- **Best dense trajectory representation:** [[trace-anything]] (if you
  trust the spline parameterization).
- **Best 4D NVS quality (feed-forward):** [[fsm]] (FSM-LVSM) — 32.16
  PSNR on Stereo4D, well above all other feed-forward methods.
- **Best long-sequence 4D scaling:** [[fsm]] (O(n) via TTT) for NVS;
  [[point4d]] (chunk chaining) for 3D tracking.

## Architectural design space (emergent)

- **No 3D inductive biases needed** (VGGT, DA3).
- **TTT as attention replacement for O(n) scaling** ([[fsm]]). New axis
  as of this source — all prior entries use O(n²) full attention.
- **Factored representation > coupled** (MapAnything, Any4D).
- **Alternating attention (frame-wise + global)** is the dominant
  backbone pattern (VGGT lineage).
- **Conditional query decoders** are an emerging design
  (4RC; conceptually similar to MapAnything's flexible input scheme).
- **Recurrent state for streaming** is now a paradigm with three
  distinct memory designs: fixed implicit state ([[cut3r]]), growing
  per-frame KV cache ([[streamvggt]], Spann3R), and explicit
  3D-indexed pointer memory ([[point3r]]). Design axis:
  what scales with — time (cache), fixed budget (state), or explored
  space (pointer).

## Open eval gaps

1. **No unified 4D benchmark protocol.** Any4D vs V-DPM vs 4RC vs
   Trace Anything need a head-to-head on identical splits + inputs.
   Current claims are mutually inconsistent.
2. **MapAnything vs DA3 vs VGGT on the same eval protocol.** DA3
   reports +35.7%/+23.6% over VGGT on its **new** benchmark; the
   delta on VGGT's original benchmark would be different.
3. **CUT3R has no direct comparison with later batch methods**
   (MapAnything, DA3) — would be informative for online-vs-batch
   trade-off study.
4. **Training-compute / data normalization:** these models vary by
   orders of magnitude in train data + compute. Apples-to-apples
   matched-budget comparison doesn't exist.

## Cross-batch connections

- **VGGT is the trunk.** Built on by [[v-dpm]], [[mapanything]],
  [[4rc]]. Competed with by [[depth-anything-3]]. Used as backbone by
  [[spatialtracker-v2]].
- **The 3D/4D reconstruction wave drives the 3D point tracking wave.**
  [[tapip3d]] uses MegaSaM (cited everywhere here); [[spatialtracker-v2]]
  uses DepthAnything + VGGT-style architecture; [[trace-anything]] is
  literally point tracking *as* 4D reconstruction.
- **The two waves are merging.** Dense trajectories
  ([[trace-anything]], [[any4d]]) are equivalent to dense 3D point
  tracking. See [[q-tracking-vs-4d-reconstruction]].

## When new 3D/4D papers arrive

Update this table. Particularly watch for:
- DUSt3R / MASt3R / VGGSfM primary ingest (currently only seeded).
- **π³ (Pi3)** — VGGT successor mentioned in multiple papers; D4RT
  reports beating it on Sintel pose AUC (78.7 → 83.5). Highest-priority
  static-3D ingest.
- **SRT (Scene Representation Transformer)** — Sajjadi 2022. The
  encoder-decoder pattern D4RT and Point4D inherit. High-priority.
- Fast3R, MV-DUSt3R+ — multi-view DUSt3R extensions.
- Spann3R — CUT3R's cache-style counterpart.
- **InfiniteVGGT / VGGT-Long** — long-sequence static 3D ancestors of
  Point4D's chaining strategy (StreamVGGT + Loger + Point3R now
  ingested).
- **St4RTrack, Flow4R, DELTA** — additional 4D + 3D-tracking methods
  Point4D and D4RT reference.
- **[[flow3r]]** (user's own paper, arXiv:2602.20157) — placeholder
  method page; ingest the PDF when available for full integration.
