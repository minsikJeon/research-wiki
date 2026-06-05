---
type: method
title: TAPNext++
status: stub
tags: [point-tracking, state-space-model, vit, online-tracking, long-sequence, re-detection]
sources:
  - "[[jung-2026-tapnext-plus-plus]]"
related:
  - "[[tapnext]]"
  - "[[point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAPNext++

Direct successor to [[tapnext]]. **Same architecture**; new training
recipe (1024-frame Kubric-1024 + sequence-parallel SSM scan + periodic
roll augmentation + occluded-point supervision). Sets SOTA on long-sequence
and re-detection benchmarks without real-data fine-tuning.

## One-line summary

TAPNext architecture, trained on 1024-frame sequences via distributed
parallel scan across GPUs, plus roll augmentation for re-entry robustness.
Introduces the **AJRD (Re-Detection Average Jaccard)** metric.

## Inputs / outputs

Identical to [[tapnext]]: video + query points → causal per-frame
`(x, y)` + visibility for each query, at 348 FPS @ 1024 points on H100.

## How it works

1. **Architecture: unchanged from [[tapnext]].** Interleaved RecurrentGemma
   SSM (time) + ViT (space) blocks. Masked-token decoding. Classification
   coordinate head. No tracking-specific inductive biases.
2. **Kubric-1024 dataset:** extends Kubric to 1024-frame sequences with
   diverse trajectory generation (random start/end on spherical shells +
   sinusoidal noise + ground-plane projection).
3. **Distributed parallel scan:** training-time technique that splits the
   SSM scan across GPUs. Inter-GPU communication only between SSM blocks
   (not ViT blocks); each GPU runs ViT layers locally, then a parallel
   prefix scan merges hidden states across chunk boundaries.
4. **Periodic roll augmentation:** image is cyclically shifted during
   training so points naturally exit one side and re-enter the other —
   the simplest possible re-entry simulator. Ablations show this is
   essential for re-detection robustness.
5. **Occluded-point supervision:** prior methods often skipped these in
   the loss; TAPNext++ supervises them too.

## Where it's been applied

- TAP-Vid DAVIS, RoboTAP, PointOdyssey — all online metrics.
- Speed/accuracy Pareto frontier in online tracking (Fig 1).

## Known limitations

- Still 2D-only. For 3D, see [[tapip3d]] / [[spatialtracker-v2]].
- Distributed parallel scan adds non-trivial training-infra complexity.
- AJRD metric is new — no community-wide baseline yet.
- Roll augmentation is 2D image-plane; doesn't generalize to 3D
  augmentation strategies.

## Related methods

- **Predecessor:** [[tapnext]] — same architecture; differences are
  purely in training data + augmentation + loss.
- **Strictly causal competitors:** [[track-on2]] (memory-based), original
  [[tapnext]] (SSM-only, short-sequence trained).
- **Pseudo-labeling alternative:** [[cotracker3]] (real-data fine-tune,
  windowed). TAPNext++ argues synthetic + better training beats this for
  long sequences.
