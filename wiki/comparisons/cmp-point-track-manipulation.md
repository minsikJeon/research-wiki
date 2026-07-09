---
type: comparison
title: Comparison of Point-Track-Based Manipulation Methods
status: growing
tags: [manipulation, point-tracking, comparison, vla, imitation-learning]
sources:
  - "[[bharadhwaj-2024-track2act]]"
  - "[[xu-2024-im2flow2act]]"
  - "[[zhi-2025-3dflowaction]]"
  - "[[kuang-2026-dex4d]]"
  - "[[kim-2026-pri4r]]"
  - "[[lee-2026-mu0]]"
  - "[[huang-2026-pointworld]]"
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[track2act]]"
  - "[[im2flow2act]]"
  - "[[3d-flow-action]]"
  - "[[dex4d]]"
  - "[[pri4r]]"
  - "[[mu0]]"
  - "[[pointworld]]"
created: 2026-06-27
updated: 2026-07-09
---

# Comparison of Point-Track-Based Manipulation Methods

Synthesis across the 7 sources currently in the wiki that use point
tracks as the manipulation interface. **Caveat:** results not directly
comparable across papers — different robots (Spot / UR5e / Franka /
XTrainer / xArm6 + LEAP / OMY-F3M / UR3), tasks, train data, and
evaluation protocols. Treat as a navigation table.

## High-level design table

| Method | Year | Track dim | Tracks at inference? | Plan source | Action policy | Embodiment | Real-robot data |
|---|---|---|---|---|---|---|---|
| [[track2act]] | 2024 | 2D full-traj | yes | DiT denoiser (web video) | Per-step PnP + residual BC | Spot mobile manip | ~400 demos |
| [[im2flow2act]] | 2024 | 2D object flow | yes (full task once) | AnimateDiff (human video) | Sim-trained diffusion + alignment | UR5e (parallel-jaw) | **none** |
| [[3d-flow-action]] | 2025 | 3D (u,v,z,vis) | yes (re-plan via GPT-4o) | AnimateDiff on ManiFlow-110k | **Optimization** (no labels) | Franka / XTrainer | none (30 human demos per task) |
| [[dex4d]] | 2026 | 3D obj tracks | yes (closed-loop online) | Wan2.6 → CoTracker3 + rel-depth | **Sim-to-real RL** (AP2AP) | **xArm6 + LEAP (dexterous)** | **none** |
| [[pri4r]] | 2026 | 3D pre-sampled pts | **no** (privileged train only) | n/a (auxiliary signal) | Stock VLA at inference | π₀ / π₀.₅ / OpenVLA-OFT | none at inference |
| [[mu0]] | 2026 | 3D semantic keypoints (B-spline) | **features** (frozen WM step) | VLM+Trace Expert (video-only pretrain) | Action expert on frozen trace features | UR3 (parallel-jaw) + RoboCasa365 sim | AE fine-tune only |
| [[pointworld]] | 2026 | Dense per-pixel 3D scene flow | **WM rollouts** (action-conditioned dynamics) | n/a (WM is a simulator, not a plan generator) | **MPPI** on WM rollouts | Franka (single-arm) + BEHAVIOR-1K sim | **none** |

## Headline numbers (cross-paper, take with salt)

### Track2Act on Spot (4-tier generalization, success rate %)

| Setting | BC | Affordance | HOMask | **Track2Act** |
|---|---|---|---|---|
| Standard Gen | 20 | 30 | 40 | **60** |
| Compositional | 0 | 10 | 25 | **55** |
| Type Gen | 0 | 5 | 20 | **40** |

### Im2Flow2Act on UR5e, real-world (lang-conditioned, SR %)

| Task | Im2Flow2Act |
|---|---|
| Pick&Place | 90 |
| Pouring | 80 |
| Drawer | 85 |
| Cloth Folding | 70 |
| **Avg** | **81** |

### 3DFlowAction on Franka/XTrainer (4 tasks, SR %)

| Method | Pour | Pen | Hang | Drawer | Total |
|---|---|---|---|---|---|
| AVDC (video WM) | 1/10 | 2/10 | 0/10 | 5/10 | 20.0 |
| ReKep (constraint code) | 2/10 | 1/10 | 3/10 | 2/10 | 20.0 |
| Im2Flow2Act* | 2/10 | 2/10 | 0/10 | 6/10 | 25.0 |
| PI0 (VLA) | 5/10 | 5/10 | 4/10 | 6/10 | 50.0 |
| **3DFlowAction** | **6/10** | **7/10** | **5/10** | **10/10** | **70.0** |

### Dex4D on dexterous tasks (real world, SR)

| Method | LiftToy | Broc | Meat | Pour | Total |
|---|---|---|---|---|---|
| NovaFlow-CL | 4/10 | 3/10 | 3/10 | 0/10 | 25.0 |
| **Dex4D** | **6/10** | **4/10** | **5/10** | **4/10** | **47.5** |

### Pri4R on VLAs (RoboCasa avg SR %, +∆ over base)

| Base VLA | Base SR | + Pri4R | ∆ |
|---|---|---|---|
| π₀ | 38.8 | 42.2 | +3.4 |
| π₀.₅ | 52.9 | 57.0 | +4.1 |
| **OpenVLA-OFT** | **33.1** | **46.3** | **+13.2** |

### µ0 vs baselines — RoboCasa365 sim (Table 2, 8 tasks, avg SR %)

| Method | Category | Avg SR |
|---|---|---|
| Diffusion Policy | scratch | 22.75 |
| π₀ | action-labeled VLA | 25.25 |
| **π₀.₅** | action-labeled VLA | **42** |
| TraceGen + AE | video-only WM | 23 |
| **µ0 + AE** | video-only WM | **30.25** |

µ0 beats π₀ (+5.0) with zero action supervision at pretraining; π₀.₅ wins
overall (not data-matched).

### µ0 vs baselines — Real UR3 (Fig 6, 3 tasks, avg SR %)

| Method | Avg SR |
|---|---|
| π₀ | 71.7 |
| VLM + AE (no trace) | 73.3 |
| π₀.₅ | 80 |
| TraceGen + AE | 81.7 |
| **µ0 + AE** | **91.7** |

µ0 wins overall: +18.4 over VLM+AE (isolates trace expert
contribution), +10 over TraceGen, +11.7 over π₀.₅, +20 over π₀.

### PointWorld — DROID test ℓ2 mover (Figure 7 scaling roadmap)

| Config | ℓ2 mover ↓ |
|---|---|
| GBND baseline | 0.0386 |
| PTv3 backbone | 0.0348 |
| + Huber loss | 0.0342 |
| + DINOv3 multi-layer | 0.0331 |
| **+ 1B params** | **0.0312** |

Log-linear scaling in both model size (50M → 1B) and data (5% → 100%).

### PointWorld — Real Franka MPC zero-shot SR (Figure 8)

| Task class | Task | SR |
|---|---|---|
| Pushing | Tissue box | 100% |
| Pushing | Book | 70% |
| Deformable | Scarf | 80% |
| Deformable | Pillow | 40% |
| Articulated | Drawer | 90% |
| Articulated | Microwave | 30% |
| Tool use | Duster | 60% |
| Tool use | Broom | 60% |

Zero-shot, no demos, no fine-tuning. Single RGB-D input.

### µ0 trace-forecasting (Table 1, best-of-5, T=32)

| Method | 2D top5-ADE | 3D top5-ADE | Latency |
|---|---|---|---|
| Track2Act | 0.293 | — | 0.85s |
| Hamster | 0.256 | — | 14.4s |
| Gemini-3.1-pro | 0.253 | — | 78s |
| GPT-5.5 | 0.272 | — | 38s |
| TraceGen | — | 0.325 | 1.20s |
| 3DFlowAction | — | 0.630 | 3.38s |
| Dream2Flow | — | 0.336 | 106.8s |
| **µ0** | **0.227** | **0.239** | **0.29s** |

Best top5 across both modalities at 3–200× lower latency.

## Where each method wins

- **Best for unseen scenes & web-video-only training (parallel-jaw mobile):**
  [[track2act]] — Spot in real kitchens/offices.
- **Best zero-real-robot-data parallel-jaw at small scale:**
  [[im2flow2act]] — 81% across rigid/articulated/deformable.
- **Best for 3D manipulation with optimization (no learned policy):**
  [[3d-flow-action]] — 70% on rotation-heavy tasks.
- **Best for sim-to-real dexterous manipulation:**
  [[dex4d]] — first to apply the interface to 22-DoF hands.
- **Best for zero-overhead inference (existing VLA + free SR gain):**
  [[pri4r]] — +13.2% RoboCasa with no architectural changes at
  inference.
- **Best for video-only pretraining as a reusable motion prior + fastest
  trace prediction:** [[mu0]] — 91.7% avg real-world SR on UR3, beats
  action-labeled π₀ / π₀.₅; 0.29s trace inference.
- **Best for zero-demo in-the-wild MPC across rigid / deformable /
  articulated / tool-use:** [[pointworld]] — 30–100% SR on Franka with
  single RGB-D, no demos. Only method in family with published scaling
  laws (log-linear in data + model size).

## Open eval gaps

1. **No head-to-head [[dex4d]] vs [[pri4r]] vs [[mu0]] vs
   [[pointworld]]** — they sit at four distinct poles of the tracks-
   role axis (conditioning / privileged / WM features / dynamics
   simulator). Different benchmarks (real dexterous vs. RoboCasa
   parallel-jaw vs. RoboCasa365 + UR3 vs. Franka in-the-wild)
   preclude direct comparison; would inform which representation
   role wins where.
2. **No long-horizon multi-object benchmark.** All seven methods are
   short-horizon single-object. µ0 evaluates trace horizons up to
   T=32; PointWorld chunks are 10 steps (1s).
3. **No matched-data ablation.** Different methods use different
   training corpora (~400 Spot demos vs. sim play vs. 30 human
   demos vs. 500K Kubric vs. µ0's 8×-TraceGen mix vs. PointWorld's
   2M-trajectory DROID+B1K); training-data-matched comparisons
   would reshape the rankings.
4. **No AJ-style metric for the *flow* itself** — except µ0's Table 1
   (ADE/FDE/DTW × top1/top5 × horizons {8, 16, 32}) and PointWorld's
   ℓ2 mover metric (per-point per-timestep). Two different metric
   traditions.
5. **µ0 vs. TraceGen alone is the sharpest video-only trace-generator
   comparison** — TraceGen not yet in the wiki. Ingest priority.
6. **PointWorld vs. µ0 fusion (µ0-trace as MPPI target, PointWorld as
   simulator) is unexplored** — natural combination given
   complementary roles.

## Recurring axes

- [[point-tracks-as-manipulation-interface]] — the binding concept.
- **Four tracks-role poles** — conditioning (T2A/I2F/3DF/Dex4D),
  privileged (Pri4R), WM features (µ0), dynamics simulator
  (PointWorld).
- **Language-conditioned trace generator vs. action-conditioned
  dynamics WM** — first six methods vs. PointWorld.
- **Heuristic vs. learned policy vs. MPC** — T2A-OL / 3DF vs. I2F2A /
  Dex4D / T2A-residual / µ0-AE vs. PointWorld (MPPI).
- **2D vs. 3D tracks** — T2A / I2F2A vs. 3DF / Dex4D / Pri4R / µ0 /
  PointWorld.
- **Sparse (fixed grid or semantic) vs. dense per-pixel** — first six
  are sparse; PointWorld is dense per-pixel.
- **Waypoints vs. B-spline vs. displacements** — most are waypoints;
  µ0 outputs B-spline control points; PointWorld outputs per-step
  per-point displacements.
- **Parallel-jaw vs. dexterous** — all except Dex4D are parallel-jaw.

## When new manipulation-via-tracks papers arrive

Update this table and the [[point-tracks-as-manipulation-interface]]
concept page. If a new method shifts the layout (e.g. learned 3D
dexterous + zero real demos at SOTA), promote findings to
[[overview]].
