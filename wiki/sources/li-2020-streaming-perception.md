---
type: source
source_type: paper
title: "Towards Streaming Perception"
authors:
  - Li, Mengtian
  - Wang, Yu-Xiong
  - Ramanan, Deva
year: 2020
venue: ECCV 2020
url: https://arxiv.org/abs/2005.10420
raw_path: papers/2005.10420v2.pdf
status: ingested
tags: [streaming-perception, latency, embodied-perception, object-detection, tracking, forecasting, autonomous-driving, evaluation, benchmark]
sources: []
related:
  - "[[streaming-perception]]"
  - "[[streaming-accuracy]]"
  - "[[online-vs-offline-tracking]]"
  - "[[mengtian-li]]"
  - "[[yu-xiong-wang]]"
  - "[[deva-ramanan]]"
  - "[[cmu-ri]]"
  - "[[uiuc]]"
  - "[[argoverse-hd-dataset]]"
created: 2026-05-28
updated: 2026-05-28
---

# Towards Streaming Perception (Li, Wang, Ramanan — ECCV 2020)

## TL;DR

This is **the foundational paper for latency-aware perception evaluation.**
It proposes **streaming accuracy** — a single metric that jointly evaluates
latency + accuracy by asking the algorithm to *continuously* report the
state of the world. The key insight: by the time the algorithm finishes
processing a frame, the world has changed, so realistic evaluation must
query the algorithm's output buffer against ground truth at every time
instant. They build a meta-benchmark that converts any single-frame task
into a streaming task and instantiate it on object detection /
segmentation in autonomous driving with their new **Argoverse-HD**
dataset.

## Why it matters

Foundational for **every** later "online" / "causal" / "real-time"
discussion in this wiki:

- [[online-vs-offline-tracking]] traces directly to this paper.
- The TAPNext / Track-On2 / TAPNext++ frame-by-frame inference debate
  is a downstream application of the streaming accuracy framing.
- The user just asked detailed questions about [[spatialtracker-v2]]'s
  online inference and the [[train-inference-mismatch]] — this paper
  is the conceptual root for why those questions matter.

Three findings worth recording as load-bearing wiki claims:

1. **Standard offline AP drops dramatically under streaming.** Hybrid
   Task Cascade goes from **AP 38.0 (offline) → 6.2 (streaming)** for
   object detection. A 6× drop.
2. **Tracking and forecasting emerge as necessary internal
   representations** for streaming perception. With the best detector +
   association + forecaster + dynamic scheduling: 6.2 → 17.8 AP.
   Infinite GPUs only get 20.3 — meaning hardware scaling does **not**
   close the gap. The bottleneck is algorithmic, not compute.
3. **"Doing nothing" can minimize latency.** Their dynamic scheduler
   sometimes chooses to sit idle and wait for a fresher frame rather
   than start processing one that will soon become stale.

This paper is **also a [[deva-ramanan]] paper** — third in this wiki
after [[mapanything]] and [[any4d]]. Confirms Ramanan as a senior with a
long-standing line on embodied / streaming perception that the
MapAnything → Any4D 4D-reconstruction line is the modern continuation of.

## Key claims

- **Streaming accuracy** (Eq 4): apply any single-frame loss `L` to
  pairs `(y_i, ŷ_{φ(t_i)})`, where `φ(t) = argmax_j (s_j < t)` is the
  most recent output produced before the GT timestamp. Equivalent to
  applying zero-order hold to the algorithm's output stream and
  comparing against ground truth at every instant.
- **Streaming forces predictive forecasting.** A finite-time algorithm
  cannot use the current frame `x_t` to predict at time `t` — so it
  must reason temporally and forecast.
- **Meta-benchmark.** Any single-frame task is convertible: pose
  estimation, segmentation, etc. They instantiate on detection +
  segmentation.
- **Argoverse-HD dataset.** They re-annotate Argoverse 1.1 with dense
  30-FPS 2D bounding boxes for 8 driving-relevant COCO classes (person,
  bicycle, car, motorcycle, bus, truck, traffic light, stop sign).
  Pseudo ground truth (from arbitrarily expensive offline detectors)
  has 0.9925 Pearson correlation with real labels → usable for cheap
  labeling at scale.
- **Two computation models:**
  - **Single GPU:** at most one GPU job concurrent; CPU jobs (association,
    forecasting) free.
  - **Infinite GPU:** unlimited concurrent GPU jobs (simulated). Used to
    distinguish algorithmic from hardware bottlenecks.
- **Optimal "sweet spot" on the latency–accuracy Pareto curve.** Not
  the fastest detector, not the most accurate — but the one whose
  runtime budget best amortizes over the streaming evaluation. For
  detection on Argoverse-HD: Mask R-CNN R50 @ 0.5 scale, 56.7ms.
- **Dynamic scheduling beats fixed-stride scheduling.** Algorithm 1
  (shrinking-tail dynamic schedule) raises AP from 12.0 → 13.0 by
  intentionally idling near frame boundaries to avoid temporal aliasing.
- **Forecasting is the biggest single boost.** Adding an async Kalman
  forecaster raises 13.0 → 16.7 → 17.8 (after re-optimizing detection
  scale). Forecasting compensates for the algorithm's own latency.
- **Visual tracking ≥ association** when the tracker is fast enough to
  run between detection calls. Tracking + forecasting under infinite
  GPU: 20.1 AP.
- **The gap to offline doesn't close with infinite compute.** Best
  streaming with infinite GPUs: 20.3 vs offline 38.0. The bottleneck
  is algorithmic.

## Methods

- **Streamer (meta-detector):** modular pipeline = Detector (GPU) +
  Association (CPU) + Forecasting (CPU) + Dynamic Scheduler.
  Pipeline is asynchronous; CPU modules run concurrently with the next
  GPU detection.
- **Dynamic scheduler (Alg. 1):** "shrinking-tail" — decides whether to
  process the next frame or idle, minimizing expected temporal
  mismatch between output stream and data stream.
- **Asynchronous Kalman forecaster:** predicts current world state
  given the latest detection + association track history; supports
  multi-step prediction-only updates (no measurement) between detections.
- **End-to-end alternative** (Appendix C.2): a model can be directly
  trained to optimize streaming accuracy, and tracking + forecasting
  representations emerge from gradient-based learning.

## Results (headline)

**Argoverse-HD detection (single GPU, streaming AP):**

| Configuration                                          | AP   |
|--------------------------------------------------------|------|
| HTC @ s1.0 (offline)                                   | 38.0 |
| HTC @ s1.0 (streaming, naive)                          | 6.2  |
| RetinaNet R50 @ s0.2 (fast detector)                   | 5.5  |
| Mask R-CNN R50 @ s0.5 (optimized)                      | 12.0 |
| + Dynamic scheduling (Alg. 1)                          | 13.0 |
| + Association + Forecasting                            | 16.7 |
| + Re-optimize detection scale (s0.5 → s0.75)           | 17.8 |
| + Infinite GPUs                                        | 20.3 |

**Visual tracking (Table 3):**

| Configuration                                                          | AP   |
|------------------------------------------------------------------------|------|
| Detection + Visual Tracking                                            | 12.0 |
| + Forecasting                                                          | 13.7 |
| + Re-optimize Detection                                                | 16.5 |
| + Infinite GPUs (w/ Forecasting)                                       | 20.1 |
| Detection + Simulated Fast Tracker (2×) + Forecasting + Single GPU     | 19.8 |

Streamer (the meta-detector) improves streaming AP by 4–80% across 80
configurations (8 detectors × 5 scales × 2 compute models), average 33%.

## Limitations / open questions

- **Hardware-dependent.** A GTX 1080 Ti at the time of publication;
  modern GPUs would shift the sweet spot. The simulation framework
  partially compensates but real hardware comparison still matters.
- **Detection + tracking only.** They flag instance segmentation in
  Appendix; broader tasks (depth, point tracking, VLA) are untested.
  This is exactly the gap that 5+ years of follow-up work has been
  filling.
- **8 classes** is a narrow slice of COCO; later streaming-perception
  work has expanded.
- **No video-foundation-model baseline.** This paper predates the
  VGGT / DUSt3R / VLA wave; the question of whether modern large
  models change the sweet-spot calculus is open.
- **Dataset annotation density.** Pseudo-GT is necessary for instance
  segmentation; the long-term feasibility of streaming benchmarks
  depends on cheap dense labels.

## Connections

- **Concept introduced:** [[streaming-perception]] and [[streaming-accuracy]]
  (the metric).
- **Foundational ancestor for:** [[online-vs-offline-tracking]] —
  the framing of "frame / window / video" latency tiers in this wiki's
  TAP analysis ultimately descends from this paper. Worth back-linking
  the concept page.
- **Dataset introduced:** [[argoverse-hd-dataset]] — Argoverse 1.1
  re-annotated at 30 FPS with 8 COCO-style classes.
- **Author overlap with existing wiki:**
  - [[deva-ramanan]] — **third source** for him after [[mapanything]],
    [[any4d]]. Promote his entity page to a richer status.
  - [[mengtian-li]] (first author, CMU) — new entity.
  - [[yu-xiong-wang]] (UIUC) — new entity.
- **Orgs:** [[cmu-ri]] (already in wiki), [[uiuc]] (new), Argo AI
  (defer until 2+ mentions).
- **Conceptual successors not yet in wiki but cited downstream:**
  StreamYOLO, Real-time Detection with Streaming Perception (CVPR),
  many streaming-detection papers.
- **Tracking-and-forecasting-emerge finding** parallels
  [[q-emergent-tracking-heuristics]] from the TAPNext discussion —
  both are about *useful representations emerging from end-to-end
  pressure*. Worth cross-linking.

## Citation

Li, M., Wang, Y.-X., & Ramanan, D. (2020). *Towards Streaming
Perception.* ECCV 2020. arXiv:2005.10420v2 [cs.CV].
https://arxiv.org/abs/2005.10420
