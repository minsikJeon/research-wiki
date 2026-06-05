---
type: concept
title: Test-Time Training (TTT)
status: growing
tags: [fast-weights, associative-memory, linear-attention, kv-compression, online-learning, 3d-reconstruction, sequence-modeling]
sources:
  - "[[elflein-2026-vgg-t3]]"
  - "[[zhang-2026-loger]]"
related:
  - "[[vgg-t3]]"
  - "[[loger]]"
  - "[[feed-forward-3d-reconstruction]]"
created: 2026-05-29
updated: 2026-05-29
---

# Test-Time Training (TTT)

## What it is

A **fast-weight associative memory** mechanism that treats some
network weights as *state* updated on-the-fly during inference. The
weights `W` of a small inner model `f_W: R^d → R^d` are optimized
against a self-supervised objective `L_t(f_W(k), v)` while consuming
input tokens. After fitting, applying `f_W(q)` retrieves information
stored in `W`.

Originally proposed for language modeling as a constant-memory
alternative to linear attention (Sun et al., 2024). Recently reframed
as **KV-space compression** for visual geometry:

- **VGGT's variable-length KV** = a scene representation that grows
  with `n` (number of input views).
- **TTT compresses that KV into a fixed-size MLP** whose weights *are*
  the compressed scene.

## The two operations

Given a token stream and per-token projections `(q_i, k_i, v_i)`:

- **Update:** `W ← W − η ∇_W L_t(f_W(k_i), v_i)`.
- **Apply:** `o_i = f_W(q_i)` (replaces softmax(QK^T)V).

Variants differ in:

- *Inner model* — MLP, SwiGLU MLP, MLP with ShortConv2D.
- *Loss* — L2 reconstruction, dot-product attention loss.
- *Optimizer* — SGD, Muon, Adam.
- *Granularity* — per-token (TTT3R), per-chunk (LoGeR), per-scene
  (VGG-T3).
- *State persistence* — reset per scene (VGG-T3), carry across
  chunks (LoGeR), carry across all frames (TTT3R).

## Why people are using it for 3D reconstruction

Three reasons converge:

1. **Linear time/memory.** Apply is `O(1)` per token; the asymptotic
   bottleneck of `O(n²)` softmax disappears.
2. **Fixed-size scene state.** The fitted `W` is a *queryable*
   compressed representation. After scene-fit, novel queries are
   cheap forward passes — enabling feed-forward visual localization
   (VGG-T3), state transfer between chunks (LoGeR), and persistent
   online state (TTT3R / [[cut3r]]).
3. **No retraining of the backbone required.** TTT is typically
   bolted onto a pre-trained transformer; only QKV projections + the
   inner model are fine-tuned. ~12% of VGGT scratch-training cost
   (VGG-T3).

## What TTT competes with

- **Softmax attention** — better quality, quadratic cost.
- **Linear attention** (Katharopoulos 2020, RetNet, Mamba family) —
  linear cost, weaker on long-context recall. TTT is *more
  expressive* than linear attention because the fast-weight model
  can be nonlinear.
- **Sliding window attention (SWA)** — lossless local but no global
  reach.
- **RNN-style persistent state** (CUT3R) — also fixed-size but no
  formal update objective; TTT formalizes the update as gradient
  descent on a loss.

## The two recent design patterns

This wiki now has two concrete TTT-for-3D designs to compare:

### Pattern A — Global offline (VGG-T3)

- Single scene = single TTT optimization.
- Fit `W` over the entire image collection (mini-batched if
  collection doesn't fit on one GPU).
- After fitting, the TTT is *done*; novel queries just forward.
- Use case: large-scale offline reconstruction, visual localization
  from compressed maps.

### Pattern B — Streaming chunk-wise (LoGeR)

- Per-chunk apply + update of `W_m`.
- `W_m` is *carried* across chunks → an evolving global state.
- Combined with SWA for adjacent-chunk lossless detail.
- Use case: minutes-long video reconstruction, streaming SLAM-like
  applications.

### Pattern C — Per-frame streaming (TTT3R; not yet ingested as primary)

- Per-frame apply + update.
- Most aggressive linear scaling.
- Worst at long-context geometric coherence (LoGeR Tab. 2 shows
  TTT3R loses to LoGeR by 4× ATE on KITTI).

## Open questions

- **Capacity-vs-context trade-off.** Is there a *required* MLP size as
  a function of scene complexity? Both VGG-T3 and LoGeR pick
  empirically; no theory.
- **What's actually stored in `W`?** VGG-T3 frames `W` as "compressed
  KV"; LoGeR frames it as "global coordinate-frame anchor." Are these
  the same thing under different decoders? Probably yes but unclear.
- **Hybrid memory generalization.** LoGeR shows SWA+TTT beats either
  alone for 3D. Does this generalize to 4D / point tracking? Open.
- **TTT for first-order tracking outputs.** The user's planning note
  asks about first-order outputs (velocity, camera dynamics). TTT is
  a candidate substrate — the update could explicitly track time
  derivatives. Unexplored.

## Connections

- **For the user's project (Topic 3 of `raw/notes/`):** TTT is the
  most concrete fast-weight mechanism for the "compressed global
  context" half of Caricature 3 (slow planner + fast controller).
  Specifically, the *planner* could maintain TTT state, and the
  *controller* could read it as a per-frame forward.
- **Cross-source pattern:** TTT joins
  [[train-inference-mismatch]] as another recurring meta-pattern in
  this wiki — when the inference-time regime differs from training,
  fast-weight adaptation is a structural fix.
