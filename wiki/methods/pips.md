---
type: method
title: PIPs (Persistent Independent Particles)
status: growing
tags: [point-tracking, optical-flow, mlp-mixer, iterative-refinement, window-based, occlusion]
sources:
  - "[[harley-2022-pips]]"
related:
  - "[[point-tracking]]"
  - "[[joint-point-tracking]]"
  - "[[trajectory-chaining]]"
  - "[[doersch-2023-tapir]]"
  - "[[tapir]]"
  - "[[cotracker]]"
  - "[[tapnext]]"
  - "[[tapip3d]]"
created: 2026-06-26
updated: 2026-06-26
---

# PIPs (Persistent Independent Particles)

The foundational deep-learning **[[point-tracking]]** method: tracks one
target pixel as an 8-frame trajectory of positions + appearance features,
refined by an iterative MLP-Mixer over multi-scale correlation pyramids.
Independent per-point inference; chained at test time for long videos.
ECCV 2022 — the wiki's TAP origin point.

## One-line summary

Initialize an 8-frame `(pos, feat)` trajectory at the query, repeatedly
read multi-scale correlation pyramids around the current positions, and
update both with a 12-block MLP-Mixer — 6 iterations, then read out
visibility.

## Inputs / outputs

- **In:** 8-frame RGB clip `[T=8, H, W, 3]` + one query `(x_1, y_1)`.
  The model can be called in parallel for N query points; computation
  is shared at the CNN backbone, *not* across tracks.
- **Out:** `(x_t, y_t)` for `t = 1..T`, plus per-frame visibility
  `v_t ∈ [0,1]`.

## How it works

1. **CNN backbone (RAFT BasicEncoder, stride 8, C=256).** Independent
   per-frame features `F_t ∈ R^{H/8 × W/8 × 256}`. No temporal convs.
2. **Initialization.** Bilinearly sample the query feature
   `f_1 = F_1[x_1, y_1]`. Set `F^0_t = f_1` and `X^0_t = (x_1, y_1)`
   for all `t` (appearance-constancy + zero-velocity priors).
3. **Multi-scale correlation pyramid `C^k`.** For each `t`, dot-product
   `F^k_t` against `F_t`, crop a 7×7 patch at `X^k_t`, repeat at L=4
   pyramid levels (radius 3). Shape per frame: `7·7·4 = 196`.
4. **MLP-Mixer refinement.** Concatenate per-frame
   `(sinusoidal_emb(X^k_t − X^0_t), F^k_t, C^k_t)` into a `T × D`
   sequence. Run a 12-block MLP-Mixer; treat time as the "token"
   axis. Linear head → `(ΔX, ΔF)` per frame. Update
   `X^{k+1} = X^k + ΔX`, `F^{k+1} = F^k + ΔF`. Iterate K=6.
5. **Visibility head.** Linear + sigmoid on `F^K` → `V^K ∈ [0,1]^T`.

### Losses

```
L_main  = Σ_k γ^(K−k) ‖X^k − X*‖_1          (γ=0.8, RAFT-style)
L_ce    = BCE(V*, V)
L_score = −log softmax(c_i) · 𝟙{V* ≠ 0}      (peak score map at GT pixel)
```

### Test-time trajectory chaining (§3.7)

To go beyond 8 frames: run the model, then re-initialize at the latest
timestep with visibility `≥ τ` (τ starts at 0.99, decremented until a
valid pivot is found), continuing from there. **Re-use the original
`F^0 = f_1`** across re-initializations to lock identity onto the
original target (prevents drift to the occluder).

This is the 2D-pixel-query progenitor of [[trajectory-chaining]] — the
same "Sim(3)-align chunks + propagate identity" template later used by
4D reconstruction methods, with the same failure mode (chain breaks
when the target is occluded across the entire window).

## Where it's been applied

- **FlyingThings++** (synthetic, the paper's own training/test split).
- **KITTI tracking** (vehicles + pedestrians from 3D-box GT).
- **CroHD** (crowd heads, 1080×1920, 8-frame splits).
- **BADJA** keypoint propagation (animal videos overlapping DAVIS).
- Inherited as a substrate by [[cotracker]] (PIPs-style correlation +
  iterative updates *with* added cross-track attention), [[tapir]]
  (PIPs refinement + global init + depthwise conv), [[tapip3d]] (3D
  lift; same authors).

## Known limitations

- **Independent per-point inference.** No information shared between
  tracks. [[cotracker]] / [[cotracker3]] / [[joint-point-tracking]]
  fill this gap.
- **Fixed 8-frame window.** MLP-Mixer requires fixed sequence length;
  longer videos need chaining. TAPIR's depthwise convolution removes
  this constraint.
- **No global per-frame initialization.** Position is initialized at
  the query and only moves via local refinement — fragile if the
  target undergoes large motion within the first refinement
  iterations. TAPIR fixes via TAP-Net-style global init.
- **No uncertainty estimate.** Visibility is binary; the model can
  confidently predict a wrong position. TAPIR adds self-supervised
  position uncertainty.
- **Chaining can lose targets across long occlusions** — if the gap
  exceeds 8 frames, identity is unrecoverable from features alone.
- **Synthetic-only training.** No real-data fine-tuning recipe (later
  closed by BootsTAP / [[cotracker3]]).

## Related methods

- **Direct successor:** [[tapir]] — combines PIPs iterative refinement
  with TAP-Net global initialization; replaces MLP-Mixer with
  depthwise conv (any-length sequences); adds position uncertainty.
- **Joint-tracking response:** [[cotracker]] — adds cross-track
  attention via proxy tokens; keeps PIPs-style cost volumes and
  iterative updates; sliding-window inference.
- **Continued cost-volume + refinement lineage:** [[cotracker3]] (real-
  data pseudo-labeling), LocoTrack, TAPTR series — all keep PIPs'
  correlation + iterate template.
- **Generic-backbone reaction:** [[tapnext]] argues PIPs' cost
  volumes + iterative refinement re-emerge as attention patterns
  when the inductive biases are dropped (see
  [[q-emergent-tracking-heuristics]]).
- **3D extension by the same author:** [[tapip3d]] — Harley co-authors;
  world-coordinate N2N attention is the modern descendant.
- **Optical-flow ancestor:** RAFT (Teed & Deng 2020) — PIPs ports
  RAFT's BasicEncoder + correlation pyramid + iterative-update +
  exponentially-weighted loss to multi-frame.
- **Classical ancestor:** Sand & Teller, "Particle Video" (CVPR 2006)
  — the conceptual seed (long-range point trajectories as the
  representation).
