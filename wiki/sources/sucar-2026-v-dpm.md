---
type: source
source_type: paper
title: "V-DPM: 4D Video Reconstruction with Dynamic Point Maps"
authors:
  - Sucar, Edgar
  - Insafutdinov, Eldar
  - Lai, Zihang
  - Vedaldi, Andrea
year: 2026
venue: arXiv (cs.CV)
url: https://arxiv.org/abs/2601.09499
raw_path: papers/Sucar et al. - 2026 - V-DPM 4D Video Reconstruction with Dynamic Point Maps.pdf
status: ingested
tags: [4d-reconstruction, video-reconstruction, transformer, dynamic-point-maps, vggt, scene-flow]
sources:
  - "[[wang-2025-vggt]]"
related:
  - "[[v-dpm]]"
  - "[[vggt]]"
  - "[[dynamic-point-maps]]"
  - "[[4d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[andrea-vedaldi]]"
  - "[[oxford-vgg]]"
created: 2026-05-24
updated: 2026-05-24
---

# V-DPM: 4D Video Reconstruction with Dynamic Point Maps

## TL;DR

V-DPM extends [[vggt]] (trained on **static** scenes) to **dynamic** 4D
reconstruction by adding **Dynamic Point Map (DPM) heads** on top. The
backbone is fine-tuned from VGGT with a "modest amount of synthetic
dynamic data" — no training from scratch. DPMs are
**viewpoint- and time-invariant** point maps that encode shape + scene
flow + camera in a unified representation. **Halves the error rate vs DPM,
MonST3R, St4RTrack.** Oxford VGG ([[andrea-vedaldi]] senior).

## Why it matters

The "smallest possible architectural change to make VGGT dynamic." The
DPM representation [Sucar 2025 — the predecessor not yet ingested] is
elegant: a point map `P_i(t_j, π_k)` encodes "where pixel u in image i
is at time t_j viewed from camera k." Pair-wise this needs O(N²) maps;
V-DPM shows how to redundancy-collapse this for video.

This is also a **strong argument for VGGT-as-backbone**: a static
reconstructor fine-tuned on dynamic data is competitive with
purpose-built 4D models, validating the "feed-forward 3D backbone is
the substrate for 4D" hypothesis.

## Key claims

- **DPMs are complete.** The 4 maps `P_0(t_0, π_0)`, `P_0(t_1, π_0)`,
  `P_1(t_0, π_0)`, `P_1(t_1, π_0)` encode shape + motion + camera
  intrinsics + camera extrinsics for the two-image case.
  Multi-view extension reduces O(N²) redundancy.
- **Two-phase backbone design.** Phase 1: time-varying viewpoint-invariant
  reconstruction. Phase 2: additional layers extract time invariance
  (implicit dynamic correspondences across the phase-1 outputs).
- **Architecture is essentially VGGT-shaped** → easy to extend
  a pre-trained static model. Two new heads (time-variant DPM and
  time-invariant DPM); the rest is fine-tuned.
- **Halves error rate vs DPM (pairwise), MonST3R, St4RTrack** on
  standard 4D benchmarks.
- **Same backbone statistics as VGGT.** Fine-tuning with synthetic
  dynamic data (Kubric) is enough; doesn't need 4D-annotated real data
  at scale.
- Recovers **dynamic depth + 3D motion of every point** — unlike π³
  (Pi3) which extends VGGT but only recovers dynamic depth.

## Methods

- **Architecture:** VGGT backbone (frozen-then-fine-tuned) +
  - **Time-variant DPM head** → per-frame DPMs at the source time
    (preserves VGGT's static behavior).
  - **Time-invariant DPM head** with AdaLN conditioning on a
    **target-time token** → reconstructs the scene at the chosen
    target time, fusing information from all input frames.
  - Camera + time-variant heads share the VGGT global+frame attention
    pattern; time-invariant decoder is added.
- **Multi-image DPMs.** Instead of the O(N²) pairwise formulation,
  predict all maps relative to a fixed reference viewpoint π_0.
  Reduces to O(N) maps per target time.
- **Training:** fine-tune VGGT on Kubric (dynamic synthetic) — relatively
  little data needed.

## Results (headline)

- More than halves error vs DPM (pairwise), MonST3R, St4RTrack on
  standard 4D benchmarks.
- Recovers dynamic scene flow + camera + depth in one shot.
- Demonstrates "VGGT fine-tunes to 4D" works well.

## Limitations / open questions

- Compared mostly against pre-VGGT 4D methods; head-to-head vs concurrent
  feed-forward 4D models ([[any4d]], [[4rc]], [[trace-anything]]) not in
  this paper. [[4rc]]'s related-work section criticizes V-DPM for
  "high computational cost and inflexible decoding."
- Trained on synthetic-only dynamic data (Kubric); real-world
  generalization not deeply evaluated.
- The two-phase (time-variant then time-invariant) architecture adds
  complexity beyond VGGT.

## Connections

- Method: [[v-dpm]] (V-DPM = Video Dynamic Point Maps).
- Concepts: [[dynamic-point-maps]], [[4d-reconstruction]],
  [[pointmap-representation]].
- **Direct precursor:** DPM (Sucar et al. 2025) — pairwise version.
- **Backbone:** [[vggt]] (fine-tuned).
- **Concurrent feed-forward 4D competitors:** [[any4d]] (CMU, scene flow
  + multi-modal), [[4rc]] (NTU+Oxford, conditional query),
  [[trace-anything]] (anonymous, trajectory fields), Pi3 (extends VGGT
  but no motion).
- Author: [[andrea-vedaldi]] (senior). Edgar Sucar (lead) + Eldar
  Insafutdinov + Zihang Lai — defer entities until 2nd paper.
- Org: [[oxford-vgg]].

## Citation

Sucar, E., Insafutdinov, E., Lai, Z., & Vedaldi, A. (2026). *V-DPM: 4D
Video Reconstruction with Dynamic Point Maps.* arXiv:2601.09499.
https://arxiv.org/abs/2601.09499
