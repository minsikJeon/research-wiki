---
type: method
title: CoTracker
status: stub
tags: [point-tracking, transformer, joint-tracking, online-tracking, window-based]
sources:
  - "[[karaev-2024-cotracker]]"
related:
  - "[[point-tracking]]"
  - "[[joint-point-tracking]]"
  - "[[cotracker3]]"
created: 2026-05-24
updated: 2026-05-24
---

# CoTracker

A sliding-window transformer for [[point-tracking]] that tracks **many
points jointly** rather than one-at-a-time, exploiting their statistical
dependence to improve tracking through occlusion.

## One-line summary

Build a 2D `[T_window, N_tracks]` token grid; apply transformer attention
along time, tracks, and (via proxy tokens) across both; run as a sliding
window with overlap; train with unrolled-window BPTT to maintain long
tracks.

## Inputs / outputs

- **In:** RGB video, set of query points `(t, x, y)`.
- **Out:** per-track `(x, y)` + visibility on every frame.

## How it works

1. **Token grid:** queries become N tokens; time becomes T positions
   (windowed).
2. **Attention patterns:** intra-track temporal attention; cross-track
   attention; full attention via proxies.
3. **Proxy tokens:** a small fixed number of register-like tokens that
   N_tracks tokens cross-attend to, replacing O(N²) self-attention with
   O(N·P). Enables ~70K tracks on single GPU.
4. **Iterative updates:** PIPs-style with 4D cost volumes within the
   window.
5. **Sliding window:** subsequent windows inherit refined estimates from
   the previous; trained with **unrolled windows** so gradients flow
   across the overlap boundary (RNN-style BPTT).

## Where it's been applied

- TAP-Vid (DAVIS, Kinetics, RGB-Stacking), PointOdyssey, Dynamic Replica.
- Reused as a **teacher** in [[cotracker3]]'s pseudo-labeling pipeline.

## Known limitations

- Windowed inference: not strictly per-frame causal — finite latency
  equal to window length.
- Trained synthetic-only — sim-to-real gap (addressed in [[cotracker3]]).

## Related methods

- **Successor:** [[cotracker3]] (same authors, simplified + real-data
  pseudo-labels).
- **Joint-tracking concept origin:** earlier Particle Video (Sand &
  Teller, 2008) did joint, but CoTracker is the deep-learning revival.
- **Window/iterative-update predecessors:** PIPs, PIPs++, TAPIR.
- **Frame-by-frame contemporaries:** [[tapnext]], [[track-on2]].
