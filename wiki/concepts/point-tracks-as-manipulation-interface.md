---
type: concept
title: Point Tracks as the Manipulation Interface
status: growing
tags: [manipulation, point-tracking, flow, cross-embodiment, vla, imitation-learning]
sources:
  - "[[bharadhwaj-2024-track2act]]"
  - "[[xu-2024-im2flow2act]]"
  - "[[zhi-2025-3dflowaction]]"
  - "[[kuang-2026-dex4d]]"
  - "[[kim-2026-pri4r]]"
  - "[[lee-2026-mu0]]"
  - "[[huang-2026-pointworld]]"
related:
  - "[[point-tracking]]"
  - "[[3d-point-tracking]]"
  - "[[vla]]"
  - "[[track2act]]"
  - "[[im2flow2act]]"
  - "[[3d-flow-action]]"
  - "[[dex4d]]"
  - "[[pri4r]]"
  - "[[mu0]]"
  - "[[pointworld]]"
  - "[[cmp-point-track-manipulation]]"
created: 2026-06-27
updated: 2026-07-09
---

# Point Tracks as the Manipulation Interface

## Definition

A growing family of manipulation methods that treats **point tracks
(2D or 3D)** as the *intermediate representation* between perception
and control: scene understanding becomes "what tracks will I see," and
control becomes "what actions reproduce those tracks." The interface
sits between cross-embodiment / web-video planning (where there are no
actions) and robot-specific execution (where there are no labels of
the right form).

## Why it matters

The 2022–2024 robotics literature has tried many things in this slot:

- **Raw video** as plan (UniPi, AVDC) — expensive to generate, prone
  to artifacts, hard to extract actions from.
- **Image-space affordances** (Where2Act, VRB) — static; lose motion.
- **Hand-object masks** (Bharadhwaj ICRA 2024) — lose object pose.
- **Object meshes / 6-DoF pose** — need clean object models, hard to
  estimate.

Point tracks hit a useful middle: **expressive** (rotation,
articulation, deformation), **embodiment-agnostic** (object-only),
**cheap to compute** (off-the-shelf TAP models), **lift-able to 3D**
(depth + projection), and crucially **abstract out appearance** so a
human demo and a robot rollout look the same in interface space.

The TAP-line ([[pips]] → [[tapir]] → [[cotracker3]] →
[[zhang-2025-tapip3d]] → [[xiao-2025-spatialtracker-v2]]) is the
infrastructure that made this practical — early manipulation flow work
existed but was either dense optical flow (limited to small motions)
or hand-engineered keypoints (limited to known objects).

## Variants

This page traces seven papers in the wiki that adopt the interface,
each at a different design coordinate:

| Method | Year | Track dim | Plan source | Action policy | Real-robot data needed |
|---|---|---|---|---|---|
| [[track2act]] | 2024 | 2D full-trajectory | DiT trained on web video | Per-step PnP transform + **residual BC policy** | ~400 Spot demos |
| [[im2flow2act]] | 2024 | 2D object-only flow | AnimateDiff trained on human videos | Sim-trained diffusion policy + temporal alignment | **None** |
| [[3d-flow-action]] | 2025 | 3D (u, v, z, vis) | AnimateDiff on ManiFlow-110k | **Optimization** (no learned policy, no action labels) | None (uses 30 human demos for fine-tuning per task) |
| [[dex4d]] | 2026 | 3D object tracks | Wan2.6 video gen → CoTracker3 + relative-depth | **Sim-to-real RL** (Anypose-to-Anypose, dexterous) | None |
| [[pri4r]] | 2026 | 3D point tracks | (not used at inference) | Auxiliary supervision for VLA backbone | None at inference; pseudo-labels at training |
| [[mu0]] | 2026 | 3D semantic keypoints (B-spline) | Frozen VLM-conditioned **world model** | **Action expert** on frozen trace features | Only for AE fine-tuning |
| [[pointworld]] | 2026 | Dense 3D scene point flow (per-pixel) | **Action-conditioned dynamics WM** (learned physics simulator) | **MPPI** on top of WM rollouts | None (zero-shot in-the-wild) |

### Four axes that organize them

1. **What role do tracks play at inference?**
   - **Conditioning input:** Track2Act, Im2Flow2Act, 3DFlowAction,
     Dex4D, NovaFlow. Need tracker + (often) flow generator at
     deployment. Model consumes decoded tracks as policy input.
   - **Privileged supervision (discard):** Pri4R. Train with tracks,
     discard head at test time.
   - **World-model features (frozen intermediate):** µ0. Trace
     prediction pretrains a frozen motion prior; downstream policy
     consumes intermediate features from a single partial denoising
     step, not the decoded track.
   - **Action-conditioned dynamics simulator:** PointWorld. Model
     predicts *what happens* given a candidate action; MPPI searches
     actions on top. Different question from all others (dynamics,
     not plan generation).
2. **What answers "what should happen?"**
   - **Language-conditioned trace generator:** Track2Act / Im2Flow2Act
     / 3DFlowAction / Dex4D / µ0. Given task + observation → predict
     future track directly.
   - **Action-conditioned dynamics WM + planner:** PointWorld. Given
     action + observation → predict future state; separate outer
     planner (MPPI) finds actions.
   - **Neither (implicit in VLA backbone):** Pri4R.
3. **Heuristic action extraction vs. learned policy vs. MPC.**
   - **Heuristic / optimization:** Track2Act (open-loop), 3DFlowAction.
   - **Learned:** Im2Flow2Act (sim BC), Dex4D (sim-to-real RL),
     Track2Act-residual, µ0 (action expert on frozen features).
   - **MPC / sampling-based planner:** PointWorld (MPPI).
4. **Fixed grid vs. semantic keypoints vs. dense per-pixel.**
   - **Fixed grid / dense sparse:** Track2Act, Im2Flow2Act,
     3DFlowAction, TraceGen. Wastes budget on backgrounds.
   - **Semantic keypoints:** µ0 (DINOv2 entity clusters + budget-per-
     entity).
   - **Dense per-pixel:** PointWorld. Every pixel in RGB-D →
     back-projected point → flow prediction. Highest resolution, most
     compute; enables contact reasoning at articulation joints.

Different combinations of these choices give the design space the line
has been exploring.

## Evidence across sources

### Track2Act establishes the recipe (2024)

[[bharadhwaj-2024-track2act]] is the first to train a *generative*
model that predicts **full future point trajectories** from web video
and convert them to robot actions. The residual policy is a concession
— ~400 demos to close the embodiment gap. Wins decisively on
compositional + type generalization tiers vs. plain BC and prior
affordance/video baselines.

### Im2Flow2Act decouples plan from execution data (2024 CoRL)

[[xu-2024-im2flow2act]] removes the real-robot residual policy
entirely by training the flow generator and the action policy on
*different* data sources (human videos vs. simulated play). Two
defensive design choices make this work: object-only flow (not grid)
and complete task flow (not next-step). 81% real-world SR with zero
real-robot training data.

### 3DFlowAction adds the third dimension (2025)

[[zhi-2025-3dflowaction]] solves Im2Flow2Act's known 2D failure mode
(out-of-plane rotation, z-axis precision) by emitting 4-channel
(u, v, z, vis) flow directly. Introduces ManiFlow-110k as a reusable
cross-source pretraining corpus and swaps the learned policy for an
optimization policy that needs no action labels. GPT-4o serves as a
closed-loop verifier of generated flow.

### Dex4D ports to dexterous (2026)

[[kuang-2026-dex4d]] is the first to apply the interface to
**dexterous (22-DoF) hands**. The action policy is sim-to-real RL
(Anypose-to-Anypose), conditioned via the novel **Paired Point
Encoding**. Wan2.6 generates the task video; CoTracker3 + median-
rescaled relative depth produces object-centric 3D tracks. From the
same Tulsiani group as Track2Act — direct lineage.

### Pri4R inverts the role (2026)

[[kim-2026-pri4r]] is the philosophical outlier: 3D point tracks are
not a deployment-time conditioning signal but a **training-time
privileged supervision target**. The auxiliary head reshapes the VLA
backbone to encode 4D dynamics and is then discarded. Wins +13.2% on
RoboCasa with zero inference overhead and shows 3D tracks beat goal-
only, 2D-track, and depth supervision targets.

### µ0 turns traces into a world model (2026)

[[lee-2026-mu0]] (UMD + SNU, Jun 2026) frames trace prediction as a
**video world model** rather than a plan generator. Pretrain a
SmolVLM2 + Trace Expert to denoise B-spline control points for
semantic (DINOv2-selected) keypoints via flow matching; freeze; feed
intermediate trace-denoising features to a small **action expert**.
Fixes three limitations of prior work directly: (1) fixed-grid →
semantic keypoints via DINO clusters; (2) camera + object motion
conflation → globally aligned 3D via chunk-wise reconstruction;
(3) episode-level captions → event-level chunks via
acceleration-valley boundaries. Data engine (**TraceExtract**) scales
curation ~8× over TraceGen. Frozen µ0 + AE beats π₀ (25.25 → 30.25%
avg RoboCasa365 SR) and beats TraceGen + AE by +7.25 pts sim / +10
pts real — despite **zero action supervision at pretraining**.

### PointWorld turns point flows into a physics simulator (2026)

[[huang-2026-pointworld]] (Stanford + NVIDIA, Jan 2026) is the
outlier along the "role of tracks" axis. Rather than predicting
task-directed trajectories from language, PointWorld predicts
**action-conditioned dynamics**: `(scene point cloud, robot action
point flow) → next scene point cloud`. Shared 3D-point-flow
representation for both state and action means the URDF-derived robot
surface points concatenate with scene points into a single point
cloud processed by **PTv3** (Point Transformer v3, 50M–1B params) +
frozen **DINOv3** features. Chunked 10-step prediction @ 0.1 s → real
time on MPPI. Deployed on real Franka via MPPI planner for
zero-shot rigid pushing, deformable manipulation, articulated objects,
tool use — from a single RGB-D image, no demos, no fine-tuning.
**Published scaling laws**: log-linear improvement across 50M → 1B
params and 5% → 100% data. Dense per-pixel prediction (vs. µ0's
sparse semantic keypoints) enables contact reasoning at
articulation joints and deformable boundaries. First member of the
family that is fundamentally a **learned physics simulator**, not a
trajectory generator.

## Contested points

- **Four distinct roles for tracks at inference:**
  - **Conditioning input** (Track2Act, Im2Flow2Act, 3DFlowAction,
    Dex4D) — tracker + flow generator required at deployment.
  - **Privileged supervision** (Pri4R) — tracks discarded at test
    time; policy is a stock VLA at inference.
  - **World-model features** (µ0) — frozen trace-expert
    intermediate features feed the action expert; decoded track
    is not consumed at inference, but denoising computation is.
  - **Action-conditioned dynamics simulator** (PointWorld) — model
    is a learned physics engine; MPPI wrapper searches actions by
    rolling out the WM under candidate action sequences. Tracks
    (dense scene flows) are the *output* of every WM step; the
    outer planner consumes them as MPC state.
- **2D vs. 3D.** 3DFlowAction and Dex4D both argue 2D is fundamentally
  insufficient for rotation / z-axis tasks; Im2Flow2Act's pouring
  failure is consistent with this. But 3D adds depth-estimation noise
  as a new failure mode.
- **Learned vs. optimization action policy.** Im2Flow2Act argues
  learned policies are necessary for deformable / articulated objects
  (heuristics fail). 3DFlowAction argues that with good 3D flow + IK,
  an optimization policy suffices and avoids needing action labels.
  Dex4D argues that high-DoF / contact-rich tasks need learned RL
  policies for hand reactivity. µ0 argues that video-only world-model
  pretraining is enough — beats action-labeled π₀ on RoboCasa365.
- **Fixed grid vs. semantic keypoints.** µ0's TraceExtract explicitly
  challenges the fixed-grid choice made by every prior member of the
  family. Ablations (µ0's Appendix E.1) show semantic + entity-budget
  sampling helps. Whether re-training e.g. Track2Act with semantic
  keypoints would erase µ0's other gains is untested.

## Open questions

- **Can the privileged-supervision form (Pri4R) match the conditioning
  form (Dex4D) or the world-model form (µ0) on hard tasks?** The
  benchmarks are different (Pri4R on LIBERO/RoboCasa parallel-jaw
  vs. Dex4D on real dexterous vs. µ0 on RoboCasa365 + UR3), so no
  clean head-to-head exists yet.
- **Long-horizon multi-object manipulation?** All six papers are
  short-horizon, single-object. The most ambitious is Dex4D's
  Anypose-to-Anypose which composes via the high-level video planner.
- **Can the flow generator be replaced by direct VLM
  flow prediction?** Pri4R hints yes (the VLA backbone learns
  dynamics implicitly); explicit flow generators (Wan2.6 in Dex4D,
  AnimateDiff in 3DFlowAction) are still the SOTA for plan quality.
  µ0 sits in between — its VLM-conditioned trace expert *is* a
  flow predictor, just over compact spline control points instead of
  waypoints.
- **Tracker dependence.** All six papers ride the TAP-line — when
  the trackers improve (faster, more robust to occlusion), so does
  this entire family. Dex4D explicitly calls this out as a
  bottleneck. µ0's TraceExtract has its own reconstruction /
  tracking pipeline; a feed-forward 4D backbone
  ([[point4d]] / [[stride]] / [[v-dpm]]) as the perception side is
  the natural next combination.

## See also

- [[cmp-point-track-manipulation]] — head-to-head comparison table.
- [[point-tracking]] — the underlying TAP problem.
- [[3d-point-tracking]] — the 3D variant.
- [[vla]] — the policy family Pri4R augments.
