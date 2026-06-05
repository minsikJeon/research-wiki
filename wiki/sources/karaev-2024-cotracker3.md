---
type: source
source_type: paper
title: "CoTracker3: Simpler and Better Point Tracking by Pseudo-Labelling Real Videos"
authors:
  - Karaev, Nikita
  - Makarov, Iurii
  - Wang, Jianyuan
  - Neverova, Natalia
  - Vedaldi, Andrea
  - Rupprecht, Christian
year: 2024
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2410.11831
raw_path: papers/Karaev et al. - 2024 - CoTracker3 Simpler and Better Point Tracking by Pseudo-Labelling Real Videos.pdf
status: ingested
tags: [point-tracking, video-understanding, transformer, online-tracking, semi-supervised, pseudo-labeling]
sources:
  - "[[karaev-2024-cotracker]]"
related:
  - "[[cotracker3]]"
  - "[[cotracker]]"
  - "[[pseudo-labeling-point-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[point-tracking]]"
  - "[[tap-vid-dataset]]"
  - "[[nikita-karaev]]"
  - "[[meta-ai]]"
  - "[[oxford-vgg]]"
created: 2026-05-24
updated: 2026-05-24
---

# CoTracker3: Simpler and Better Point Tracking by Pseudo-Labelling Real Videos

## TL;DR

CoTracker3 does two things at once: (1) **simplifies** the architecture vs.
[[cotracker]] / BootsTAPIR / LocoTrack — strips global matching, keeps only
the components ablations actually justify; and (2) introduces a **simple
pseudo-labeling pipeline** where existing trackers act as teachers on
~100K real Internet videos. Beats BootsTAPIR with **1000× less real data
(15K vs 15M videos)** while being 27% faster than LocoTrack. Comes in
online and offline variants from the same code.

## Why it matters

The most important contribution is the **data-scaling study**: shows you
don't need BootsTAPIR's elaborate self-training recipe (15M videos,
EMA student/teacher, three loss masks, augmentations). A random-teacher
distillation with no augmentation, no EMA, no synthetic ground-truth
mix, and 0.1% of the data beats it. This is a strong argument for
**[[pseudo-labeling-point-tracking]]** as a general recipe and reframes
the [[synthetic-to-real-gap]] discussion.

## Key claims

- **Simpler = better:** removing the global-matching stage and other
  recent additions did not hurt — the iterative-updates + cross-track
  attention + 4D correlation core is sufficient.
- **1000× less data:** with 15K pseudo-labeled real videos, CoTracker3
  beats BootsTAPIR-trained-on-15M.
- **Teacher diversity > teacher quality:** randomly sampling among CoTracker,
  TAPIR, CoTracker3-online, CoTracker3-offline as the teacher each batch
  improves the student more than any single teacher. The student inherits
  complementary strengths (online sticks near query origin; offline
  handles occlusion).
- **Cross-track attention** is responsible for most of the occluded-point
  gain (+5.1 occluded AJ on Dynamic Replica), confirming the [[cotracker]]
  hypothesis.
- **Online + offline from one recipe:** training the same model with both
  sliding-window and single-window protocols gives both variants without
  separate codebases.

## Methods

- Architecture borrows: PIPs's iterative updates + conv features,
  CoTracker's cross-track attention + virtual/proxy tokens + unrolled
  training, LocoTrack's 4D correlation. Drops: BootsTAPIR/LocoTrack
  global matching.
- Pseudo-label pipeline: 100K Internet videos, ~30s each. For each batch:
  pick a random teacher → sample query points via SIFT (favoring
  "good-to-track" Shi-Tomasi corners) on random keyframes → run teacher
  → train student on the pseudo-labels using the same loss as on
  synthetic data (visibility/confidence supervision skipped on real
  to avoid catastrophic forgetting; handled by a separate linear head).
- No augmentation, no EMA, no synthetic-data mix during fine-tune.

## Results (headline)

- **TAP-Vid DAVIS / Kinetics / RGB-Stacking + Dynamic Replica + RoboTAP**:
  CoTracker3 (Kubric+15K) beats BootsTAPIR (Kubric+15M) on most metrics.
- Online variant on TAP-Vid is competitive with offline; CoTracker3-online
  is better than LocoTrack on occluded points where joint tracking pays off.
- Self-training on its own predictions (no other teachers) still
  gives +1.2 AJ — suggesting the data-distribution shift toward real
  videos matters even with weak labels.

## Limitations / open questions

- Still uses sliding-window inference — not strictly per-frame causal.
  (Compare to [[tapnext]] / [[track-on2]] / [[tapnext-plus-plus]].)
- Doesn't solve long-sequence re-detection — points lost across long
  occlusions don't recover (this is the gap [[tapnext-plus-plus]] targets).
- The teacher pool itself is bounded: as all teachers were trained on
  Kubric, errors common to Kubric-trained models can become baked into
  the student.

## Connections

- Method: [[cotracker3]]. Predecessor: [[cotracker]].
- Concepts: [[pseudo-labeling-point-tracking]], [[synthetic-to-real-gap]].
- BootsTAP is the contrast point — same goal, much heavier machinery.
  Promote BootsTAP to its own method page on 3rd mention (already noted
  in CoTracker3 + TAPNext + TAPNext++).
- Datasets used: [[tap-vid-dataset]], [[dynamic-replica-dataset]],
  RoboTAP (entity page deferred — 2 mentions so far),
  [[pointodyssey-dataset]] (implicit via teacher set).

## Citation

Karaev, N., Makarov, I., Wang, J., Neverova, N., Vedaldi, A., & Rupprecht, C.
(2024). *CoTracker3: Simpler and Better Point Tracking by Pseudo-Labelling
Real Videos.* arXiv:2410.11831v1. https://arxiv.org/abs/2410.11831
