---
type: concept
title: Streaming Perception
status: growing
tags: [streaming-perception, latency, embodied-perception, evaluation, online-tracking, autonomous-driving]
sources:
  - "[[li-2020-streaming-perception]]"
related:
  - "[[streaming-accuracy]]"
  - "[[online-vs-offline-tracking]]"
  - "[[argoverse-hd-dataset]]"
  - "[[streamer-meta-detector]]"
  - "[[q-emergent-tracking-heuristics]]"
created: 2026-05-28
updated: 2026-05-28
---

# Streaming Perception

## Definition

A perception evaluation paradigm where the algorithm must **continuously
report the current state of the world** under realistic latency
constraints, rather than being scored offline on per-frame predictions.
Introduced by Li, Wang & Ramanan (ECCV 2020).

The key reframing: by the time the algorithm finishes processing frame
`t`, the world has moved on. Realistic evaluation must compare the
algorithm's *most recent output* against ground truth at every time
instant — a zero-order hold on the output stream vs. the data stream
(see [[streaming-accuracy]] for the formal metric).

## Why it matters

Foundational concept for the entire "online / causal / real-time"
discussion across this wiki:

- [[online-vs-offline-tracking]] is the application of streaming
  perception to TAP — frame / window / video latency tiers.
- The current SOTA TAP debate around [[tapnext]] / [[track-on2]] /
  [[tapnext-plus-plus]] is essentially a streaming-perception
  optimization problem: minimize latency, maximize accuracy under
  zero-order-hold evaluation.
- [[spatialtracker-v2]]'s `--track_mode online` sliding-window
  inference is a streaming-perception architecture, even if the paper
  doesn't use that vocabulary.

## Three load-bearing findings from Li et al. 2020

1. **Offline AP drops dramatically under streaming.** HTC goes from
   38.0 → 6.2 on Argoverse-HD detection.
2. **Tracking and forecasting are necessary**, not optional. The best
   single-GPU pipeline (Mask R-CNN R50 @ s0.5 + association + Kalman
   forecaster + dynamic scheduler) recovers to 17.8 AP. Infinite GPUs
   only get to 20.3 — the bottleneck is algorithmic, not compute.
3. **Doing nothing can minimize latency.** Dynamic scheduling sometimes
   chooses to wait for a fresh frame rather than start processing a
   stale one. Optimal scheduling is non-greedy.

## Variants observed in this wiki

- **2D detection** ([[li-2020-streaming-perception]]) — the original.
- **Online point tracking** ([[tapnext]], [[track-on2]], implicit in
  [[spatialtracker-v2]]) — the same problem at a different output
  granularity.
- **Online 3D / 4D reconstruction** ([[cut3r]], [[point4d]] chunk
  chaining, [[spatialtracker-v2]] online) — streaming geometry instead
  of streaming detection.
- **Real-time robot control** ([[rtc]]) — streaming *actions* under
  inference delay. Not the original framing but the same conceptual
  problem; RTC's `d := ⌊δ/Δt⌋` inference delay is structurally
  identical to streaming perception's algorithm latency.

## Evidence across sources

- [[li-2020-streaming-perception]]: original framing + meta-benchmark
  + [[argoverse-hd-dataset]].
- [[anon-2026-point4d]]: long-video 4D tracking via chunk chaining is
  implicitly a streaming-perception solution to occlusion-robust 3D
  trajectory estimation.
- [[black-2025-rtc]]: applies the same "report-at-time-t" framing to
  robot actions, with explicit `d`-step inference delay.

## Open questions

- **How do modern foundation models (VGGT, DepthAnything-3, π0.5)
  change the latency-accuracy sweet spot?** The 2020 paper used Mask
  R-CNN; the calculus is likely very different with VLAs and feed-
  forward 3D reconstructors.
- **Streaming forecasting is task-specific.** Forecasting bounding
  boxes is a solved problem; forecasting per-pixel scene flow or 6-DOF
  poses is not. Open: a unified forecaster for arbitrary geometric
  outputs.
- **End-to-end streaming-aware training.** The 2020 paper's Appendix
  C.2 has a sketch; later work has built on it. But the wiki doesn't
  yet have a definitive "streaming-aware training" reference for
  modern foundation models.
