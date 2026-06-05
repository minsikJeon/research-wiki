---
type: method
title: Track-On2
status: stub
tags: [point-tracking, online-tracking, transformer, memory, classification-first, dinov3, synthetic-training]
sources:
  - "[[aydemir-2025-track-on2]]"
related:
  - "[[point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[synthetic-to-real-gap]]"
created: 2026-05-24
updated: 2026-05-24
---

# Track-On2

Causal, frame-by-frame transformer tracker with a **single expandable
memory module** and a **classification-first** localization pipeline.
Trained on synthetic data only; matches or beats real-data-fine-tuned
trackers across short and long benchmarks.

## One-line summary

Frozen DINOv2/v3 ViT → multi-scale features → memory-augmented decoder
where queries (= tracked points) attend to current-frame features +
expandable memory → coarse patch classification + re-ranking +
sub-patch offset regression. Causal per-frame; >30 FPS; flat memory
in video length.

## Inputs / outputs

- **In:** RGB video stream, query points `(t_init, x, y)` and memory
  state from previous frame.
- **Out:** per-query `(x, y)` and visibility for the current frame,
  updated memory state.

## How it works

1. **Visual encoder:** frozen DINOv2 (or DINOv3 in the stronger variant)
   ViT; multi-scale features via ViT-Adapter.
2. **Queries:** the points to track. Decoder cross-attends to current
   frame features + the expandable memory bank.
3. **Memory:** single expandable bank carrying context from previous
   frames. Inference-time memory extension (IME) lets the model exceed
   the sequence lengths seen in training.
4. **Localization (classification-first):**
   - Coarse patch-level classification (which patch contains the point?)
     via embedding similarity.
   - Re-ranking step suppresses distractor matches.
   - Sub-patch offset regression for sub-pixel precision.
5. **Losses:** top-k uncertainty loss + top-k score loss for patch
   classification; ℓ1 for offset regression; supervision on visibility.
6. **Training:** long synthetic clips. Authors emphasize **training clip
   length is the dominant factor** for long-horizon generalization;
   memory capacity / sampling strategy are secondary.

## Where it's been applied

- TAP-Vid DAVIS / Kinetics, RoboTAP, Dynamic Replica, PointOdyssey
  (the last is where memory-based methods stand out vs window-based).
- Robotics-relevant: RoboTAP (274-frame seqs) is where Track-On2's
  largest margins appear (+5.6 AJ vs CoTracker3-Window).

## Known limitations

- More engineered than the [[tapnext]] family (memory module +
  re-ranker + sub-patch head). Reproduction burden is higher.
- Frozen backbone — gains from DINOv3 hint at headroom from
  end-to-end fine-tuning at training cost.
- Trained synthetic-only — no proof yet that adding real-data fine-tune
  doesn't further improve.

## Related methods

- **Predecessor:** Track-On (ICLR 2025) — dual-memory predecessor.
- **Closest design philosophy:** [[tapnext]] / [[tapnext-plus-plus]] —
  both causal per-frame, both lean on classification coordinate heads.
  TAPNext uses an SSM for memory; Track-On2 uses an explicit memory bank.
- **Contrast on training:** synthetic-only here vs real-data fine-tuned
  CoTracker3 / BootsTAPNext.
