---
type: source
source_type: paper
title: "Point4D: Long-range 4D Motion Reconstruction"
authors:
  - Anonymous (NeurIPS 2026 submission, double-blind review)
year: 2026
venue: NeurIPS 2026 (under review)
url: https://openreview.net/forum?id=TBD
raw_path: "papers/4D_NeurIPS_2026 (9).pdf"
status: ingested
tags: [4d-reconstruction, long-sequence, 3d-query, trajectory-chaining, feed-forward, 3d-point-tracking, dense-tracking]
sources:
  - "[[karhade-2025-any4d]]"
  - "[[luo-2026-4rc]]"
  - "[[sucar-2026-v-dpm]]"
  - "[[anon-2026-trace-anything]]"
  - "[[xiao-2025-spatialtracker-v2]]"
  - "[[zhang-2025-tapip3d]]"
  - "[[lin-2025-depth-anything-3]]"
  - "[[wang-2025-vggt]]"
  - "[[wang-2025-cut3r]]"
related:
  - "[[point4d]]"
  - "[[4d-reconstruction]]"
  - "[[3d-point-tracking]]"
  - "[[trajectory-chaining]]"
  - "[[d4rt]]"
  - "[[depth-anything-3]]"
created: 2026-05-26
updated: 2026-05-29
---

# Point4D: Long-range 4D Motion Reconstruction

> **Authorship (private note, not for redistribution):** This is the
> **user's (Minsik Jeon's) first-authored work**, NeurIPS 2026
> currently under double-blind review. Confirmed by user on
> 2026-05-29. The formal `authors:` field above is kept as "Anonymous"
> to mirror the venue's double-blind status; the wiki is private and
> this note exists for internal continuity. When explaining or
> critiquing Point4D, remember the lead author is the audience.
> Advised by [[shubham-tulsiani]] at [[cmu-ri]].

## TL;DR

Point4D directly attacks the **biggest limitation** of the current
feed-forward 4D wave: methods like [[any4d]], [[4rc]], [[v-dpm]],
[[trace-anything]] are stuck at **<100-frame windows**. Point4D chains
overlapping chunks autoregressively. The key trick is **3D queries
instead of 2D pixels** — a 3D point coordinate stays well-defined under
occlusion / out-of-frame, so the predicted endpoint of one chunk can be
**directly re-queried in the next** without reprojection or matching.
Adds a flexible visual descriptor (extract once from any frame where
the point is visible, reuse across all chunks). NeurIPS 2026 anonymous;
encoder + geometry heads initialize from [[depth-anything-3]].

## Why it matters

**Closes a specific gap in [[overview]]:** the prior 3D/4D batch all
operate on short windows; long-horizon 4D was an open slot. Point4D
is the first feed-forward method to reliably do dense 4D over **200-300
frames**.

The 3D-query insight is the conceptual novelty:

- 2D-pixel queries (D4RT, Trace Anything, 4RC) require the point to be
  *visible* in the source frame. Across chunk boundaries with occlusion
  or out-of-frame events — which are normal in long videos — they break.
- 3D queries decouple point identity from image-plane visibility. The
  3D coordinate is always well-defined. Chain by `Sim(3)`-transforming
  the predicted endpoint into the next chunk's frame.

This also directly addresses [[q-tracking-vs-4d-reconstruction]]:
Point4D's design is **3D-tracker-like queries on a 4D-reconstruction
backbone** — explicit fusion of the two threads.

## Key claims

- **3D queries > 2D pixel queries.** Ablation Table 3: switching from
  2D to 3D pixel queries improves single-chunk EPE 1.022 → 0.803 on
  PointOdyssey; long-video survival rate 0.245 → 0.388. (Full Point4D
  with arbitrary-frame patches: 0.526 / 0.535.)
- **Visual descriptor from *any* visible frame.** Not just the source
  frame. Necessary for chunk chaining — the descriptor can be sourced
  from a different chunk entirely.
- **Direct re-querying via Sim(3)**. No reprojection (which is undefined
  for occluded points and compounds camera prediction error).
- **SOTA on long-video tracking** (Table 1, 200-frame seqs, 48-frame
  chunks with 8-frame overlap):
  - **PointOdyssey:** EPE 0.526 (vs V-DPM 0.644, 4RC 0.688, TraceAnything
    2.059); survival 0.535.
  - **Dynamic Replica:** EPE 0.256 (ties SpatialTrackerV2 0.256);
    APD 0.759.
  - **PStudio:** survival 0.643 (vs V-DPM 0.640, 4RC 0.542).
- **Comparable to peers on single-chunk** (Table 2) — the long-video win
  is from chaining, not a better decoder.
- **Per-chunk degradation is slow.** V-DPM starts higher on chunk 1
  but Point4D surpasses it within a few chunks (Fig 6).

## Methods

**Encoder.** Standard ViT with alternating frame-wise + global attention
(same pattern as [[vggt]] / [[depth-anything-3]] / [[4rc]]). Outputs
scene representation `F` + per-frame camera tokens `c_i` + time tokens
`t_i`. Camera head predicts pose `P_i`; DPT head predicts depth `D_i`.
**Initialized from [[depth-anything-3]] pretrained weights**; decoder is
trained from scratch.

**3D query construction.**

- Select a pixel `(u, v)` visible in some frame `t_ref`; extract local
  RGB patch `S` as visual descriptor.
- Lift to 3D at source time `t_src`: `p = (x, y, z)` in frame
  `t_src`'s camera coordinates. When `t_ref ≠ t_src`, the point may have
  moved and be occluded at `t_src` — fine, the 3D coord is well-defined
  regardless.
- Full query: `(p, t_src, t_tgt, t_cam, S)` — "where is the 3D point at
  position `p` (in frame `t_src`'s coord) with visual descriptor `S` at
  time `t_tgt`, expressed in camera `t_cam`'s coord?"

**Query embedding.** Fourier-encode `p`; pass `t_src` / `t_cam` through
camera-token MLP; pass `t_tgt` through time-token MLP. Sum with `S`.

**Decoder.** Lightweight cross-attention to `F` (no self-attention
between queries → each query decoded independently → arbitrary batching
at inference).

**Loss.** `L = L_point + L_conf + L_2d + L_vis` where `L_point` is L1
under a signed-log transform (dampens far-away points); `L_conf`
modulates by per-query confidence; `L_2d` is reprojection consistency;
`L_vis` is per-query visibility BCE.

**Trajectory chaining (Algorithm 1).**

1. Partition `V` into `K` overlapping chunks (48-frame, 8-frame overlap).
2. Lift initial 2D queries to 3D at `t=0`.
3. For each chunk: encode → decode trajectories.
4. At overlap, estimate `Sim(3)` from dense depth on shared frames →
   transform endpoint to next chunk's coords → re-query with **same**
   visual descriptor `S`.

**Training.** Two phases. Phase 1: width 252, 8-18 frames per sample.
Phase 2: width 420, 4-7 frames. Mix of dynamic GT-trajectory datasets
(PointOdyssey, Dynamic Replica, Kubric Movi-F, CoTracker-Kubric,
Waymo) + static datasets (ScanNet, ScanNet++, BlendedMVS, Co3Dv2,
WildRGBD) treated as stationary trajectories.

## Results (headline)

**Long-video tracking (200 frames, Table 1):**

| Method | PointOdyssey EPE↓ | Dyn. Replica EPE↓ | PStudio EPE↓ |
|---|---|---|---|
| TAPIP3D | 0.529 | 0.311 | 0.310 |
| SpatialTrackerV2 | 0.529 | 0.256 | 0.131 |
| TraceAnything | 2.059 | 0.779 | 0.643 |
| Any4D | 0.939 | 0.582 | 0.471 |
| 4RC | 0.688 | 0.409 | 0.383 |
| V-DPM | 0.644 | 0.363 | 0.263 |
| **Point4D** | **0.526** | **0.256** | 0.246 |

Point4D beats all feed-forward 4D baselines on long video and is
competitive with iterative 3D trackers (which need expensive per-frame
optimization).

**Single-chunk tracking (Table 2):** Point4D is comparable to V-DPM /
4RC / Any4D (not always best). The long-video gain is purely from the
3D-query chaining.

## Limitations / open questions

- **Point must be visible in at least one frame per chunk.** A point
  absent from all frames in a chunk cannot be tracked within it.
- **Depth-prediction error compounds across chunks** via the `Sim(3)`
  alignment.
- **No comparison with offline / per-scene-optimization methods** like
  MegaSaM or Shape of Motion at the same long-horizon — Point4D's
  claim is "feed-forward SOTA at long range," not "best 4D at long
  range."
- **Chunk-size / overlap is a hyperparameter** (48 / 8 in eval).
  Sensitivity analysis is in appendices not yet read.
- Anonymous submission — reproducibility hinges on the promised release.

## Connections

- Method: [[point4d]].
- Concept introduced: [[trajectory-chaining]] (for 4D).
- **Direct conceptual predecessor:** [[d4rt]] (Zhang et al. 2025,
  [[zhang-2025-d4rt]]) — introduced the query-based 4D decoder for 2D
  pixels. Point4D explicitly extends D4RT to 3D queries. Same encoder
  pattern, same independent-cross-attention decoder, same SRT lineage.
  D4RT is now ingested as a primary source — see that page for full
  D4RT capabilities (six 4D tasks via query variation).
- **Backbone:** [[depth-anything-3]] (initialization for encoder +
  geometry heads).
- **Architectural pattern:** [[vggt]] alternating attention.
- **Direct competitors (4D feed-forward):** [[any4d]], [[4rc]],
  [[v-dpm]], [[trace-anything]].
- **3D point-tracking competitors:** [[tapip3d]],
  [[spatialtracker-v2]] (iterative; not dense).
- **Long-sequence 3D ancestors cited:** [[cut3r]], **InfiniteVGGT**
  [Yuan et al. 2026], **StreamingVGGT** [Zhuo et al. 2025],
  **VGGT-Long** [Deng et al. 2025], **Loger** [J. Zhang et al. 2026]
  — all do long static 3D, not 4D. Promote on 2nd mention.
- **Datasets used:** PointOdyssey ([[pointodyssey-dataset]]), Dynamic
  Replica ([[dynamic-replica-dataset]]), PStudio (from
  [[tapvid-3d-dataset]]), LSFOdyssey (defer dataset page), Kubric
  ([[kubric-dataset]]), Co3Dv2, WildRGBD, ScanNet, ScanNet++,
  BlendedMVS, Waymo (defer until 2+ mentions).
- **Newly cited methods to track (defer until 2nd mention):**
  D4RT, Flow4R, DELTA, St4RTrack, Shape of Motion, OmniMotion (already
  in the wiki references), MonST3R, SceneTracker.
- **User-affinity (primary):** Point4D *itself* is the user's
  first-authored work (see authorship note at top of page). The user
  is therefore the lead author of one of the wiki's load-bearing
  contributions, and discussions of long-range 4D / 3D-query design
  should treat them as an insider, not as someone needing
  introduction.
- **User-affinity citation:** Point4D reference [3] is [[flow3r]]
  (Cong, Zhao, **Jeon**, Tulsiani, arXiv:2602.20157, 2026) — the
  user's co-authored paper with advisor [[shubham-tulsiani]]. Point4D
  cites Flow3R as a peer in the feed-forward visual-geometry-prediction
  lineage. Promote Flow3R to a primary source page when the PDF lands
  in `raw/papers/`.

## Citation

Anonymous. (2026). *Point4D: Long-range 4D Motion Reconstruction.*
NeurIPS 2026 submission, under double-blind review.
