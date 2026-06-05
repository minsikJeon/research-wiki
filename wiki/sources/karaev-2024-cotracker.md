---
type: source
source_type: paper
title: "CoTracker: It is Better to Track Together"
authors:
  - Karaev, Nikita
  - Rocco, Ignacio
  - Graham, Benjamin
  - Neverova, Natalia
  - Vedaldi, Andrea
  - Rupprecht, Christian
year: 2024
venue: ECCV 2024
url: https://arxiv.org/abs/2307.07635
raw_path: papers/Karaev et al. - 2024 - CoTracker It is Better to Track Together.pdf
status: ingested
tags: [point-tracking, video-understanding, transformer, online-tracking, joint-tracking]
sources: []
related:
  - "[[cotracker]]"
  - "[[joint-point-tracking]]"
  - "[[point-tracking]]"
  - "[[tap-vid-dataset]]"
  - "[[pointodyssey-dataset]]"
  - "[[dynamic-replica-dataset]]"
  - "[[nikita-karaev]]"
  - "[[meta-ai]]"
  - "[[oxford-vgg]]"
created: 2026-05-24
updated: 2026-05-24
---

# CoTracker: It is Better to Track Together

## TL;DR

CoTracker reframes [[point-tracking]] as a **joint** estimation problem
instead of N independent per-point predictions: a sliding-window transformer
attends across both time and tracks, exploiting statistical dependence
between points that move together. Two key engineering moves —
**proxy tokens** for attention efficiency, and **unrolled-window training**
mimicking RNN BPTT — let it jointly track up to 70K points on a single GPU
and ride through long occlusions.

## Why it matters

This is the paper that introduced **[[joint-point-tracking]]** as a distinct
sub-paradigm — every subsequent SOTA tracker has had to position itself
relative to "tracks share information" or "tracks are independent."
CoTracker3, BootsTAPIR, TAPNext, Track-On2 all cite this work.
Cross-track attention turns out to be the single largest contributor to
**occluded-point performance** (+5.1 occluded vs. +1.6 visible AJ delta on
Dynamic Replica, ablation Table 3).

## Key claims

- **Joint > independent tracking** for occluded and grouped points
  (background dragged by foreground motion is fixed by joint tracking;
  Fig 2).
- **Support points improve target tracking.** Adding extra queried points
  the user didn't ask for *as context* increases accuracy on the points
  they did ask for.
- **Proxy tokens** make attention scale to ~70K tracks on a single GPU by
  replacing track-track self-attention with track↔proxy cross-attention.
- **Unrolled-window training** (porting RNN training to point tracking)
  lets the model maintain identity across occlusions far longer than a
  single window.
- Sliding-window online operation: each window inherits refined estimates
  from the previous, so the tracker is causal in practice.

## Methods

- 2D token grid `[T_window, N_tracks]`; transformer with three attention
  patterns: time, tracks, time+tracks (via proxies).
- Standard PIPs-style **4D cost volumes** and iterative updates per window.
- Trained only on TAP-Vid-Kubric (synthetic). Window size and overlap are
  hyperparameters; unrolled training back-props across overlapping windows.

## Results (headline)

- SOTA on TAP-Vid DAVIS, RGB-Stacking, PointOdyssey, Dynamic Replica at
  publication time. Especially strong on occluded-point metrics.
- Scales to dense / quasi-dense grids — 70K tracks at once.

## Limitations / open questions

- Synthetic-only training → noticeable sim-to-real gap (later addressed
  by CoTracker3 [[karaev-2024-cotracker3]]).
- Windowed inference still has finite latency (window length frames
  before first output).
- The "joint tracking helps occlusion" effect depends on having enough
  visible neighbors of the occluded point — long, isolated occlusions
  remain hard.

## Connections

- Method: [[cotracker]]. Successor: [[cotracker3]] (same authors,
  simplified arch + real-data pseudo-labels).
- Concept introduced/championed: [[joint-point-tracking]].
- Tracking framing axis: window-based, see [[online-vs-offline-tracking]].
- Authors: [[nikita-karaev]] (lead), [[andrea-vedaldi]] (senior — entity
  page not yet created), [[christian-rupprecht]] (senior — entity not
  yet created).
- Orgs: [[meta-ai]] and [[oxford-vgg]] (joint affiliation).
- Datasets used: [[tap-vid-dataset]] (Kubric train + DAVIS/Kinetics eval),
  [[pointodyssey-dataset]], [[dynamic-replica-dataset]].
- Future seeds to promote on 2nd mention: TAPIR, PIPs, PIPs++, TAP-Net,
  OmniMotion, MFT, RoboTAP (eval), Particle Video, RAFT, GMFlow.

## Citation

Karaev, N., Rocco, I., Graham, B., Neverova, N., Vedaldi, A., & Rupprecht, C.
(2024). *CoTracker: It is Better to Track Together.* ECCV 2024.
arXiv:2307.07635v3. https://arxiv.org/abs/2307.07635
