---
type: method
title: Streamer (Meta-Detector for Streaming Perception)
status: stub
tags: [streaming-perception, object-detection, tracking, forecasting, scheduling, autonomous-driving]
sources:
  - "[[li-2020-streaming-perception]]"
related:
  - "[[streaming-perception]]"
  - "[[streaming-accuracy]]"
  - "[[argoverse-hd-dataset]]"
created: 2026-05-28
updated: 2026-05-28
---

# Streamer (Meta-Detector for Streaming Perception)

Modular asynchronous pipeline that turns any off-the-shelf detector into
a streaming-perception system. Introduced alongside the streaming-accuracy
metric in [[li-2020-streaming-perception]].

## One-line summary

Detector (GPU, async) + Association (CPU) + Asynchronous Kalman
Forecaster (CPU) + Dynamic shrinking-tail scheduler — all running
concurrently to report current-world-state at arbitrary frame rate
under continuous evaluation.

## Inputs / outputs

- **In:** real-time sensor stream (e.g. Argoverse-HD video at 30 FPS).
- **Out:** continuously updated buffer of bounding boxes (or any
  single-frame task output), queryable at any time `t`.

## How it works

1. **Detector** runs on the GPU at its native runtime (e.g. Mask R-CNN
   R50 @ s0.5 ≈ 57 ms).
2. **Association** (CPU) links bounding boxes between consecutive
   detections via IoU greedy matching — lightweight (<2 ms).
3. **Asynchronous Kalman Forecaster** (CPU) runs a prediction step
   right before each ground-truth query, producing the algorithm's
   best estimate of the *current* world state, compensating for
   detector latency. Multiple prediction-only updates can run between
   detections.
4. **Dynamic shrinking-tail scheduler** (Alg. 1) decides whether to
   process the next frame or idle. Avoids temporal aliasing where a
   stale-frame detection would land just before the next fresh frame.
   Sometimes the right action is to **do nothing**.
5. **Alternative tracker module** (Sec 4.4 of source): a fast
   detection-conditioned tracker replaces the association module when
   the tracker can re-localize between detection calls faster than
   running detection.

## Where it's been applied

- **Argoverse-HD** object detection — main result.
- **Argoverse-HD** instance segmentation (Appendix).
- Generalizes to any single-frame task with a defined association +
  forecasting interface.

## Known limitations

- Forecasting is task-specific (Kalman filter is great for bounding
  boxes, harder for segmentation masks).
- Hardware-dependent — sweet spot shifts on different GPUs.
- Hand-designed pipeline; later end-to-end approaches (e.g. StreamYOLO)
  try to absorb the modules into a single network.

## Related methods

- **Concept origin:** [[streaming-perception]].
- **Direct successors** (not yet ingested): StreamYOLO, DAMO-StreamNet,
  Real-time Streaming Perception (NeurIPS), etc.
- **Adjacency:** any sliding-window 3D / 4D pipeline in this wiki
  (e.g. [[spatialtracker-v2]] online, [[cut3r]]) is implicitly subject
  to the same streaming-accuracy framing this paper introduced.
