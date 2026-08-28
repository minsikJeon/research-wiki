---
type: source
source_type: paper
title: "CoWTracker: Tracking by Warping instead of Correlation"
authors:
  - Lai, Zihang
  - Insafutdinov, Eldar
  - Sucar, Edgar
  - Vedaldi, Andrea
year: 2026
venue: "arXiv preprint (2026)"
url: https://cowtracker.github.io
raw_path: papers/cowtracker.pdf
status: ingested
tags: [point-tracking, dense-tracking, optical-flow, warping, cost-volume-free, transformer, vggt, synthetic-data]
related:
  - "[[cowtracker]]"
  - "[[point-tracking]]"
  - "[[joint-point-tracking]]"
  - "[[cmp-tap-methods]]"
  - "[[vggt]]"
  - "[[cotracker]]"
  - "[[cotracker3]]"
  - "[[pips]]"
  - "[[zihang-lai]]"
  - "[[eldar-insafutdinov]]"
  - "[[edgar-sucar]]"
  - "[[andrea-vedaldi]]"
  - "[[oxford-vgg]]"
  - "[[meta-ai]]"
  - "[[tap-vid-dataset]]"
created: 2026-08-28
updated: 2026-08-28
---

# CoWTracker: Tracking by Warping instead of Correlation

## TL;DR

A dense point tracker that **drops cost volumes entirely** and matches by
**warping** instead (feature at the current track estimate, sampled from
each target frame back to the query frame), iteratively refined by a
spatial-temporal transformer. Borrows the warp-based idea from optical-flow
work (WAFT) and the joint-track reasoning from [[cotracker|CoTracker]].
Because there is no quadratic cost-volume, it can index features at **high
resolution** (½ stride) → SOTA dense point tracking on TAP-Vid
DAVIS/Kinetics/RGB-Stacking + RoboTAP, **and** the *same model* transfers
zero-shot to optical flow, sometimes beating specialized flow methods
(Sintel/KITTI/Spring). Trained only on Kubric. Best with a [[vggt|VGGT]]
backbone.

## Why it matters

- **New matching-mechanism axis for TAP.** Every prior tracker in the wiki
  ([[pips]] → [[tapir]] → [[cotracker]] → [[tapnext]] → [[spatialtracker-v2]]
  …) builds a **cost volume / correlation** to search for matches. CoWTracker
  is the wiki's first tracker that establishes correspondence by
  **warping-only** — a single feature pairing per iteration, refined
  globally by attention. Adds a design pole the [[point-tracking]] concept
  did not have.
- **Unifies dense point tracking and optical flow.** One trained model does
  both; optical flow = a 2-frame "video." Directly bridges the two
  literatures the field kept separate.
- **Cost-volume-free → high-resolution indexing.** The quadratic cost of
  cost volumes forced prior dense trackers (AllTracker) to coarse grids.
  Warping is linear in targets × resolution × iterations, so CoWTracker
  indexes at ½ stride — the ablated source of its wins on thin structures,
  boundaries, and post-occlusion re-acquisition.
- **Same Oxford-VGG quartet as [[v-dpm]]** (Lai, Insafutdinov, Sucar,
  Vedaldi). The VGG tracking/3D line continues to set architectural
  direction.

**For the user's real-time 3D streaming-tracker project:** relevant as a
2D dense-tracking backbone design, but note the **latency caveat** — its
throughput is dominated by the **VGGT backbone, which is quadratic in video
length** (authors flag this; must chunk long clips). So CoWTracker is *not*
streaming/causal; it is an offline dense tracker. The warping head itself is
linear and cheap — the cost-volume-free idea could pair well with a
linear-time streaming backbone (e.g. [[zipmap]]-style) for a real-time dense
tracker. Clean design lead, not a solution.

## Key claims

- **Warping replaces cost volumes** (§3.2): iterative recurrence
  `u' = Φ(u | F)`; warp `G_t(p) = sample(F_t, p + u_t(p))` is the *only*
  place cross-frame features are paired. No correlation tensor / pyramid.
- **SOTA dense point tracking** (Tab 1, trained on Kubric only): mean over
  DAVIS/RGB-Stacking/RoboTAP/Kinetics **AJ 71.3 / δavg 81.8 / OA 93.3** —
  beats prior dense SOTA AllTracker (Kub+Mix: 68.9/80.5/91.5) and strong
  sparse baselines (CoTracker3 offline 62.0/74.4/89.6). At matched training
  data (both Kubric) the gap widens to +3.3/+2.2/+3.0 AJ/δ/OA over
  AllTracker.
- **Best occlusion accuracy** — +3.0 OA over AllTracker(Kub), +4.3 on DAVIS.
  Hypothesis: warping-indexed features preserve channel cues that
  dot-product cost-volume similarity discards.
- **Zero-shot optical flow, competitive-to-SOTA** (Tab 2, no flow training):
  Sintel Clean **0.78** EPE / Final 1.48 (beats SEA-RAFT 0.97/1.96, WAFT
  0.94/1.86 → 17–20% relative); KITTI **1.04** EPE / 4.87 Fl-all (beats all,
  −9.6% vs best WAFT); Spring 0.17 EPE / 0.75% 1px (within 0.06 of
  specialized WAFT).
- **More robust to large motion than cost volumes** (Fig 6): EPE-vs-motion
  slope 0.16 vs SEA-RAFT 0.30; **46% lower EPE** in the highest-motion bin.
  High-res local warping beats large cost-volume search radii.
- **Ablations** (Tab 3): VGGT best backbone (+1.7 δ over Pi3, ConvNet much
  worse); DPT upsampler best (+5.5 δ DAVIS over no-upsampler); temporal
  attention crucial on long videos (+11.7/+11.2 δ RGB/RoboTAP); iterative
  refinement crucial (+6.6 δ DAVIS); **warping crucial** — non-warping
  variant −23.4 δ DAVIS; ½-stride indexing beats coarser. K=5 iterations.

## Methods

Three parts (Fig 3):

1. **Backbone** — pretrained **[[vggt|VGGT]]** (patch-embed frozen, last
   blocks finetuned) produces low-res per-frame features; VGGT processes all
   frames jointly.
2. **DPT upsampler** — lifts features to ½ stride (`s'=2`) for
   high-resolution warping; + a small U-Net on raw images concatenated in.
3. **Warping-only tracker** — init displacement `u=0`, hidden state from
   `ϕ(F_0 ⊕ F_t)`. Each of K=5 iterations: warp all target features to the
   query frame at the current estimate (`G = W(F,u,p)`), concat `[G ⊕ F_0 ⊕
   u ⊕ h]`, process with a **spatial-temporal transformer** (ViViT-style:
   two spatial self-attention blocks per one temporal attention block;
   spatial = over query points P, temporal = over frames T), linear head
   predicts residual `Δu`. Visibility + confidence = sigmoid readouts on
   final hidden state.

No cost volume in the head → cost scales **linearly** in targets T,
resolution |P|, iterations K.

**Training:** Kubric only; Huber loss (visible + occluded tracks),
iteration-weighted; BCE for visibility/confidence (GT confidence = within
12 px). AdamW, lr 5e−4 cosine, batch 32 × up to 16 frames @ 336×560, 50k
iters, random frame-rate / length aug.

## Results

- **Point tracking:** Tab 1 (above) — mean 71.3 AJ, SOTA on all four
  datasets, all three metrics.
- **Optical flow (zero-shot):** Sintel 0.78/1.48, KITTI 1.04/4.87%, Spring
  0.17/0.75%.
- **Qual:** maintains coherent track through full occlusion (BMX, Fig 4)
  where DELTA loses target and AllTracker drifts/fragments.

## Limitations / open questions

- **Not streaming / causal.** Offline dense tracker. Backbone (VGGT) is
  **quadratic in video length** → long clips must be chunked. Throughput is
  backbone-dominated; the warping head is cheap.
- **Kubric-only synthetic training** — real data "significantly
  underleveraged"; no pseudo-label / real-video recipe (unlike
  [[cotracker3]]). Robustness to real lighting/noise is future work.
- **Fails** under extreme viewpoint change, long full occlusion, strong
  specularities.
- **Self-correction ceiling** — iterative refinement saturates at K=5–6.
- **2D only** — no 3D/depth output (unlike [[tapip3d]] / [[spatialtracker-v2]]).

## Connections

- **Contrasts every cost-volume tracker in the wiki:** [[pips]], [[tapir]],
  [[cotracker]], [[cotracker3]], [[tapnext]], [[spatialtracker-v2]] — all
  build correlation/cost volumes; CoWTracker warps instead. New axis on
  [[point-tracking]].
- **Inherits from [[cotracker]]:** the joint-track spatial-temporal
  attention ([[joint-point-tracking]]) — CoWTracker frames CoTracker's
  cross-track attention as equivalent to WAFT's global warp reasoning.
- **Backbone [[vggt]]:** best feature extractor in ablation; also the source
  of the quadratic-length limitation.
- **Optical-flow lineage (external):** WAFT (warp-based flow), SEA-RAFT,
  RAFT — CoWTracker imports warping from this side and beats it zero-shot.
- **Same authors as [[v-dpm]]:** [[edgar-sucar]] (V-DPM lead),
  [[eldar-insafutdinov]], [[zihang-lai]], [[andrea-vedaldi]] — the
  [[oxford-vgg]] × [[meta-ai]] cluster.
- **Evaluated on [[tap-vid-dataset]]** + RoboTAP.

## Citation

```
Lai, Z., Insafutdinov, E., Sucar, E., & Vedaldi, A. (2026). CoWTracker:
Tracking by Warping instead of Correlation. arXiv preprint. Visual Geometry
Group, University of Oxford + Meta AI. Project: https://cowtracker.github.io
```
