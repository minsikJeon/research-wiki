---
type: source
source_type: paper
title: "3DFlowAction: Learning Cross-Embodiment Manipulation from 3D Flow World Model"
authors: [Zhi, Hongyan; Chen, Peihao; Zhou, Siyuan; Dong, Yubo; Wu, Quanxi; Han, Lei; Tan, Mingkui]
year: 2025
venue: "arXiv preprint (Jun 2025; under review)"
url: https://arxiv.org/abs/2506.06199
raw_path: papers/2506.06199v1.pdf
status: ingested
tags: [point-tracking, manipulation, flow, video-diffusion, world-model, cross-embodiment, vla]
sources: []
related:
  - "[[3d-flow-action]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[xu-2024-im2flow2act]]"
  - "[[bharadhwaj-2024-track2act]]"
  - "[[kuang-2026-dex4d]]"
  - "[[cotracker3]]"
  - "[[3d-point-tracking]]"
created: 2026-06-27
updated: 2026-06-27
---

# 3DFlowAction

## TL;DR

Lifts [[xu-2024-im2flow2act]]'s 2D-flow generation pipeline to **3D
optical flow** and pairs it with three downstream modules: (1) a
**GPT-4o-based closed-loop verifier** that renders the predicted final
state and re-prompts re-generation if it doesn't match the task; (2) a
**task-aware grasp selector** using affordance + IK reachability; (3)
an **optimization-based action policy** that requires no action labels
(minimize ‖proj(p_t) − predicted‖ over end-effector poses). Built on
**ManiFlow-110k**, a synthesized cross-source 3D flow dataset from
BridgeV2 / RT-1 / AgiWorld / DROID / RH20T-Human / HOI4D / LIBERO.

## Why it matters

This is the direct **2D → 3D lift** of [[xu-2024-im2flow2act]]. The
2D-flow line had a known gap: out-of-plane rotation and z-axis precision
fail (Im2Flow2Act's pouring failures, Track2Act's grasp errors). 3DFlow
solves this by:

- generating flow with an explicit depth channel (4-channel: u, v, z,
  visibility) rather than recovering depth post-hoc;
- using the SVD-fit transform from first→last flow to render a
  predicted final RGB scene, then asking GPT-4o "does this match the
  instruction?" as a closed-loop checker; and
- minimizing 3D point distances directly in the optimization policy.

ManiFlow-110k is the first **cross-source 3D flow dataset** (110K
trajectories from 7 datasets) and is positioned as a reusable training
corpus for the flow-as-interface line.

## Key claims

- **70% average success rate** on 4 foundation tasks (pour, insert pen,
  hang cup, open drawer) vs. 20–25% for prior world-model / 2D-flow
  baselines (AVDC, ReKep, Im2Flow2Act*) — Table 1.
- **Cross-embodiment transfer** — same trained model works on Franka
  (67.5%) and XTrainer/Dobot (70%) with no hardware-specific training
  (Table 2).
- **3D flow strictly beats 2D flow** on tasks involving rotation:
  insert pen 7/10 vs. 2/10, hang cup 5/10 vs. 0/10 (vs.
  Im2Flow2Act*).
- **Closed-loop GPT-4o verification** catches incorrect flow
  predictions and triggers re-generation, important in disturbed
  scenes.
- **Optimization > learned BC for novel tasks** when only 30 human
  demos per task are available (no action labels): 3DFlowAction 70%
  vs. PI0 50%, Im2Flow2Act 27.5% (Table 3).

## Methods

### ManiFlow-110k construction (§3.1)

1. **Moving-object auto-detect:** Grounding-SAM2 segments gripper from
   frame 1 → sample grid points outside gripper mask → Co-Tracker3
   tracks them → keep points with significant motion → max bbox =
   moving-object region.
2. Co-Tracker3 again to extract 2D object flow.
3. DepthAnythingV2 → project 2D flow to 3D (4-channel output).
4. 110K trajectories from BridgeV2 (27%), RT-1 (18%), RH20T-Human
   (27%), AgiWorld (8%), DROID (13%), HOI4D (3%), LIBERO (4%).

### 3D flow world model (§3.2)

- AnimateDiff (SD v1.5 backbone) inflated with motion module.
- LoRA on SD layers; motion module trained from scratch.
- **Bypass the SD VAE** because it can't encode depth (4-channel input
  fed directly to UNet).
- Conditions: CLIP-encoded initial RGB + task prompt; sinusoidal PE
  for initial points F₀.

### Action generation (§4)

1. **Closed-loop flow generation:** SVD-fit `T = SVD(P₂, P₁)`, apply T
   to object point cloud, render predicted final scene, GPT-4o checks
   alignment, re-generate if FAIL.
2. **Task-aware grasp:** GPT-4o suggests which part of the object to
   grasp → AnyGrasp generates candidates → apply T → IK check
   reachability → select.
3. **Optimization action policy:** N farthest-point samples → minimize
   ‖p_t^i − (predicted-flow at i, t)‖² over end-effector pose at each
   timestep. No action labels needed.

## Results

### World-model comparison (Table 1, success / 40 trials total)

| Method | Pour | Pen | Hang | Drawer | Total |
|---|---|---|---|---|---|
| AVDC | 1/10 | 2/10 | 0/10 | 5/10 | 20.0% |
| ReKep | 2/10 | 1/10 | 3/10 | 2/10 | 20.0% |
| Im2Flow2Act* | 2/10 | 2/10 | 0/10 | 6/10 | 25.0% |
| **3DFlowAction** | **6/10** | **7/10** | **5/10** | **10/10** | **70.0%** |

### Imitation-learning comparison (Table 3)

| Method | Pour | Pen | Hang | Drawer | Total |
|---|---|---|---|---|---|
| PI0 (VLA) | 5/10 | 5/10 | 4/10 | 6/10 | 50.0% |
| Im2Flow2Act | 4/10 | 2/10 | 0/10 | 5/10 | 27.5% |
| **3DFlowAction** | **6/10** | **7/10** | **5/10** | **10/10** | **70.0%** |

## Limitations / open questions

- **Optimization-based action policy** is slow vs. learned policies; no
  reactivity to disturbances mid-step (only the GPT-4o re-plan is
  closed-loop).
- **Depth from DepthAnythingV2** introduces per-frame depth noise;
  Dex4D specifically critiques this and uses VideoDepthAnything +
  median-rescaling for smoother depth.
- **GPT-4o latency** in the verifier loop (~few seconds per check) —
  not suitable for highly dynamic tasks.
- **No fingered manipulation** — parallel-jaw only.

## Connections

- **Predecessor:** [[xu-2024-im2flow2act]] — same AnimateDiff flow
  generator, lifted from 2D to 3D and from sim-trained learned policy
  to optimization policy.
- **Concurrent / contemporary:** [[bharadhwaj-2024-track2act]] uses
  rigid-transform fitting (same idea as 3DFlowAction's SVD step) but
  with full 2D track field + residual policy.
- **Direct successor:** [[kuang-2026-dex4d]] — adopts the same
  "generated video → object 3D point tracks → policy condition"
  pipeline, but trains a sim-to-real **dexterous** policy rather than
  doing optimization, and uses CoTracker3 + relative depth calibration.
  NovaFlow (cited but not yet ingested) is positioned by Dex4D as the
  closest 3D-flow baseline.
- **VLM-as-checker:** Same pattern as ReKep (GPT-4o constraint code)
  and many recent VLA verifier setups.
- **Point tracker:** [[cotracker3]].
- **Concept page:** [[point-tracks-as-manipulation-interface]].

## Citation

```
Zhi, H., Chen, P., Zhou, S., Dong, Y., Wu, Q., Han, L., & Tan, M.
(2025). 3DFlowAction: Learning Cross-Embodiment Manipulation from 3D
Flow World Model. arXiv preprint arXiv:2506.06199.
```
