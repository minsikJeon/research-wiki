---
type: source
source_type: paper
title: "TAPNext++: What's Next for Tracking Any Point (TAP)?"
authors:
  - Jung, Sebastian
  - Zholus, Artem
  - Sundermeyer, Martin
  - Doersch, Carl
  - Goroshin, Ross
  - Tan, David Joseph
  - Chandar, Sarath
  - Triebel, Rudolph
  - Tombari, Federico
year: 2026
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2604.10582
raw_path: papers/Jung et al. - 2026 - TAPNext++ What's Next for Tracking Any Point (TAP).pdf
status: ingested
tags: [point-tracking, online-tracking, state-space-model, long-sequence, re-detection, augmentation]
sources:
  - "[[zholus-2025-tapnext]]"
related:
  - "[[tapnext-plus-plus]]"
  - "[[tapnext]]"
  - "[[point-tracking]]"
  - "[[artem-zholus]]"
  - "[[carl-doersch]]"
  - "[[google-deepmind]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAPNext++: What's Next for Tracking Any Point (TAP)?

## TL;DR

TAPNext++ is the direct follow-up to [[zholus-2025-tapnext]]. **Same
architecture** (RecurrentGemma SSM + ViT, no tracking-specific inductive
biases), **new training recipe**: trains on **1024-frame Kubric-1024**
sequences via distributed parallel scan, adds **periodic roll**
augmentation for re-entry robustness, and supervises occluded points.
Sets new SOTA on online tracking across multiple long-sequence benchmarks
*without* real-data fine-tuning, and introduces a new **AJRD
(Re-Detection Average Jaccard)** metric. Tracks 1024 points at 348 FPS
on H100.

## Why it matters

- Settles the open question flagged in the TAPNext ingest: yes, the
  long-video failure was a **training-data length issue**, not an
  inherent SSM limitation. Long-sequence training fixes it.
- Identifies **re-detection** (recovering a point after long occlusion
  or off-frame) as a blind spot of all current TAP metrics and proposes
  AJRD to measure it explicitly.
- Demonstrates that a **purely synthetic training pipeline** can beat
  real-data-fine-tuned BootsTAPNext on long sequences. Aligns with
  [[aydemir-2025-track-on2]]'s "synthetic alone is enough" claim — two
  independent confirmations now.
- Engineering contribution: **distributed parallel scan** for SSM training
  enables 1024-frame sequences across GPUs — a transferable trick for any
  recurrent-transformer training.

## Key claims

- **Long-sequence training is the fix.** Kubric-1024 (1024-frame variant)
  is the dominant factor. The architecture is unchanged from TAPNext.
- **Periodic roll augmentation** simulates point re-entries during
  training; without it, even TAPNext++ struggles with re-detection
  (qualitative Fig 2: "TAPNext++ w/o Roll" fails the juggling test).
- **Supervising occluded points** is a free improvement — prior methods
  often skipped these in the loss.
- **AJRD metric** (Re-Detection AJ): varies the minimum-undetectable-frame
  window `dmin` to specifically score recovery after occlusion. Existing
  AJ + survival-rate metrics miss this.
- 348 FPS @ 1024 points on H100; flat memory in sequence length thanks
  to SSM linearity.
- Beats BootsTAPNext (real-data fine-tuned) and Track-On on PointOdyssey
  and RoboTAP AJRD.

## Methods

- Architecture: unchanged from [[tapnext]] — interleaved RecurrentGemma
  SSM (time) + ViT (space), masked-token decoding, classification
  coordinate head. Authors explicitly emphasize **no new architecture
  needed**.
- **Distributed parallel scan** (Fig 3): SSM hidden states are merged
  across GPUs during training, enabling 1024-frame back-prop on multi-GPU
  setups. Inter-GPU comms only between SSM blocks, not ViT blocks.
- Kubric-1024 dataset: Kubric extension to 1024-frame sequences.
- Trajectory generation includes random start/end positions sampled
  on a spherical shell + sinusoidal noise + ground-plane projection,
  for diverse motion.
- Periodic roll: image is cyclically shifted during training, making
  points naturally exit one side and re-enter the other — simulating
  the off-frame-then-back scenarios that break re-detection.

## Results (headline)

- **PointOdyssey** (~2400 frames): TAPNext++ tops AJRD; BootsTAPNext +
  Track-On + CoTracker3 all degrade as `dmin` grows; TAPNext++ stays flat.
- **RoboTAP** (~274 frames): SOTA AJRD; clock-hand-tracking qualitative
  comparison (Fig 7) shows the originals fail full-rotation tracking.
- **DAVIS** (~68 frames): comparable or better than prior SOTA at higher
  FPS (Fig 1 speed-accuracy trade-off frontier).
- Compute: 348 FPS @ 1024 pts on H100.

## Limitations / open questions

- Still 2D-only — doesn't address 3D tracking ([[tapip3d]],
  [[spatialtracker-v2]] are the answers in that direction).
- Distributed parallel scan adds infra complexity that smaller labs may
  struggle with — the "no architectural changes" claim is true but
  training-infra changes are non-trivial.
- AJRD is new — no community baseline yet on its exact form. Other groups
  will need to evaluate to confirm relative ranking.
- The roll augmentation is specifically a 2D-image cyclic shift; how it
  composes with 3D-aware augmentation is unexplored.

## Connections

- Method: [[tapnext-plus-plus]]. Predecessor: [[tapnext]] (architecture
  unchanged; only training/data/augmentation differ).
- Concept: [[synthetic-to-real-gap]] — adds evidence for "synthetic only
  is enough."
- Authors: [[artem-zholus]] (co-first, also TAPNext lead),
  [[carl-doersch]] (DeepMind TAP-series). Sebastian Jung co-first
  (defer to entity until 2nd paper).
- Org: [[google-deepmind]] + Google + DLR + KIT + Mila + UdeM +
  Polytechnique Montréal + TUM (the academic-industry collab pattern
  recurring in the TAP-series).
- Comparison: [[cmp-tap-methods]].

## Citation

Jung, S., Zholus, A., Sundermeyer, M., Doersch, C., Goroshin, R., Tan, D. J.,
Chandar, S., Triebel, R., & Tombari, F. (2026). *TAPNext++: What's Next for
Tracking Any Point (TAP)?* arXiv:2604.10582v1.
https://arxiv.org/abs/2604.10582
