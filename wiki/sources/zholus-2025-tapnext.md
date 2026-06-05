---
type: source
source_type: paper
title: "TAPNext: Tracking Any Point (TAP) as Next Token Prediction"
authors:
  - Zholus, Artem
  - Doersch, Carl
  - Yang, Yi
  - Koppula, Skanda
  - Patraucean, Viorica
  - He, Xu Owen
  - Rocco, Ignacio
  - Sajjadi, Mehdi S. M.
  - Chandar, Sarath
  - Goroshin, Ross
year: 2025
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2504.05579
raw_path: papers/2504.05579v2.pdf
status: ingested
tags: [point-tracking, video-understanding, state-space-model, transformer, vit, online-tracking, masked-decoding]
sources: []
related:
  - "[[point-tracking]]"
  - "[[tapnext]]"
  - "[[tap-vid-dataset]]"
  - "[[carl-doersch]]"
  - "[[google-deepmind]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAPNext: Tracking Any Point (TAP) as Next Token Prediction

## TL;DR

TAPNext recasts [[point-tracking]] as masked token decoding: video patches and
per-point trajectory tokens are concatenated, and unknown trajectory positions
are filled in by a generic backbone (interleaved SSM + ViT, no
tracking-specific inductive biases). It runs **causally per-frame** with
~5 ms latency yet achieves **SOTA** on [[tap-vid-dataset]] DAVIS and Kinetics,
beating windowed and offline trackers on most metrics. Classical tracking
heuristics (cost-volume attention, motion-cluster attention, coordinate
read-out) **emerge** as attention patterns rather than being hard-coded.

## Why it matters

Most prior TAP methods (TAPIR, CoTracker, LocoTrack, TAPTR) lean on
heavily-engineered components: cost volumes, iterative refinement, windowed
inference, local search windows, temporal smoothness priors. TAPNext is
evidence that **none of this is required** if you have (a) a unified token
stream, (b) a linear recurrent backbone for temporal carry, and (c) a
classification-style coordinate head. For robotics applications where
**per-frame latency** is the binding constraint, this is a meaningful
architectural simplification — the model runs at 187+ FPS on H100 with
~5 ms latency vs. 80-2210 ms for windowed/video baselines.

## Key claims

- **Generic backbone beats domain-specific:** an off-the-shelf TRecViT
  (interleaved SSM + ViT, originally for video classification) fine-tuned with
  the right loss exceeds all tracking-specific architectures on TAP-Vid
  (8 of 12 metrics, p. 6 Table 1).
- **Causality is not a tradeoff:** strictly causal per-frame inference matches
  or beats bidirectional / windowed methods that look at future frames.
- **Tracking heuristics emerge:** attention maps show cost-volume-like heads
  (appearance matching), coordinate-read-out heads, and motion-cluster heads
  arise from end-to-end training without being designed in (Figs 3-4).
- **Coordinate head as classification > regression:** discretizing the
  coordinate into 256 bins per axis with a softmax + Huber combined loss is
  *the* most important ablation (44.7 → 55.0 AJ when swapped for regression,
  Table 3).
- **SSM beats temporal attention for length generalization:** swapping the SSM
  for temporal attention with RoPE drops full-length DAVIS AJ from 55.0 → 17.3
  (Table 4). RoPE doesn't save attention here.
- **BootsTAPNext** (semi-supervised fine-tune on ~15M real video clips
  following [13]) is the best variant — 65.2 AJ DAVIS-first, 68.9 DAVIS-strided.
- **Long-video failure is an SSM-state issue:** beyond ~150 frames performance
  collapses. Partial mitigation: clipping the SSM forget gate to [0, 0.1] and
  broadcasting query features along the temporal axis.

## Methods

- **Input:** video as `[T, h×w, C]` ViT patch tokens + `[T, Q, C]` per-point
  trajectory tokens. The known query token holds a positional encoding of
  `(x, y)`; all other trajectory tokens are a learned mask token. No temporal
  positional embedding (it hurt length generalization).
- **Backbone:** L layers, each one a TRecViT block = RecurrentGemma SSM scan
  along time (treating space as batch) → ViT self-attention across
  `h×w + Q` tokens per frame (treating time as batch). Two sizes: TAPNext-S
  (56M), TAPNext-B (194M).
- **Heads:** coordinate head (256-way classification per axis, expectation
  read-out for continuous prediction; softmax CE + Huber loss) + visibility
  head (binary sigmoid). Auxiliary loss applied at every layer, not just last.
- **Training:** 256-batch, 48-frame clips, 256 points/video, 256×256 resolution,
  300K steps on a custom 500K-video Kubric extension (with camera panning +
  motion blur); BootsTAPNext fine-tunes 1.5K steps on ~15M real clips using the
  BootsTAP student-teacher EMA recipe.
- **Inference:** causal, per-frame; for fair query-strided comparison they run
  forward + backward from each query.

## Results

- **TAP-Vid DAVIS (BootsTAPNext-B, 256×256):** AJ 65.2 first / 68.9 strided;
  δ_avg 78.5 / 82.4; OA 91.2 / 91.6. Beats CoTracker3, TAPTRv3, LocoTrack-B,
  BootsTAPIR on most metrics.
- **TAP-Vid Kinetics:** AJ 57.3 first / 62.2 strided. SOTA among per-frame
  online methods; competitive with offline.
- **Latency (H100, 256 query points):** 5.05 ms vs. CoTracker3's 80 ms vs.
  LocoTrack-B's 2210 ms. ~40× faster than the next-best per-point methods.

## Limitations / open questions

- **Long videos:** breaks past ~150 frames (trained on 48). The proposed
  forget-gate clamp + query-broadcast is a band-aid, not a fix.
- **No explicit appearance memory:** unlike CoTracker, the model has no
  named "memory" slot. Robustness across long occlusions relies entirely on
  whatever the SSM state happens to carry.
- **Synthetic-data dependence:** the baseline model is sim-only Kubric; the
  best variant needs the ~15M-clip semi-supervised pipeline from BootsTAP,
  which is non-trivial to reproduce.
- **Open question:** can the emergent attention patterns (cost-volume,
  motion-cluster) be quantified or *forced* with auxiliary losses to improve
  data efficiency?
- **Open question:** does the masked-decoding framing extend to dense
  prediction tasks beyond TAP (segmentation tracking, 3D scene flow,
  pose tracking)? The authors hint yes; no evidence yet.

## Connections

- Method: [[tapnext]] — full method-page treatment.
- Concept: [[point-tracking]] — TAP as a problem family.
- Dataset: [[tap-vid-dataset]] — the benchmark used.
- People: [[carl-doersch]] — common thread across TAP-Vid, TAPIR, BootsTAP,
  and TAPNext (the "TAP line" at DeepMind).
- Org: [[google-deepmind]] — primary affiliation.
- Future seeds (not yet pages): TAPIR, CoTracker (1-3), LocoTrack, TAPTR(v2/v3),
  Track-On, BootsTAP, RecurrentGemma, TRecViT, Kubric, DAVIS, Kinetics,
  MultiMAE, VideoMAE. Promote to entity/method pages when a second source
  references them.

## Citation

Zholus, A., Doersch, C., Yang, Y., Koppula, S., Patraucean, V., He, X. O.,
Rocco, I., Sajjadi, M. S. M., Chandar, S., & Goroshin, R. (2025).
*TAPNext: Tracking Any Point (TAP) as Next Token Prediction.*
arXiv:2504.05579v2 [cs.CV]. https://arxiv.org/abs/2504.05579
