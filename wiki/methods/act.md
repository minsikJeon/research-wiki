---
type: method
title: ACT (Action Chunking with Transformers)
status: growing
tags: [robotics, manipulation, imitation-learning, action-chunking, transformer, vae, bimanual, aloha]
sources:
  - "[[zhao-2023-act]]"
related:
  - "[[action-chunking]]"
  - "[[diffusion-policy]]"
  - "[[rtc]]"
  - "[[pi-r-squared]]"
  - "[[chelsea-finn]]"
  - "[[sergey-levine]]"
created: 2026-06-08
updated: 2026-06-08
---

# ACT (Action Chunking with Transformers)

The originating policy architecture for the [[action-chunking]]
paradigm. CVAE-style transformer that predicts `k`-action chunks
conditioned on multi-camera RGB + joint state.

## One-line summary

Multi-cam ResNet → transformer encoder → transformer decoder emits a
`k = 100` action chunk in parallel; CVAE structure handles multi-modal
demos; temporal ensembling at inference smooths boundaries.

## Inputs / outputs

- **In:** 4 RGB cameras (top, wrist L/R, front) + 14-dim joint state.
- **Out:** chunk of `k = 100` joint-target actions; execute open-loop
  for the chunk, then re-query.

## How it works

### CVAE

- **Encoder `q(z | a_{0:k}, joint)`**: transformer over GT action chunk
  + current joint state → mean/variance of latent `z ∈ R^32`. KL
  regularized against `N(0, I)`.
- **Decoder `π(a_{0:k} | obs, z)`**:
  - Per-camera ResNet-18 → spatial features.
  - Concatenate features + joint state + `z` → transformer encoder.
  - Transformer decoder cross-attends to encoder output, emits `k`
    actions in parallel.

### Test-time inference

- Sample `z = 0` (CVAE prior mode) for deterministic deployment.
- **Temporal ensembling**: at each step `t`, average actions from all
  chunks predicted in `{t − k + 1, ..., t}` with exponential weights
  `w_i ∝ exp(−m·i)`, `m = 0.01`.

### Training

- Pure imitation: action L1 + KL on `z`.
- 10 min of demos per task → ~50 epochs.

## Where it's been applied

- **ALOHA bimanual rig** (4 arms, ~\$20k) — original platform.
- **6 fine-manipulation tasks**: thread zip cable tie, NIST board #2,
  juggle pingpong, open lid, slot battery, manipulate tape. 80–90% SR.
- **Successors** (same lead): ALOHA Unleashed (Zhao 2024) — scaled
  recipe. Not yet ingested.

## Known limitations

- **Within-chunk open loop.** No reactivity inside the `k = 100`
  window — the exact limitation [[rtc]] / [[pi-r-squared]] later fix.
- **Temporal ensembling produces invalid actions** when adjacent chunks
  represent different demo modes — averages don't honor constraints.
- **CVAE replaced.** Diffusion ([[diffusion-policy]]) and flow matching
  ([[rtc]] base policies) handle multimodality more naturally.
- **Custom 6-task benchmark.** No shared evaluation protocol.
- **Hardware-specific tuning.** Latency-critical leader-follower setup.

## Related methods

- **Concept origin:** [[action-chunking]] — ACT introduces and
  champions this paradigm.
- **Direct policy successor:** [[diffusion-policy]] — replaces CVAE
  with DDPM; same chunking skeleton.
- **Boundary-smoothness successors:** [[rtc]], [[training-time-rtc]],
  [[pi-r-squared]] — replace temporal ensembling with inpainting /
  diffusion-forcing schedules.
- **Hardware successors:** ALOHA Unleashed; π0/π0.5 form-factor uses
  the same arm class.
