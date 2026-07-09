---
type: concept
title: 4D Reconstruction (Dynamic 3D + Time)
status: growing
tags: [4d-reconstruction, 3d-reconstruction, video-understanding, scene-flow]
sources:
  - "[[karhade-2025-any4d]]"
  - "[[sucar-2026-v-dpm]]"
  - "[[luo-2026-4rc]]"
  - "[[anon-2026-trace-anything]]"
  - "[[ma-2026-fsm]]"
  - "[[anon-2026-stride]]"
  - "[[allshire-2025-videomimic]]"
related:
  - "[[3d-point-tracking]]"
  - "[[pointmap-representation]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[trajectory-fields]]"
  - "[[dynamic-point-maps]]"
  - "[[test-time-training]]"
  - "[[videomimic]]"
created: 2026-05-24
updated: 2026-07-09
---

# 4D Reconstruction (Dynamic 3D + Time)

## Definition

Recover both the **3D structure** and **how it evolves over time** from a
video. "4D" = 3D space + 1D time. Includes scene shape, scene flow,
camera intrinsics, camera extrinsics, sometimes per-pixel motion or
trajectories. The natural extension of [[feed-forward-3d-reconstruction]]
to dynamic scenes.

## Why it matters

- **Robotics:** predictive control, manipulation, navigation in dynamic
  environments all need 4D scene state.
- **Generative AI:** dynamic video synthesis, 4D scene generation,
  controllable video editing.
- **Embodied AI / AR/VR:** continuously understand how the world is
  changing around you.

## The 2025-2026 feed-forward 4D wave

A coherent batch of papers attacking the same problem with overlapping
but distinct strategies (all in this wiki):

| Method | Representation | Key trick |
|---|---|---|
| [[v-dpm]] | [[dynamic-point-maps]] (time-variant + time-invariant) | fine-tune [[vggt]] |
| [[any4d]] | factored: ego (depth+ray) + allo (pose+scene flow) + global metric scale | multi-modal inputs (RGB-D, IMU, Doppler) |
| [[4rc]] | base geometry + time-dependent displacement; queryable | encode-once, decode-anytime/anywhere |
| [[trace-anything]] | [[trajectory-fields]] (per-pixel parametric splines) | atomic-level dense trajectories |
| [[fsm]] | novel-view images (LVSM) or 4D Gaussians (LRM) | [[lacet]] (elastic TTT); O(n) long-sequence scaling |
| [[stride]] | 3DGS with per-Gaussian velocity + learned instance assignment | camera+LiDAR fused in 3D point space via PTv3; first self-supervised feed-forward decomposition |

All are **feed-forward** (no per-scene optimization), all built on top of
or in dialog with [[vggt]] / [[depth-anything-3]] / DUSt3R lineage, all
contemporary (Dec 2025 – Jun 2026). **No unified eval protocol exists
yet** — apples-to-apples comparison is an open community gap.
[[stride]] is the **first driving-scene multi-modal** entry; previous
entries are image-only (Any4D supports RGB-D/IMU/Doppler but reasons
in factored 2.5D space, not 3D point space).

## Older / non-feed-forward 4D approaches (cited but not yet ingested)

- **MegaSaM:** per-scene optimization; high quality but slow.
- **MonST3R, POMATO, Easi3R:** DUSt3R-style pairwise extensions; require
  post-hoc optimization for global alignment.
- **St4RTrack:** feed-forward pairwise 4D; restricted to image pairs.
- **CasualSAM:** finetunes monocular depth on a single video.
- **Dynamic NeRFs / Gaussians:** per-scene test-time optimization;
  high quality but slow.

The 2025-2026 wave's contribution: drop per-scene optimization AND
support arbitrary view counts, in one network.

## Key design axes (emergent from the batch)

- **Output representation:** per-frame pointmaps vs. trajectory fields
  vs. base + relative motion.
- **Inference flexibility:** all-at-once batch (V-DPM, Any4D) vs.
  conditional query (4RC).
- **Metric scale:** Any4D / MapAnything yes; most others no.
- **Multi-modal:** Any4D yes (RGB-D, IMU, Doppler); [[stride]] yes
  (camera + LiDAR, fused in 3D point space); others image-only.
- **Scene decomposition:** only [[stride]] decomposes the scene into
  per-instance dynamic Gaussians (motion-driven, label-free); others
  emit unstructured point/Gaussian/trajectory sets.
- **Online vs. batch:** most of this batch is batch. [[fsm]] is
  architecturally streaming-capable (LaCET state carries across chunks)
  but only evaluated in batch mode. CUT3R does online 3D + handles
  dynamic scenes, but no explicit motion representation.
- **Attention complexity:** V-DPM / Any4D / 4RC use O(n²) full
  attention; [[fsm]] achieves O(n) via [[test-time-training|TTT]] state.
  This is a new axis as of this source.
- **Sparse vs. dense motion:** [[trace-anything]] + [[any4d]] are dense;
  [[tapip3d]] + [[spatialtracker-v2]] are sparse trackers in the same
  problem space.

## Evidence across sources

- All 4 source papers ([[any4d]], [[v-dpm]], [[4rc]], [[trace-anything]])
  claim SOTA on overlapping but non-identical 4D benchmarks.
  Direct head-to-head comparisons exist in [[4rc]]'s related-work section
  (critical of the other three) but not as standardized eval numbers.
- [[any4d]]: 2–3× lower error / 15× faster than next-best.
- [[v-dpm]]: more than halves error vs DPM / MonST3R / St4RTrack
  (older baselines).
- [[4rc]]: beats prior + concurrent on camera pose / video depth /
  pointcloud / 3D tracking / dense motion.
- [[trace-anything]]: SOTA on its own new benchmark; competitive on
  TAP-Vid.

## Open questions

- **Unified eval protocol** — head-to-head Any4D vs V-DPM vs 4RC vs
  Trace Anything on identical splits + identical input conditions
  (mono, RGB-D, etc.).
- **Online / streaming 4D** — all current batch methods are offline.
  Combining CUT3R's recurrence with 4D outputs is an obvious gap.
- **Beyond geometry+motion:** can these representations encode object
  identity, material, illumination jointly? Currently they're geometry-
  only.
- **Real-world generalization** — most evals on synthetic or curated
  benchmarks. Robotics deployment data is sparse.
- See [[q-tracking-vs-4d-reconstruction]] for the broader convergence
  thread.
