---
type: source
source_type: paper
title: "Streaming Visual Geometry Transformer"
authors:
  - Zhuo, Dong
  - Zheng, Wenzhao
  - Guo, Jiahe
  - Wu, Yuqi
  - Zhou, Jie
  - Lu, Jiwen
year: 2026
venue: arXiv preprint (under review)
url: https://arxiv.org/abs/2507.11539
raw_path: papers/2507.11539v2.pdf
status: ingested
tags: [3d-reconstruction, streaming, causal-attention, distillation, vggt, online, point-map, depth-estimation, camera-pose, kv-cache]
related:
  - "[[wang-2025-vggt]]"
  - "[[wang-2025-cut3r]]"
  - "[[streamvggt]]"
  - "[[cut3r]]"
  - "[[vggt]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[online-vs-offline-tracking]]"
created: 2026-06-12
updated: 2026-07-02
---

# Streaming Visual Geometry Transformer (StreamVGGT)

## TL;DR

StreamVGGT is a **causal-attention + KV-cache** restructuring of [[vggt|VGGT]]
that turns it into a streaming 3D reconstruction model. The architecture
replaces VGGT's global self-attention with **temporal causal attention**
and adds a **cached memory token** mechanism so each new frame only
attends to itself and predecessors. Trained by **distillation from
VGGT** (no per-dataset GT engineering), it achieves on-the-fly online
3D reconstruction with a ~5× speedup on the current-frame inference at
40-frame sequences while maintaining VGGT-comparable accuracy on
7-Scenes, NRGBD, ETH3D, and TUM-dynamics.

## Why it matters

For the user's middle-ground 3D-tracker, StreamVGGT is **the streaming
3D backbone alternative to [[cut3r|CUT3R]]**. The design doc currently
slots CUT3R into the "history/context" role; StreamVGGT achieves the
same streaming-3D goal via a different mechanism (causal attention +
KV cache) and *outperforms CUT3R* on 7-Scenes, NRGBD, ETH3D,
TUM-dynamics, and camera pose. It also distills from VGGT, which means
the streaming model inherits VGGT's geometry priors — likely better
generalization than CUT3R's from-scratch recurrent training.

Direct relevance to the 3D-tracker design:
- **Streaming backbone slot:** StreamVGGT is a candidate replacement
  for the CUT3R streaming backbone in §5.1 of the middle-ground design.
- **Feature lifting bug:** the design audit flagged that CUT3R's hidden
  state isn't a spatial grid. StreamVGGT's cached memory tokens are
  per-frame per-patch — *they are a spatial grid*. The TAPIP3D-style
  query-feature lifting becomes straightforward.
- **Camera pose:** StreamVGGT has an explicit per-frame Camera Head
  (quaternion + translation + FoV) — same role CUT3R plays in the
  design doc.

## Key claims

- **VGGT global attention → temporal causal attention** is enough to
  enable streaming, but training-from-scratch causal models lose
  significant accuracy. Distillation closes that gap (Table 9).
- **Cached memory tokens (Eq 6):** keys/values of past frames are cached;
  current frame cross-attends to the cache. Equivalent to causal
  self-attention but with O(N) compute and O(N) memory per new frame,
  vs VGGT's full O(N²) recompute.
- **Knowledge distillation from VGGT** unifies multi-task supervision
  (depth + pointmap + camera + tracking) under one teacher's soft
  targets + confidences. Eliminates per-dataset annotation engineering.
- **Outperforms CUT3R** on **all measured benchmarks** (Tables 1–6) —
  3D reconstruction Acc/Comp/NC, ETH3D pointmap, TUM-dynamics 4D,
  single-frame + video depth, camera pose ATE/RPE.
- **FlashAttention-2 compatible** — gets the LLM-style efficiency
  optimizations for free (Table 10: 88ms inference at frame 5 vs CUT3R's
  102ms).
- **Two scalability strategies for very long sequences (Table 8):**
  windowed streaming (chunk-then-align) and K-nearest-frames caching
  (attend only to recent K frames).

## Methods

### Architecture (Fig 3)

```
For each new frame I_T:
  F_T = ImageEncoder(I_T)             # DINOv2 patchify
  G_T = SpatioTemporalDecoder(F_T, {M_t}_{t=1..T-1})    # causal
  P_T, D_T, g_T, y_T = MultiTaskHead(G_T)
  M_T = TokenCachedMemory(G_T)         # store K, V for future
```

**Spatio-Temporal Decoder** = L=24 alternating layers of:
- **Spatial attention** (intra-frame, like ViT).
- **Temporal causal attention** (across frames, but each token attends
  only to itself and predecessors).

This replaces VGGT's alternating frame-wise + global self-attention.
Same parameter count.

### Cached memory token (Eq 6)

```
G_T = Decoder(CrossAttn(F_T, {M_t}_{t=1..T-1}))
M_T = TokenCachedMemory(G_T)
```

Each frame's processed tokens are cached as keys+values; new frame queries
them via cross-attention. Identical to an LLM KV-cache.

**Camera token** is a learnable register on frame 1 used as the global
world-frame reference for all subsequent frames — i.e., **world frame =
camera₀ convention**, same as TAPVid-3D / [[spatialtracker-v2]].

### Multi-task heads (§3.2)

- **Camera Head:** quaternion (4) + translation (3) + FoV (2) = `g_i ∈ R^9`.
- **Geometry Head:** point map `P_i ∈ R^{3×H×W}` + depth `D_i ∈ R^{H×W}` +
  confidences. Built on DPT.
- **Track Head:** dense per-pixel tracking features. Cross-frame tracks via
  the same correlation mechanism as VGGT.

### Distillation training (§3.3, Eqs 7–10)

Loss = `L_camera + L_depth + L_pmap`, where:
- `L_camera` = Huber loss on quaternion + translation + FoV.
- `L_depth`, `L_pmap` = confidence-weighted L1 + gradient term — VGGT's
  loss form, with **VGGT's predictions as pseudo-GT instead of dataset GT.**

950M parameters fine-tuned from VGGT init for 10 epochs on 4× NVIDIA A800
in ~7 days. Datasets: 13 sources spanning Co3Dv2, BlendMVS, ARKitScenes,
MegaDepth, WildRGB, ScanNet, HyperSim, OmniObject3D, MVS-Synth,
PointOdyssey, Virtual KITTI, Spring, Waymo.

## Results

### 3D reconstruction (Tables 1–2)

**7-Scenes (Acc / Comp / NC):**

| Method | Type | Acc | Comp | NC |
|---|---|---|---|---|
| VGGT (dense-view) | offline | 0.088 / 0.039 | 0.091 / 0.039 | 0.787 / 0.890 |
| CUT3R | streaming | 0.126 / 0.047 | 0.154 / 0.031 | 0.727 / 0.834 |
| **StreamVGGT** | streaming | **0.129 / 0.056** | **0.115 / 0.041** | **0.751 / 0.865** |

**ETH3D overall** (Chamfer ↓):

| Method | Type | Acc | Comp | Overall |
|---|---|---|---|---|
| VGGT (dense-view) | offline | 0.928 | 0.443 | 0.686 |
| CUT3R | streaming | 1.426 | 1.395 | 1.411 |
| **StreamVGGT** | streaming | **0.609** | **0.545** | **0.577** |

### Camera pose (Table 6)

ScanNet ATE/RPE-trans/RPE-rot:

| Method | ScanNet ATE | ScanNet RPE rot |
|---|---|---|
| VGGT (dense-view) | 0.035 | 0.307 |
| CUT3R | 0.099 | 0.600 |
| **StreamVGGT** | **0.048** | **0.557** |

Roughly matches VGGT — significantly beats CUT3R, [[point3r]], Spann3R.

### Inference efficiency (Fig 2, Table 7)

40-frame sequence, inference of current frame:

| Method | Time | Memory |
|---|---|---|
| VGGT | 2099 ms | 11.4 GB |
| **StreamVGGT** | **216 ms** | **6.6 GB** |

(Note: StreamVGGT's memory grows linearly with sequence length due to
the KV cache; VGGT also grows but does full O(N²) recompute.)

### Ablations (Tables 8–10)

- **Distillation is necessary:** w/o KD, StreamVGGT loses ~30% accuracy
  vs distilled (Table 9).
- **Cache + FlashAttn:** 88ms vs 1135ms w/o either (Table 10).
- **Window streaming** (50/100 frames) and **K-nearest caching** (K=50/100)
  both maintain quality while bounding memory.

## Limitations / open questions

- **Memory grows linearly with sequence length.** Unlike CUT3R's
  fixed-size hidden state, StreamVGGT's KV cache scales with N.
  Mitigated by window-streaming or K-nearest caching with measurable
  accuracy drop.
- **Causal training requires KD.** From-scratch causal training loses
  significant accuracy — KD is the load-bearing piece (Table 9).
- **No dynamic scene improvement over VGGT.** Same dynamic-scene
  limitations as VGGT (TUM-dynamics is the only dynamic benchmark and
  results are comparable to CUT3R, not dramatically better).
- **Causal-attention inheritance question.** It's unclear how much of
  the geometric prior VGGT learned via global attention survives the
  causal restriction. KD recovers most but not all metrics.

## Connections

- **[[wang-2025-vggt]]** — the offline teacher. StreamVGGT is essentially
  VGGT with the global self-attention pattern restricted to causal
  temporal + cached KV.
- **[[wang-2025-cut3r]]** — the direct streaming competitor. Different
  mechanism: CUT3R uses a fixed-size token bank updated by attention;
  StreamVGGT uses a growing KV cache + causal attention. StreamVGGT
  beats CUT3R on essentially every measured metric.
- **[[feed-forward-3d-reconstruction]]** — concept; StreamVGGT is the
  streaming feed-forward variant.
- **[[online-vs-offline-tracking]]** — concept; clear example of
  online-from-offline distillation.
- **[[lact]]** — both papers are streaming sequence models for 3D, but
  via different mechanisms: StreamVGGT uses a growing KV cache (no test-
  time training); LaCT uses a fast-weight MLP whose weights *are* the
  cache (gradient updates per chunk). Two complementary efficiency
  strategies.
- **Same group / line:** Wenzhao Zheng (Tsinghua) is project lead. Same
  group has done streaming perception work — [[point3r]] (Wu, Zheng,
  Zhou, Lu 2025; NeurIPS 2025) is the same-group companion using
  "explicit geometry-aligned spatial pointer memory" vs StreamVGGT's
  "implicit cached visual tokens." Zheng is 2nd author on Point3R and
  project lead on StreamVGGT — two complementary streaming-memory
  designs from the same Tsinghua Automation group.
- **For the user's design:** StreamVGGT is the **leading candidate to
  replace CUT3R as the streaming 3D backbone** in the middle-ground
  3D-tracker design. The cached memory tokens *are* a per-frame spatial
  grid (unlike CUT3R's compressed hidden state), so TAPIP3D-style
  feature lifting becomes natural rather than a leap of logic.

## Citation

Zhuo, D., Zheng, W., Guo, J., Wu, Y., Zhou, J., & Lu, J. (2026).
*Streaming Visual Geometry Transformer.* arXiv:2507.11539.
https://arxiv.org/abs/2507.11539
