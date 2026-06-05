---
type: concept
title: Pseudo-Labeling for Point Tracking
status: growing
tags: [point-tracking, pseudo-labeling, semi-supervised, distillation, sim2real]
sources:
  - "[[karaev-2024-cotracker3]]"
related:
  - "[[point-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[cotracker3]]"
created: 2026-05-24
updated: 2026-05-24
---

# Pseudo-Labeling for Point Tracking

## The recipe (general form)

1. Train one or more **teacher** trackers on synthetic data (Kubric).
2. Run teachers on a pool of **unlabeled real videos** to produce
   per-track pseudo-labels.
3. Train a **student** tracker on those pseudo-labels (often a copy of
   the teacher architecture, or a new design).
4. Optionally: keep synthetic ground-truth in the mix; apply augmentations;
   use EMA / consistency losses.

The student often outperforms each individual teacher because of
ensemble + distribution-shift + voting effects.

## Recipe variants observed in this wiki

**Heavy (BootsTAP / BootsTAPIR / BootsTAPNext family — Doersch et al.):**
- 15M unlabeled real clips.
- Student–teacher EMA.
- Three loss masks.
- Augmentation equivariance (student sees affine-transformed + JPEG-
  corrupted; teacher sees clean).
- Synthetic ground-truth retained to prevent forgetting.

**Light (CoTracker3 — Karaev et al., [[karaev-2024-cotracker3]]):**
- 100K Internet videos (~30 sec each); 15K used for the main result.
- Random teacher per batch from {CoTracker, TAPIR, CoTracker3-online,
  CoTracker3-offline}.
- SIFT-based query sampling (Shi-Tomasi "good-to-track" corners).
- No augmentation, no EMA, no synthetic ground-truth mix.
- Visibility / confidence supervised separately (avoid forgetting).
- **Beats BootsTAPIR with 1000× less data.**

## Key empirical findings

- **Teacher diversity > teacher quality** ([[karaev-2024-cotracker3]],
  Table 5): random sampling among complementary teachers beats any
  single teacher.
- **Self-training works** even with a single teacher (the model on its
  own predictions): +1.2 AJ improvement, suggesting the data shift
  toward real videos matters even with weak labels.
- **Light recipes are not strictly worse than heavy ones.** The win for
  BootsTAP comes from data scale; the win for CoTracker3 comes from
  teacher diversity. Both are valid.

## Counterpoint: synthetic-only camp

[[aydemir-2025-track-on2]] and [[jung-2026-tapnext-plus-plus]] argue
pseudo-labeling isn't necessary at all if the synthetic training recipe
(length, augmentation) is good enough. See [[synthetic-to-real-gap]]
for the broader debate.

## Open questions

- How does pseudo-labeling compose with 3D tracking ([[tapip3d]],
  [[spatialtracker-v2]])? ST-V2's multi-dataset training is the closest
  precedent.
- What's the failure mode of pseudo-labeled trackers on out-of-domain
  real video (e.g. underwater, microscopic, medical)?
- Can foundation-model trackers (DINOv3 features in [[track-on2]]) replace
  the need for real-data fine-tuning entirely?
