---
type: source
source_type: paper
title: "PointWorld: Scaling 3D World Models for In-The-Wild Robotic Manipulation"
authors: [Huang, Wenlong; Chao, Yu-Wei; Mousavian, Arsalan; Liu, Ming-Yu; Fox, Dieter; Mo, Kaichun; Fei-Fei, Li]
year: 2026
venue: "arXiv preprint (Jan 2026)"
url: https://arxiv.org/abs/2601.03782
raw_path: papers/2601.03782v1.pdf
status: ingested
tags: [manipulation, world-model, 3d-point-tracking, dense-tracking, point-transformer, mpc, dinov3, foundation-model, cross-embodiment]
sources: []
related:
  - "[[pointworld]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[lee-2026-mu0]]"
  - "[[bharadhwaj-2024-track2act]]"
  - "[[xu-2024-im2flow2act]]"
  - "[[zhi-2025-3dflowaction]]"
  - "[[kuang-2026-dex4d]]"
  - "[[kim-2026-pri4r]]"
  - "[[cmp-point-track-manipulation]]"
  - "[[wang-2025-vggt]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[anon-2026-stride]]"
created: 2026-07-09
updated: 2026-07-09
---

# PointWorld

## TL;DR

**Large pretrained 3D dynamics model** for robot manipulation.
Represents both **state and action as 3D point flows in the same
space**: state = scene point cloud from RGB-D, action = robot surface
points propagated via forward kinematics from URDF. Trained on
**~2M trajectories / 500 hours** (DROID real + BEHAVIOR-1K sim,
custom 3D-annotation pipeline). **PTv3 backbone (50M–1B params) +
DINOv3 features + chunked 10-step prediction @ 0.1s/step**. Deployed
via **MPPI** sampling-based planner on real Franka — zero-shot rigid
pushing, deformable / articulated / tool-use manipulation from
**single RGB-D image**, no demos, no fine-tuning. Predicts what
*happens* given a candidate action; MPPI searches actions. Distinct
role from language-conditioned trace generators (Track2Act,
[[mu0]], 3DFlowAction) — this is a learned **physics simulator**,
not a plan generator.

## Why it matters

Distinct paradigm within [[point-tracks-as-manipulation-interface]]:

1. **Action-conditioned dynamics WM, not language-conditioned trace
   generator.** Track2Act / Im2Flow2Act / 3DFlowAction / Dex4D / µ0
   answer "what motion achieves this task?" PointWorld answers "given
   this action, what happens to the scene?" Two different questions,
   two different downstream stacks (trace → policy vs. dynamics →
   MPPI).
2. **Shared state-action representation in 3D.** Both state and
   action live as 3D point flows. State from RGB-D back-projection;
   action from URDF forward kinematics (robot surface points).
   Consequence: single point cloud (scene ∪ robot) fed to PTv3
   backbone — action geometry directly conditions scene dynamics via
   spatial proximity, not via a separate encoder.
3. **Embodiment-agnostic action space via URDF.** Any robot with a
   URDF → surface points → same interface. Trained jointly on
   single-arm Franka (DROID) + bimanual humanoid (BEHAVIOR-1K).
   Positive transfer across drastically different embodiments.
4. **Real-time (0.1 s / chunk) via chunked-not-autoregressive
   prediction.** MPPI can evaluate 30-step candidate trajectories in
   3 forward passes. Diffusion-based video WMs need seconds.
5. **Scaling roadmap distilled.** GBND → PointNet → PTv3; L2 →
   movement-weighted + uncertainty + Huber; DINOv2 → DINOv3; 50M →
   1B. **Log-linear scaling in both data and model size.** Rare
   published scaling curve for 3D dynamics.
6. **Custom 3D annotation pipeline unlocks DROID for 3D.** Sensor
   depth + dataset extrinsics were insufficient. Their pipeline =
   FoundationStereo depth + VGGT-init + robot-alignment optimization
   + CoTracker3 → recovers 60% of DROID (~200 hours) with reliable
   3D flows. Reusable infrastructure for the whole community.

## Key claims

### Scaling roadmap (Figure 7, DROID test ℓ2 mover)

| Change | ℓ2 mover | Notes |
|---|---|---|
| GBND baseline | 0.0386 | Graph-based neural dynamics (prior SOTA) |
| → PointNet | 0.0370 | |
| → PTv3 | 0.0348 | Modern PT-v3 backbone |
| + uncertainty | 0.0348 | Aleatoric regularization |
| + movement weighting | 0.0350 | (small step back before Huber) |
| + Huber loss | 0.0342 | |
| + DINOv2 ViT-S | 0.0335 | |
| + DINOv2 ViT-L | 0.0332 | |
| + DINOv3 ViT-L | 0.0334 | |
| + multi-layer features | 0.0331 | |
| + scale 50M → 132M | 0.0324 | |
| + 411M | 0.0315 | |
| + 1B | 0.0312 | **Final** |

**~20% error reduction stack-to-stack** from baseline to final.

### Scaling laws (Figure 9)

Log-linear ℓ2 mover vs both:
- Data fraction (5% → 100%): monotonic.
- Model size (50M → 1B): monotonic.

Analogous to LM / vision scaling laws — first published version for
3D world modeling.

### Backbone comparison (Table 1, latency in ms)

| Backbone | Params | Latency (ms) | ℓ2 mover |
|---|---|---|---|
| GBND | 1.00× | 13.46 | 0.0390 |
| PointNet | 1.03× | 5.93 | 0.0369 |
| PointNet++ | 1.07× | 327.08 | 0.0368 |
| SparseConv | 33.31× | 17.70 | 0.0396 |
| Transformer | 41.06× | 30.43 | 0.0339 |
| PTv3-50M | 49.14× | 59.60 | 0.0331 |
| PTv3-132M | 127.22× | 69.60 | 0.0324 |
| PTv3-411M | 398.67× | 102.47 | 0.0315 |
| **PTv3-1B** | 957.71× | **123.65** | **0.0312** |

PTv3 = default. Scales to 957× GBND params at modest memory/latency.

### Action representation ablation (Figure 11)

| Action rep | ℓ2 mover DROID | ℓ2 mover B1K |
|---|---|---|
| Joint positions (low-dim) | ~0.038 | ~0.021 |
| EEF pose (low-dim) | ~0.037 | ~0.020 |
| Full robot 500pts (whole-body flow) | ~0.038 | ~0.017 |
| Full robot 3000pts | ~0.036 | ~0.015 |
| **Gripper only (~300-500 pts)** | **~0.035** | **~0.013** |

Sparse gripper-only flows **beat** dense whole-body flows on real
data (whole-body dilutes learning signal) AND **beat** low-dim EEF /
joint (better contact-geometry conditioning).

### Chunked prediction beats autoregressive (Figure 12)

Chunked-in-training + chunked-in-inference = lowest drift across
1–10 step horizons. Autoregressive teacher-forcing (train) →
self-feeding (test) degrades badly. Also **10× more compute-
efficient**: single forward pass vs. 2–10 for AR.

### Generalization (Table 2, ℓ2 mover)

D = DROID, B = BEHAVIOR-1K, H = held-out real DROID lab.

| Split | Zero-shot | Finetuned (5% steps) |
|---|---|---|
| In-domain D→D | 0.0315 | — |
| In-domain B→B | 0.0087 | — |
| Cross D→B | 0.1460 | 0.0107 |
| Cross B→D | 0.0558 | 0.0378 |
| Held-out D→H | 0.0305 | 0.0271 |
| Held-out B→H | 0.0531 | 0.0299 |
| D+B → H | 0.0300 | 0.0272 |
| Scratch on H | 0.0293 | — |

**Zero-shot D→H matches scratch specialist.** Finetune with **20×
fewer updates** surpasses scratch. Real→sim transfer is asymmetric
better than sim→real. Real+sim co-training helps zero-shot on H.

### Partial observability (Figure 13)

Random-camera-count training generalizes best. Fixed 3-cam training
+ 1-cam eval degrades; random-cam training smooths this.

### Real robot MPC deployment (Figure 8, Franka + RealSense D435 +
FoundationStereo depth)

Zero-shot, no demos, no finetune. Tasks + SR:
- **Pushing:** tissue box 100%, book 70%.
- **Deformable:** scarf 80%, pillow 40%.
- **Articulated:** microwave 30%, drawer 90%.
- **Tool use:** duster 60%, broom 60%.

Confirms pretrained 3D dynamics capture material properties,
articulation, shape completion, and object-object dynamics implicitly.

## Methods

### Problem formulation

Model dynamics as `F_θ : S × A → S` with **chunked H-step
prediction**:
```
F_θ^H : (s_t, a_{t:t+H-1}) → s_{t+1:t+H}
```
`H = 10 steps × 0.1s = 1 s` horizon per call. Amortizes compute,
reduces drift.

### State representation

`s_t = { (p_{t,i}, f_i^S) }_{i=1}^{N_S}` — scene point flow. Positions
`p_{t,i} ∈ R³` from back-projected RGB-D; features `f_i^S ∈ R^{D_S}`
time-constant from **frozen DINOv3** (multi-layer, projected via
calibrated camera).

Robot pixels **masked** using forward kinematics + URDF; remaining
pixels back-projected to obtain scene points.

### Action representation

`a_{t+k} = { (r_{t+k,j}, f_{t+k,j}^R) }_{j=1}^{N_R}` — robot point
flow. Sample robot surface points once; propagate via forward
kinematics from joint config sequence `{q_{t+k}}`. Time-varying
features via temporal embedding.

Ablation confirms **sample gripper points only** (300–500 pts) beats
whole-body sampling — most contact happens at gripper, rest dilutes
signal.

Key benefit: URDF is known a priori → action = **fully observable**
geometry propagated deterministically, unlike scene points which are
partially observable.

### Architecture

```
[scene points (DINOv3 feats)] ⊕ [robot points (temporal embed)]
        ↓ concatenated single point cloud
        PTv3 backbone (50M–1B params, U-Net hierarchy)
        ↓ per-point features
        Shared MLP head
        ↓
        Per-point 3D displacements for each of H timesteps
```

Chunked output → per-step Cartesian displacement for every scene
point.

### Training objective

Two challenges: (i) most points static → naive L2 has sparse signal;
(ii) noisy real data → need robustness.

```
L = Σ_{k,i} w_{k,i} · ρ_δ(P̂_{t+k,i} - P_{t+k,i}) · e^{-s_{k,i}} + s_{k,i}
```

- `ρ_δ` = elementwise Huber loss on 3D residual.
- `w_{k,i} = m_{k,i} / Σ m_{k,i}` = movement weight; `m_{k,i} = σ(κ(δ_{k,i} - τ))` where `δ` = GT displacement norm. Focuses loss on moving points.
- `s_{k,i}` = predicted log-variance → **aleatoric uncertainty regularization**. Prevents overfitting noisy pseudo-labels. Emerges to capture action-conditioned uncertainty (e.g., high uncertainty along cloth edges after robot release).

Points flagged not-visible by CoTracker3 → excluded from supervision.

### 3D annotation pipeline (for DROID)

Sensor depth + DROID extrinsics = insufficient. Custom 3-stage:

1. **FoundationStereo** depth → replaces sensor depth (better at
   close manipulation range).
2. **VGGT-initialized camera extrinsics** → refined via optimization
   that aligns robot depth to known robot mesh from URDF. Median
   error 1.8 cm / 1.9°.
3. **CoTracker3** 2D → lift to 3D using refined depth + extrinsics →
   propagate visibility labels.

Recovers reliable 3D flows for **60% of DROID** (~200 hours). Filters
scenes with depth reprojection loss > 0.10.

### MPPI-based action inference (§3.2)

```
Given: RGB-D → s_0; task-relevant scene points I_task with targets {g_i}
Loop MPPI iterations:
  Sample K action perturbations ℓ_{1:K} via time-correlated cubic-spline noise
  For each sample:
    Construct robot point-flow action a_{1:T}^{(ℓ)} via FK
    Roll out PointWorld: s_{1:T}^{(ℓ)} = F_θ^T(s_0, a_{1:T}^{(ℓ)})
    Compute cost J^{(ℓ)} = Σ_k [c_task(s_k) + c_ctrl(E_k)]
    where c_task(s_k) = (1/|I_task|) Σ_{i ∈ I_task} ||p_{k,i} - g_i||²
  Update nominal trajectory as weighted average with ω_ℓ ∝ exp(-J^{(ℓ)}/β)
Execute first step; replan
```

Task-relevant scene points → specified via GUI or VLM. Target
positions `{g_i}` → user- or VLM-specified. Cost purely geometric →
applies rigid / deformable / articulated uniformly.

## Results

See "Key claims" — scaling roadmap, backbone comparison, action-rep
ablation, chunked-vs-AR, generalization table, MPC SR.

## Limitations / open questions

Authors do not exhaustively enumerate. Implicit from results:

- **Zero-shot cross-domain still poor.** D→B ℓ2 mover 0.146 (12×
  in-domain B→B). Real↔sim gap remains. Finetune closes it with 5%
  data.
- **Simulation-only pretraining underperforms scratch** on real
  held-out — matches Sim2Real convention that real-data pretraining
  is more transferable.
- **Requires calibrated RGB-D + accurate depth.** Deployment uses
  FoundationStereo — a strong depth model, not always available on
  arbitrary robots.
- **Task-relevant point selection is a bottleneck.** MPC needs
  `I_task` + goal positions `{g_i}`. GUI or VLM specification. Not
  fully autonomous.
- **Deformable / articulated / tool-use SR is medium** (30–80%).
  Rigid pushing works; contact-rich long-horizon is harder.
- **No comparison against language-conditioned trace generators** —
  Track2Act, µ0, 3DFlowAction operate in a very different regime
  (VLM plan → policy), so head-to-head is difficult. But PointWorld
  claims "zero-shot in-the-wild without demos" which µ0 also claims
  — a direct benchmark comparison would inform which paradigm wins
  for what task class.
- **URDF requirement** limits to robots with clean kinematic models.
  Humans (Ego4D-scale) not directly usable.
- **10-step (1s) horizon.** Longer-horizon dynamics not tested;
  MPPI's short receding horizon compensates.

## Connections

- **Cited by [[lee-2026-mu0]]** (Huang et al. 2026, CVPR 2026) as
  prior 3D WM. µ0's framing paragraph explicitly contrasts pixel-
  space WMs with intermediate-representation WMs; PointWorld sits in
  the latter camp.
- **Complementary role to µ0** — PointWorld = action-conditioned
  dynamics (what will happen?); µ0 = language-conditioned trace
  generation (what should happen?). Combining them = MPC over µ0's
  trace target using PointWorld as the simulator. Neither paper
  attempts this fusion.
- **Same family (concept):** [[point-tracks-as-manipulation-interface]]
  — but new sub-axis introduced (see concept page updates).
- **Backbone shared with [[anon-2026-stride]]:** both use **PTv3**
  for point-cloud processing. STRIDE flags PTv3's aggregate-at-once
  window as a streaming limitation — same trade-off applies here.
  PTv3 now has 2 wiki sources; org-tool page candidate.
- **Uses [[karaev-2024-cotracker3]]** to build DROID 2D tracks →
  lift to 3D via refined depth. Reuses [[wang-2025-vggt]] for
  camera extrinsics initialization. Reinforces both as
  infrastructure primitives.
- **Adjacent baselines cited but not ingested:**
  - **GBND** (Ai et al. 2025 Science Robotics) — graph-based neural
    dynamics; the prior SOTA baseline PointWorld beats.
  - **FoundationStereo** (Wen et al. 2024) — depth estimator.
  - **MPPI** (Williams et al. 2017) — planner.
  - **DROID** (Khazatsky et al. RSS 2024) — real dataset.
  - **BEHAVIOR-1K** (Li et al. 2023) — sim dataset.
  - **VoxPoser / ReKep** (Huang et al. — same first author, prior
    work; VLM-based waypoint / constraint methods).
- **Author cluster:** Wenlong Huang lead (Stanford / Fei-Fei group;
  also VoxPoser, ReKep). Yu-Wei Chao × Arsalan Mousavian × Ming-Yu
  Liu × Dieter Fox × Kaichun Mo (NVIDIA robotics). Li Fei-Fei
  co-senior (Stanford). First Stanford Fei-Fei paper in wiki (µ0 is
  UMD; Im2Flow2Act is Stanford Shuran Song).
- **DINOv3 tag** now has a first primary source in the wiki (PTv3 +
  DINOv3 feature stack).

## Citation

```
Huang, W., Chao, Y.-W., Mousavian, A., Liu, M.-Y., Fox, D., Mo, K.,
& Fei-Fei, L. (2026). PointWorld: Scaling 3D World Models for
In-The-Wild Robotic Manipulation. arXiv preprint arXiv:2601.03782.
```
