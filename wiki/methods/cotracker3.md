---
type: method
title: CoTracker3
status: stub
tags: [point-tracking, transformer, joint-tracking, online-tracking, offline-tracking, pseudo-labeling]
sources:
  - "[[karaev-2024-cotracker3]]"
related:
  - "[[cotracker]]"
  - "[[pseudo-labeling-point-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# CoTracker3

A simplified successor to [[cotracker]] paired with a **lightweight
pseudo-labeling pipeline** that achieves SOTA with 1000× less real
training data than BootsTAPIR. Single recipe yields both online and
offline variants.

## One-line summary

Strip CoTracker / BootsTAPIR / LocoTrack to its load-bearing pieces
(iterative updates, cross-track attention, 4D correlation, virtual
tracks), then fine-tune on ~15K Internet videos pseudo-labeled by a
**random ensemble of existing trackers as teachers**.

## Inputs / outputs

Same interface as [[cotracker]]: video + query points → per-track
`(x, y)` + visibility + confidence.

## How it works

1. **Architecture (simplified):**
   - PIPs-style iterative updates + conv features.
   - CoTracker-style cross-track attention + virtual / proxy tokens +
     unrolled-window training.
   - LocoTrack-style 4D correlation.
   - **Removed:** global matching stage (from BootsTAPIR / LocoTrack).
2. **Pseudo-label training pipeline:**
   - Dataset: ~100K Internet videos, ~30s each.
   - For each batch, pick **one random teacher** from
     {CoTracker3-online, CoTracker3-offline, CoTracker, TAPIR}.
   - Sample query points via SIFT keypoints (biasing toward Shi-Tomasi
     "good-to-track" corners) on random keyframes.
   - Train student with the same loss as on synthetic data.
   - **No augmentation, no EMA, no synthetic ground-truth mix.**
   - Visibility / confidence supervised via separate linear heads (to
     avoid forgetting their predictions).
3. **Online + offline from one recipe:** train both protocols
   simultaneously in the same model.

## Where it's been applied

- TAP-Vid DAVIS / Kinetics / RGB-Stacking, Dynamic Replica, RoboTAP.
- Used as a baseline / teacher in many follow-on papers
  ([[track-on2]], [[tapnext-plus-plus]], [[spatialtracker-v2]]).

## Known limitations

- Still windowed, not strictly per-frame causal.
- Doesn't solve long-occlusion re-detection — points lost across long
  occlusions stay lost.
- Teachers are all Kubric-trained → systematic errors common to that
  data can transfer to the student.

## Related methods

- **Predecessor:** [[cotracker]].
- **Contemporary "simpler is enough" / "synthetic enough" claim:**
  [[track-on2]] (synthetic-only) and [[tapnext-plus-plus]] (long-sequence
  training fixes TAPNext without real-data).
- **Comparison point on pseudo-labeling:** BootsTAPIR / BootsTAP family
  (heavier recipe, 15M videos). Promote to its own method page on
  3rd source mention.
