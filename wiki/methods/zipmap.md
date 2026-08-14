---
type: method
title: ZipMap
status: growing
tags: [3d-reconstruction, feed-forward, test-time-training, linear-complexity, streaming, stateful, fast-weights, vggt, novel-view-synthesis, scalability]
sources:
  - "[[jin-2026-zipmap]]"
related:
  - "[[lact]]"
  - "[[test-time-training]]"
  - "[[vggt]]"
  - "[[cut3r]]"
  - "[[streamvggt]]"
  - "[[point3r]]"
  - "[[vgg-t3]]"
  - "[[loger]]"
  - "[[fsm]]"
  - "[[feed-forward-3d-reconstruction]]"
created: 2026-08-14
updated: 2026-08-14
---

# ZipMap

**Linear-time, stateful, feed-forward 3D reconstruction.** A VGGT-style
backbone with all global attention replaced by a large-chunk
[[lact|LaCT]] test-time-training layer, so cost is **O(N)** in the number
of input views instead of O(N²) — while matching or beating quadratic
VGGT/π³ quality. The fitted fast-weights double as a **queryable scene
state** (novel-view point/depth at ~100 FPS, independent of N) and support
**streaming** (online per-view update).

## One-line summary

Zip an entire image collection into the fast-weights of a SwiGLU-MLP in
one forward pass (via [[lact|LaCT]]); reconstruct cameras + depth +
pointmaps from it in O(N); query it at novel cameras in constant time.

**Problem**
	 bidirectional architectures: powerful but scales quadratic to input frames vs sequential architectures: fast but worse performance.
**Idea**
	 Use TTT layer as an alternative to global attention layer, which behaves as global attention but only takes linear time.
**Key Details**
	 - TTT layer is compact version of kv cache, which means it learns how to assign v corresponding to each q. It is updated with qkv embedding of every patches.
	 - can be easily modified into a streaming model, which uses token from current frame to update fast weight
**What's not solved**
	- fixed size parameter as a memory -> still limited in number of input frames.

## Inputs / outputs

- **In:** N RGB images `{I₁,…,I_N}` (video or unstructured collection);
  optional target **ray map** `T ∈ R^{H×W×9}` to query a novel camera.
- **Out (per input view):** camera `c_i ∈ R^9` (quat + translation + 2
  intrinsics), depth `D_i` + confidence `Σ_i`, local pointmap `P_i`.
- **Out (per query ray map):** target-view RGB `Iᵗ`, depth `Dᵗ`, `Σᵗ`.
- **State:** fast weights `W = {W₁,W₂,W₃}` of the SwiGLU-MLP — a
  fixed-size (`6d²`/layer) compressed scene representation.

## How it works

1. **Tokenize** (§3.1): frozen DINOv2 patch tokens per image + 1 camera
   token + 4 register tokens. Ray-map inputs get a special **query token**.
2. **Backbone** (§3.2): L=24 blocks, each =
   - **Local window attention** — self-attention within one view's tokens
     (RoPE); intra-view spatial structure.
   - **Global large-chunk TTT** ([[lact|LaCT]]) — the O(N) global mix:
     - virtual KV loss `L = −f_W(k_i)ᵀv_i`, per-token LR `η_i`;
     - one gradient step, **Muon / Newton–Schulz** conditioning + L2 norm:
       `Ŵ = ∥W∥·(W−NewtonSchulz(g))/∥W−NewtonSchulz(g)∥`;
     - **apply** `o′_i = f_Ŵ(q_i)` — linear-cost self-attention analogue;
       same `Ŵ` applied to ray-map query tokens = cross-attention analogue
       at constant per-query cost;
     - gated output `o_i = RMSNorm(o′_i)·SiLU(W_g o′_i)`.
3. **Streaming variant** (§3.3): update `W^(t) ← TTTUpdate(W^(t−1); k_t,v_t)`
   one view at a time (same KV objective, current view only).
4. **Heads** (§3.4): camera (VGGT design) · point (π³-style local) · depth
   (+confidence) · query (ray-map → RGB + depth).

### Frozen / reused / trained

| From VGGT | ZipMap does |
|---|---|
| DINOv2 encoder | reuse (frozen encoder) |
| frame-wise attention | init **local window attention** |
| global attention | init a **subset** of TTT params, then train |
| — | train TTT blocks (lr 1e−4), heads, projections |

~1.40B params. 64 H100, 3 stages: 80K static + reference view → 40K
+dynamic finetune → 60K remove reference view (π³-style affine-invariant
camera loss). 29 datasets.

## Where it's been applied

All in [[jin-2026-zipmap]]:

- **Camera pose** — Re10K, Co3Dv2, Sintel, TUM-dyn, ScanNet: matches VGGT/π³,
  beats CUT3R/TTT3R (Sintel ATE 0.132 vs CUT3R 0.216 / TTT3R 0.204).
- **Point maps** — DTU, ETH3D, 7-Scenes, NRGBD: matches quadratic SOTA,
  ~3–4× better than linear baselines.
- **Depth** — Sintel/Bonn/KITTI video + NYU-v2 mono: best O(N), ≥ VGGT.
- **Novel-view / scene-state query** — point/depth at ~100 FPS from state
  alone; infers unseen low-frequency structure (walls/floors).
- **Efficiency** — 700+ frames <10 s (75 FPS), >20× VGGT, ~3× CUT3R/TTT3R.

## Known limitations

- **Reconstruction, not tracking** — no temporal correspondence / dynamic
  scene-flow output. Handles dynamic scenes in *recon* (Fig 3) but reports
  no dense-4D-tracking numbers vs [[v-dpm]]/[[4rc]]/[[point4d]].
- **Deterministic state** — infers common structure, cannot hallucinate
  unseen objects.
- **Streaming under-reported** (appendix only); main claims bidirectional.
- **Empirical chunk/state size** (inherited [[lact]] limitation).
- **Heavy training** — full backbone from VGGT init, not a cheap TTT
  retrofit like [[vgg-t3]].

## Related methods

- **Core dependency:** [[lact]] — ZipMap = LaCT applied to feed-forward 3D
  reconstruction, + a queryable ray-map path + streaming update + 3D heads.
- **TTT-for-3D siblings:** [[vgg-t3]] (offline-global cheap retrofit),
  [[loger]] (chunk-streaming, π³ backbone), [[fsm]]/[[lacet]] (4D-NVS,
  elastic consolidation). ZipMap is [[test-time-training]] **Pattern F** —
  the one that does bidirectional batch *and* streaming from one recipe.
- **Streaming-3D it beats:** [[cut3r]], [[point3r]], [[streamvggt]] — all
  O(N), but degrade with N; ZipMap holds quadratic quality. Note
  [[aleksander-holynski]] is senior on both ZipMap and CUT3R.
- **Quadratic baselines it matches:** [[vggt]] (init + encoder reused), π³.
- **For a 3D streaming *tracker*** (user project): ZipMap is the closest
  backbone recipe — O(N), stateful, streaming, queryable — but stops at
  geometry. Add a correspondence/scene-flow head + dynamic-tracking eval.
