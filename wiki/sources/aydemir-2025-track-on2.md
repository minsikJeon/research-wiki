---
type: source
source_type: paper
title: "Track-On2: Enhancing Online Point Tracking with Memory"
authors:
  - Aydemir, Görkay
  - Xie, Weidi
  - Güney, Fatma
year: 2025
venue: arXiv (cs.CV) — IEEE TPAMI submission
url: https://arxiv.org/abs/2509.19115
raw_path: papers/Aydemir et al. - 2025 - Track-On2 Enhancing Online Point Tracking with Memory.pdf
status: ingested
tags: [point-tracking, online-tracking, transformer, memory, classification-first, synthetic-training]
sources: []
related:
  - "[[track-on2]]"
  - "[[online-vs-offline-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[point-tracking]]"
  - "[[gorkay-aydemir]]"
created: 2026-05-24
updated: 2026-05-24
---

# Track-On2: Enhancing Online Point Tracking with Memory

## TL;DR

Track-On2 extends the ICLR'25 Track-On line: a **strictly causal,
frame-by-frame** transformer tracker with a **single expandable memory** and
a **classification-first** localization head (coarse patch classification →
refinement). Despite **synthetic-only training**, it matches or beats both
online and offline competitors (CoTracker3, BootsTAPNext, TAPTRv3) on
DAVIS / Kinetics / RoboTAP / Dynamic Replica / PointOdyssey. >30 FPS with
flat memory cost in video length.

## Why it matters

Two important empirical claims that recur across the batch:

1. **Synthetic training alone is enough** if the architecture and training
   recipe are right — no real-data fine-tuning needed to reach SOTA.
   Direct counterpoint to BootsTAPIR / CoTracker3 / BootsTAPNext, which
   all rely on real-data pseudo-labels. Reopens the [[synthetic-to-real-gap]]
   debate.
2. **Training video length is the dominant factor** governing long-horizon
   memory behavior — more than memory capacity or sampling strategy. This
   is a clean, actionable finding for anyone training causal trackers.

Also reaffirms the **classification-first** approach to coordinate
prediction (compare TAPNext's classification head — convergent design).

## Key claims

- **Single expandable memory** replaces Track-On's dual memory module,
  with inference-time memory extension (IME) so the model scales past
  training-time sequence lengths without retraining.
- **Multi-scale features from ViT-Adapter** replace pooled single-scale
  maps. Modest accuracy gain, larger speed gain.
- **Training clip length dominates** long-horizon generalization; frame
  sampling strategy and memory capacity are complementary but secondary.
  Switching to longer training clips alone delivers most of the gain.
- **Classification → re-ranking → sub-patch refinement** localization
  pipeline. Top-k uncertainty + top-k score losses.
- DINOv3 backbone (vs DINOv2) gives further gains: +5.6 AJ on RoboTAP
  vs CoTracker3 (Window), +4.8 vs CoTracker3 (Video).
- Linear memory cost in sequence length (Fig 3) — video-level methods
  scale poorly and OOM on PointOdyssey.

## Methods

- Visual encoder: DINOv2 or DINOv3 ViT (frozen), patch tokens.
- Queries = the points to track, decoded against per-frame features and
  the memory module.
- Memory: dynamic, expandable; carries contextual cues across frames.
- Causal per-frame inference: outputs immediately on each new frame; no
  future-frame access.
- Trained only on synthetic data (TAP-Vid-Kubric and extensions).

## Results (headline)

- **TAP-Vid DAVIS** (queried-first): outperforms TAPTRv3 in AJ + OA,
  beats TAPNext-B in δ_avg.
- **RoboTAP** (274-frame robotics seqs): +5.6 AJ vs CoTracker3-Window,
  +4.8 vs CoTracker3-Video. Strongest gains here — robotics-relevant.
- **PointOdyssey** (very long, ~2.4K frames avg): video/window methods OOM;
  Track-On2 sustains performance.
- **DAVIS short**: top across online + offline despite synthetic-only train.

## Limitations / open questions

- Frozen backbone — gains from DINOv3 imply more is achievable with
  finer-tuned features but at training-cost.
- Architecture is the most "engineered" of this batch — memory module,
  re-ranker, sub-patch refinement; non-trivial to reproduce vs the
  more minimal [[tapnext]] family.
- Open: does the "training length is dominant" finding hold for the
  TAPNext-style SSM architecture? [[jung-2026-tapnext-plus-plus]] gives
  a partial answer (yes — long-sequence training fixes TAPNext's failure
  mode), but the two papers don't directly compare training-recipe deltas.

## Connections

- Method: [[track-on2]]. Predecessor: Track-On (ICLR'25) — not yet a page.
- Concepts: [[online-vs-offline-tracking]], [[synthetic-to-real-gap]].
- Authors: [[gorkay-aydemir]] (lead, KU). Weidi Xie + Fatma Güney are
  joint supervisors — not yet entity pages.
- Datasets: [[tap-vid-dataset]], [[pointodyssey-dataset]],
  [[dynamic-replica-dataset]], RoboTAP (defer until 3rd mention).
- Comparison entry: [[cmp-tap-methods]].
- Track-On2 is explicitly compared against [[tapnext]] in Fig 1 — the
  "online frame-by-frame tracker without real-data fine-tuning" niche.

## Citation

Aydemir, G., Xie, W., & Güney, F. (2025). *Track-On2: Enhancing Online
Point Tracking with Memory.* arXiv:2509.19115v2 [cs.CV] (submitted to
IEEE TPAMI). https://arxiv.org/abs/2509.19115
