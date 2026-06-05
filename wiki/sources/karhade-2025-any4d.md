---
type: source
source_type: paper
title: "Any4D: Unified Feed-Forward Metric 4D Reconstruction"
authors:
  - Karhade, Jay
  - Keetha, Nikhil
  - Zhang, Yuchen
  - Gupta, Tanisha
  - Sharma, Akash
  - Scherer, Sebastian
  - Ramanan, Deva
year: 2025
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2512.10935
raw_path: papers/Karhade et al. - 2025 - Any4D Unified Feed-Forward Metric 4D Reconstruction.pdf
status: ingested
tags: [4d-reconstruction, feed-forward, transformer, metric-scale, multi-modal, scene-flow, dense-tracking, sfm, robotics]
sources:
  - "[[keetha-2025-mapanything]]"
related:
  - "[[any4d]]"
  - "[[mapanything]]"
  - "[[4d-reconstruction]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[jay-karhade]]"
  - "[[nikhil-keetha]]"
  - "[[deva-ramanan]]"
  - "[[sebastian-scherer]]"
  - "[[cmu-ri]]"
created: 2026-05-24
updated: 2026-05-24
---

# Any4D: Unified Feed-Forward Metric 4D Reconstruction

## TL;DR

Any4D is the **4D sibling of [[mapanything]]**: a feed-forward
transformer that produces **dense metric-scale 4D reconstructions**
(geometry + scene flow + camera poses) from N video frames in a single
pass. **Multi-modal inputs** (RGB, RGB-D, IMU, Radar Doppler) when
available. **15× faster + 2-3× lower error** than prior SOTA 4D methods.
**CMU** — same authors as MapAnything ([[nikhil-keetha]], [[deva-ramanan]],
[[sebastian-scherer]]) plus [[jay-karhade]] as lead. Direct robotics-relevant
paper.

## Why it matters

The robotics-deployment story:

- Real robots have **diverse sensor configurations** (camera + IMU + Radar
  + Lidar) that prior 4D methods ignore. Any4D natively consumes them.
- **Metric scale is required** for any physical-world deployment.
  Most prior 4D methods output up-to-scale.
- **Feed-forward + single pass** = real-time deployable; prior 4D
  methods like MegaSaM need per-scene optimization.

For the user's MSR context: this is plausibly **the most directly
applicable paper** in the wiki for robot perception tasks where you
need both geometry and motion in one shot.

## Key claims

- **Factored 4D representation:**
  - **Allocentric (world-frame):** scene flow (3D per-pixel motion from
    view 1) + camera poses (per-view in view 1's frame).
  - **Egocentric (local-camera-frame):** ray directions + scale-normalized
    depth.
  - **Global metric scale factor** that upgrades the local quantities to
    metric.
  This factoring lets diverse training datasets contribute partial
  supervision (metric-no-motion, non-metric-with-motion, etc.).
- **Multi-modal inputs** improve reconstruction when available:
  RGB-D, IMU poses, Doppler velocity.
- **Dense (per-pixel) 3D point tracking** as a byproduct, not sparse
  trajectories. Differentiates from prior 3D point trackers
  ([[tapip3d]], [[spatialtracker-v2]]) which output sparse tracks.
- **Allocentric vs egocentric tracking:** Any4D works in world
  coordinates, like [[tapip3d]]. Other 3D trackers (SpatialTracker,
  DELTA) work egocentric.
- **15× faster than next-best 4D method**; **2-3× lower error**.
- Single feed-forward pass over all N frames (contrast: prior methods
  do pairwise + optimization).

## Methods

- **Architecture:** scalable multi-view transformer (extends MapAnything
  to 4D). Alternating attention over view tokens; multi-modal encoders
  for optional sensor inputs.
- **Outputs** (per view i, with `~` denoting prediction):
  - `R̃_i` — ray directions (3×H×W).
  - `D̃_i` — scale-normalized depth.
  - `T̃_i` — camera pose in view 1's frame (translation + quaternion).
  - `F̃_i` — scale-normalized forward scene flow from view 1 to view i.
  - `s̃` — global metric scaling factor.
- **Recovery:** metric pointmap `G̃_i = s̃ · T̃_i · R̃_i · D̃_i`;
  motion-applied pointmap `G̃'_i = G̃_i + s̃ · F̃_i`.
- **Training:** mix of metric-scale 3D datasets (no motion) + non-metric
  datasets (with motion). Factoring enables training on partial labels.

## Results (headline)

- **TAPVid-3D:** competitive with [[tapip3d]] and [[spatialtracker-v2]]
  despite producing **dense** rather than sparse tracks.
- **15× faster** than next-best 4D reconstruction method.
- **2-3× lower error** on combined 4D metrics.
- Multi-modal inputs (RGB-D, IMU, Doppler) further improve over RGB-only.

## Limitations / open questions

- Concurrent / contemporary with [[4rc]], [[v-dpm]], [[trace-anything]] —
  all making overlapping claims about feed-forward 4D. No unified eval
  protocol yet.
- Scene flow is "first-frame-to-other-frames" — symmetric long-range
  trajectories may need additional handling.
- Multi-modal datasets remain scarce; most ablations show RGB-only
  baseline.

## Connections

- Method: [[any4d]].
- Concepts: [[4d-reconstruction]], [[feed-forward-3d-reconstruction]].
- **Sister CMU paper:** [[mapanything]] (static 3D counterpart, same
  authors and architecture lineage). Any4D = MapAnything + time + motion.
- **Concurrent / competing 4D feed-forward:** [[v-dpm]] (VGG Oxford,
  extends VGGT with DPM heads), [[4rc]] (NTU+Oxford, conditional query
  decoder), [[trace-anything]] (anonymous, trajectory fields).
- **3D tracking competitors:** [[tapip3d]] (sparse, ego-centric option),
  [[spatialtracker-v2]] (sparse, end-to-end).
- **Useful for:** [[3d-point-tracking]] via dense scene flow.
- Authors: [[jay-karhade]] (lead), [[nikhil-keetha]] (MapAnything lead +
  Any4D co-author), seniors [[deva-ramanan]] + [[sebastian-scherer]].
- Org: [[cmu-ri]].

## Citation

Karhade, J., Keetha, N., Zhang, Y., Gupta, T., Sharma, A., Scherer, S., &
Ramanan, D. (2025). *Any4D: Unified Feed-Forward Metric 4D Reconstruction.*
arXiv:2512.10935. https://arxiv.org/abs/2512.10935
