---
type: method
title: µ0 (mu0)
status: growing
tags: [manipulation, point-tracking, 3d-point-tracking, world-model, flow-matching, vla, cross-embodiment, foundation-model, b-spline, semantic-keypoint]
related:
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[track2act]]"
  - "[[im2flow2act]]"
  - "[[3d-flow-action]]"
  - "[[dex4d]]"
  - "[[pri4r]]"
  - "[[pointworld]]"
  - "[[trace-anything]]"
  - "[[cmp-point-track-manipulation]]"
sources:
  - "[[lee-2026-mu0]]"
created: 2026-07-09
updated: 2026-07-09
---

# µ0 (mu0)

Query-conditioned **3D trace-space world model** for cross-embodiment
manipulation. Predicts future 3D trajectories for semantically
selected interaction points (objects, tools, hands, contact regions)
via B-spline control points + flow matching. Frozen and reusable as a
motion prior for downstream action experts.

## One-line summary

Video-only pretraining of a VLM-conditioned trace expert →
frozen features feed a small action expert → matches or beats
action-labeled VLAs (π₀ / π₀.₅) on downstream manipulation.

## Inputs / outputs

**Inputs (trace prediction mode):**
- RGB observation `I_t`
- Optional depth `I_depth`
- Language instruction `l_c`
- Query keypoint set `Q_t = {q_t^n}` (semantic keypoints from DINOv2
  clusters)
- Optional history traces `T_ref^{t-h:t}`

**Output:** future 3D traces `T̂_ref^{t:t+H}` in the reference-camera
frame, one per query point.

**Inputs (policy mode):**
- Frozen µ0 features from a single partial-denoising step
- Gripper-camera image, proprioception, language

**Output:** continuous action chunks for the target embodiment.

## How it works

Two big pieces:

### (1) TraceExtract — data engine

Turns heterogeneous manipulation videos into event-captioned 3D trace
tuples `{I_t, l_c, Q_t, T_ref^{t-h:t+H}}`. Three stages:

1. **Semantic keypoint sampling**: DINOv2 patch features → entity
   clusters → propagate identity → allocate fixed budget per entity →
   spatially diverse keypoints on high-visibility frames. Movement
   filter suppresses static / background tracks.
2. **Global-local 3D trace construction**: sparse anchor frames define
   shared global frame; dense local chunks reconstructed and aligned
   back; tracks propagated across chunk boundaries via each point's
   last valid world-space position. Reproject into per-chunk reference
   camera; arc-length reparameterize to normalize speed.
3. **Event-centric captioning**: Savitzky–Golay smooth trace
   acceleration → chunk boundaries at low-acceleration valleys → VLM
   caption each chunk from start/midpoint/end frames → text LLM merges
   into hierarchical (event + task) captions.

Curates **~8×** the scale of TraceGen's fixed-grid trace dataset.

### (2) µ0 model

```
Prediction:  ( I_t, l_c, Q_t, T_ref^{t-h:t} )  →  T̂_ref^{t:t+H}
```

Three components:

- **Multi-modal conditioning backbone (§3.1)** — pretrained
  SmolVLM2-2.2B prefix encodes RGB + language; depth (optional) via
  separate patch stem sharing deeper SigLIP layers. Trace expert
  cross-attends to VLM KV cache while maintaining a separate motion-
  specific stream.
- **Permutation-equivariant Trace Expert (§3.2)** — each keypoint is
  an exchangeable query token. Query token = segment embed (history
  vs future) ⊕ Fourier embed of current pixel loc ⊕ local DINO
  feature. Future represented as **cubic B-spline control points**
  after subtracting current 3D anchor.
- **Flow matching with semantic structure (§3.3)** — conditional flow
  model over B-spline control points. adaLN-Zero flow-time modulation.
  Loss:
  ```
  L = L_flow + λ_done L_done + λ_rig L_rig
  ```
  - `L_flow` = velocity field toward clean control points.
  - `L_done` = validity prediction (per-step termination under
    occlusion / track loss).
  - `L_rig` = **semantic rigidity** — keypoints within the same DINO
    cluster preserve local geometry.

Inference = denoising loop → decode control points → smooth 3D trace.

### (3) Action expert (downstream)

Freeze pretrained µ0. Train a small **action expert** that reads
frozen trace-denoising features from a **single partial-denoising
step**, injects via gated cross-attention into VLM features, then
predicts continuous action chunks conditioned on gripper-camera +
proprioception + language.

Video-only WM pretraining → action-labeled downstream training only.

## B-spline representation detail

For each query, future motion = cubic B-spline defined by a small set
of control points (compact, smooth, low output dim vs dense
waypoints). Same primitive family as [[trace-anything]]. At
manipulation horizons T ∈ {8, 16, 32} the primitive works well; the
wiki's long-horizon critique of Bézier splines (200-frame benchmarks
where Trace Anything underperforms) doesn't bite at these lengths.

## Training

- Backbone init: SmolVLM2-2.2B.
- Objective: flow matching + validity + rigidity (see §3.3).
- Pretraining data: heterogeneous human + robot manipulation videos
  processed through TraceExtract.
- Scaling: prediction quality improves monotonically with model size
  up to 2.59B params.
- Policy: fine-tune action expert only; µ0 frozen.

## Where applied

- **Trace forecasting**: 2D + 3D benchmarks; beats Track2Act, Hamster,
  3DFlowAction, TraceGen, Dream2Flow, Gemini-3.1-pro, GPT-5.5 on top5
  across horizons. 0.29s inference (2.9× faster than Track2Act,
  ~200× faster than Gemini API).
- **Downstream sim**: RoboCasa365 8 tasks — 30.25% avg SR vs π₀
  25.25% (video-only pretraining beats action-labeled by +5 pts).
- **Downstream real**: UR3 3 tasks — 91.7% avg vs π₀.₅ 80%, TraceGen
  81.7%, π₀ 71.7%.

## Known limitations

- Inherits perception-stack errors (DINOv2 clustering, 3D recon,
  tracking, captioning).
- No forces / tactile / contact modes — geometry + motion only.
- Tabletop / short-horizon evals only; no mobile manip, dexterous
  hands, or long-horizon composite tasks.
- π₀.₅ still wins in sim (42% vs 30.25%) — action-labeled pretraining
  not fully replaceable at RoboCasa scale.
- Frozen µ0 vs co-fine-tuned µ0+AE not benchmarked.

## Related methods

- **Direct predecessor:** TraceGen (Lee et al. CVPR 2026) — fixed-grid
  traces + episode captions + depth-required. µ0 fixes all three.
  **Not yet in wiki.**
- **Same family:** [[track2act]] (2D DiT + PnP + residual BC),
  [[im2flow2act]] (object-only 2D flow + sim BC), [[3d-flow-action]]
  (3D flow + GPT-4o + optimization policy), [[dex4d]] (Wan2.6 → 3D
  tracks → sim-to-real RL, dexterous), [[pri4r]] (3D tracks as
  privileged VLA supervision), [[pointworld]] (action-conditioned
  dynamics simulator; complementary role).
- **Design axis µ0 introduces:** world model (frozen features
  consumed by AE) vs. plan generator (tracks executed by controller).
- **B-spline lineage:** [[trace-anything]] (per-pixel Bézier splines
  for video-length 4D). µ0 reuses the primitive at manipulation
  horizons where it holds up.
- **VLA baselines it competes with:** π₀ (Black 2025), π₀.₅ (Physical
  Intelligence 2025) — both from the wiki's [[physical-intelligence]]
  cluster.

## Open directions

- Long-horizon composite tasks (Bézier holds at T=32; unknown at T=200
  for manipulation).
- Dexterous / mobile manipulator embodiments.
- Head-to-head against [[pri4r]] on shared benchmarks — training-time
  vs. inference-time trace supervision.
- Combining µ0's trace features with 4D-reconstruction backbones
  ([[point4d]] / [[stride]] / [[v-dpm]]) — currently µ0 uses its own
  reconstruction pipeline inside TraceExtract; using a feed-forward 4D
  backbone as the perception side is unexplored.
- Force / tactile / contact-mode conditioning.
