---
type: source
source_type: paper
title: "MapAnything: Universal Feed-Forward Metric 3D Reconstruction"
authors:
  - Keetha, Nikhil
  - Müller, Norman
  - Schönberger, Johannes
  - Porzi, Lorenzo
  - Zhang, Yuchen
  - Fischer, Tobias
  - Knapitsch, Arno
  - Zauss, Duncan
  - Weber, Ethan
  - Antunes, Nelson
  - Luiten, Jonathon
  - Lopez-Antequera, Manuel
  - Rota Bulò, Samuel
  - Richardt, Christian
  - Ramanan, Deva
  - Scherer, Sebastian
  - Kontschieder, Peter
year: 2025
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2509.13414
raw_path: papers/Keetha et al. - 2025 - MapAnything Universal Feed-Forward Metric 3D Reconstruction.pdf
status: ingested
tags: [3d-reconstruction, feed-forward, transformer, metric-scale, multi-modal, foundation-model, sfm]
sources: []
related:
  - "[[mapanything]]"
  - "[[vggt]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[nikhil-keetha]]"
  - "[[deva-ramanan]]"
  - "[[sebastian-scherer]]"
  - "[[meta-reality-labs]]"
  - "[[cmu-ri]]"
created: 2026-05-24
updated: 2026-05-24
---

# MapAnything: Universal Feed-Forward Metric 3D Reconstruction

## TL;DR

MapAnything is a **single feed-forward model** that solves **12+ different
3D reconstruction tasks** (unconstrained SfM, calibrated SfM, MVS, monocular
depth, camera localization, depth completion, ...) using a **factored
scene representation** (depth maps + local ray maps + camera poses +
metric scale factor). Matches or beats specialist feed-forward models on
each task. Meta Reality Labs + **CMU** ([[deva-ramanan]] + [[sebastian-scherer]]).

## Why it matters

The "universal 3D reconstruction backbone" claim is the key. Where
[[vggt]] is one general-purpose model that produces all 3D quantities
at once, MapAnything goes further: it can also **consume** geometric
inputs (calibration, poses, depth) when available, enabling the
same model to serve as both a from-scratch SfM solver AND a
calibrated-MVS / depth-completion model.

For robotics (the user's MSR context): robotic platforms often have
**partial** geometric information — known intrinsics, sometimes
extrinsics from IMU, sometimes depth from RGB-D. MapAnything's flexible
input scheme matches this real-world condition better than
VGGT's images-only interface.

## Key claims

- **Factored representation > coupled pointmaps.** Decompose into
  per-view depth + ray map + pose + global metric scale. Allows clean
  training on partial annotations (datasets with non-metric "up-to-scale"
  geometry contribute via the scale factor; datasets with depth-only
  contribute via the depth head; etc.).
- **Multi-modal input encoders.** DINOv2 for images + dedicated
  encoders for ray directions, pose (rotation/translation/scale), depth.
  Optional inputs are auto-handled.
- **Metric-scale output.** Single learnable scale token + scale head.
  Most prior feed-forward methods output up-to-scale.
- **12+ task configurations** from one model: uncalibrated SfM,
  calibrated SfM, posed SfM, MVS, RGB-D localization, depth completion,
  etc.
- **Outperforms or matches specialist models** for each task —
  consolidation is not lossy.
- Apache 2.0 release — code + weights.

## Methods

- **Encoder:** DINOv2 for image patches; dedicated encoders for ray maps,
  pose, depth (when provided). Patch features summed across modalities
  per view.
- **Transformer:** alternating-attention (per-frame and cross-frame),
  similar to VGGT. Reference view embedding added to view 1 features;
  single learnable scale token appended.
- **Heads:**
  - DPT head decodes per-view ray directions + ray depths + masks +
    confidences (local).
  - Average-pooling pose head decodes per-view poses in view-1 frame.
  - MLP on scale token → global metric scale factor.
- **Training:** large dataset aggregation with standardized supervision;
  flexible input augmentation (randomly drop modalities during training
  so the model learns to handle any subset at inference).

## Results (headline)

- Matches or beats specialist feed-forward models across all 12+
  configurations.
- Particularly competitive on **uncalibrated SfM** (where the closest
  competitor is VGGT, which lacks metric scale).
- Joint training across tasks is more compute-efficient than training
  per-task specialists.

## Limitations / open questions

- Static scenes only — dynamic content not addressed (see
  [[v-dpm]], [[any4d]], [[4rc]]).
- Comparison vs VGGT is "matches or surpasses" — not always a clear win;
  the value is unification + metric + multi-modal more than headline
  per-task gain.
- Training-data aggregation across 17+ datasets is non-trivial to
  reproduce.

## Connections

- Method: [[mapanything]].
- Concepts: [[feed-forward-3d-reconstruction]],
  [[pointmap-representation]].
- **Closest competitor:** [[vggt]] (less flexible inputs, no metric
  scale). [[depth-anything-3]] (strips further to depth+ray only).
- **Sister CMU paper:** [[any4d]] — same authors ([[nikhil-keetha]],
  [[deva-ramanan]], [[sebastian-scherer]]) but Any4D extends to 4D
  with motion + multi-modal sensors. MapAnything is the static counterpart.
- **Used or cited by:** [[trace-anything]] (lists MapAnything as a
  "one-pass over all frames" precedent), [[4rc]] (cites as feed-forward
  ancestor).
- Authors: [[nikhil-keetha]] (lead, [[cmu-ri]] + Meta), seniors
  [[deva-ramanan]] + [[sebastian-scherer]] (CMU).
- Org: [[meta-reality-labs]] + [[cmu-ri]] joint.

## Citation

Keetha, N., Müller, N., Schönberger, J., Porzi, L., Zhang, Y., Fischer, T.,
Knapitsch, A., Zauss, D., Weber, E., Antunes, N., Luiten, J., Lopez-
Antequera, M., Rota Bulò, S., Richardt, C., Ramanan, D., Scherer, S., &
Kontschieder, P. (2025). *MapAnything: Universal Feed-Forward Metric 3D
Reconstruction.* arXiv:2509.13414. https://arxiv.org/abs/2509.13414
