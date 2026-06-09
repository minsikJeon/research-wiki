---
type: method
title: πR² (Reactive Real-time Flow Policies)
status: growing
tags: [robotics, manipulation, vla, flow-matching, diffusion-forcing, action-chunking, real-time, fast-slow-policy, asynchronous-control, latency-adaptive]
sources:
  - "[[anon-2026-pi-r-squared]]"
related:
  - "[[diffusion-forcing]]"
  - "[[fast-slow-policy]]"
  - "[[rtc]]"
  - "[[training-time-rtc]]"
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[vla]]"
created: 2026-06-08
updated: 2026-06-08
---

# πR² (Reactive Real-time Flow Policies)

A drop-in modification to any DiT-style flow-matching VLA action head
that turns it into a **real-time, closed-loop, reactive policy** while
preserving the pretrained backbone. Two orthogonal mechanisms:
asynchronous vision/text processing (slow channel) + latency-adaptive
flow schedule via diffusion forcing.

## One-line summary

DiT action head with **per-position AdaLN-zero conditioning** (one
`(γ_p, β_p)` per chunk position), trained with a staircase
diffusion-forcing schedule + random inference delay + random
slow-channel staleness; at inference, run **one denoising substep per
control tick** while the VLM refreshes asynchronously.

## Inputs / outputs

- **In (training):** `(obs, action_chunk)` pairs from demos. Per-batch
  random `d ∼ Unif{1, ..., d_max}`, random
  `d_vlm ∼ Unif{0, ..., d_max_vlm}`.
- **In (inference, each call):** fresh proprioception `s_t`, cached
  vision/text feature `(I_{t−d_vlm}, T_{t−d_vlm})`, current inference
  delay `d` (measured wall-clock per call), current chunk buffer state.
- **Out:** one denoising substep over the chunk; emits `d` clean
  actions; appends `d` fresh-noise slots at the back.

## How it works

### Architecture (the one-line change)

The only architectural modification to a DiT-style action head: AdaLN
conditioning becomes **per-position**.

```
Standard DiT block (e.g. π0.6 action expert):
    h = AdaLN(h, τ)           # one shared τ for the whole chunk
    h = h + attn(h)
    h = h + mlp(h)

πR² DiT block:
    h_p = AdaLN(h_p, τ_p)     # one τ_p per chunk position
    h = h + attn(h)
    h = h + mlp(h)
```

Same parameter count. AdaLN-zero generates `(γ_p, β_p)` per position
from `τ_p`. Attention / MLP / VLM backbone / slow-fast paths
unchanged.

### Slow/fast channel split

- **Slow channel:** image preprocess + VLM forward (~60 ms on A5000
  with GR00T-N1.7). Runs in a background thread; cached feature
  refreshed when complete.
- **Fast channel:** proprioception through a small MLP every control
  tick.
- Action head conditions on **(slow_cached, proprio_fresh)** at every
  call. Slow feature is `d_vlm` ticks stale; the model is trained with
  `d_vlm` randomized and embedded as a scalar input, so staleness is
  in-distribution.

### Latency-adaptive staircase schedule

For target inference delay `d` (integer ticks):

```
τ_p^⋆,d = { 1                  0 ≤ p < d           (front: clean, in-flight)
          { 1 − (p−d)/(H−2d)   d ≤ p < H−d         (interior: ramp clean→noise)
          { 0                  H−d ≤ p ≤ H−1       (tail: pure noise)
```

Three roles:
- **Front `[0, d)`**: clamped-clean actions already in flight; conditions
  generation downstream. Mirrors training-time RTC.
- **Interior `[d, H−d)`**: linear ramp; slope `s = 1/(H−2d)`.
- **Tail `[H−d, H)`**: pure-noise slots awaiting future denoising.

### Inference cycle (one call = one Euler substep)

```
1. measure current d
2. read fresh proprio s_t; check if slow feature refreshed
3. update τ for the buffer to target τ⋆,d
4. one Euler step on the action head:
       x_p ← x_p + Δτ_p · v_θ(x, τ, o)
       τ_p ← τ_p + Δτ_p
   with Δτ_p chosen so:
     • positions [d, 2d) reach τ=1 and are EMITTED as clean actions
     • the rest of the buffer rotates right by d slots
     • d fresh-noise slots are appended at the back
5. enqueue emitted actions for execution
6. return; next call in 1 control tick
```

When `d` holds steady, the schedule reproduces exactly. When `d`
changes between calls, the `Δτ_p` pattern adapts.

### Training

- Sample `d ∼ Unif{1, ..., d_max}` per batch; build `τ⋆,d`.
- Front `d` slots: fill with GT actions; exclude from loss
  (`m_p = 1[p ≥ d]`).
- Interior / tail: noise at staircase levels; supervise per-position MSE.
- Apply symmetric jitter `τ_p ← clip(τ_p + δ_p, 0, 1)` with
  `δ_p ∼ Unif[−j, j]` to absorb per-call `d` jitter.
- **20% probability**: train on standard flow (`τ ∼ Unif[0,1]`, shared,
  no mask) so the model can also warm-start from pure noise at
  episode start.
- Sample `d_vlm ∼ Unif{0, ..., d_max_vlm}`; embed via learned scalar
  embedding added to slow representation.

## Where it's been applied

- **Sim:** Leap Cube Reorientation (MuJoCo Playground). 200 trajectories.
- **Real:** xArm6 + XHand bimanual, fine-tuned from GR00T-N1.7. Two
  contact-rich dexterous tasks: Don't Spill (carrying a ball in a bowl)
  and Tidy Up Book (scrape-grasping a book from a pile).

## Known limitations

- **External (network) latency not handled.** Model-internal latency
  only.
- **Backbone unchanged.** Dedicated proprio attention heads could
  amplify reactivity further; left to future work.
- **Single base policy validated** (GR00T-N1.7). Generalization to
  π0.5/π0.6 is structurally compatible but not measured.
- **No per-component ablation of the slow/fast split.** The paper
  reports the joint effect of staircase + async, not either alone in
  isolation on the real robot.
- **No external comparison vs SmolVLA / Streaming Diffusion Policy** —
  related but at different scale; would clarify scaling story.

## Related methods

- **Immediate parent / generalization target:** [[training-time-rtc]] —
  the `s_interior = 0` corner of πR²'s ramp schedule (flat τ=0 across
  interior + tail).
- **Inference-time predecessor:** [[rtc]] — same problem solved with
  inpainting, but pays per-step ΠGDM cost.
- **Underlying mechanism:** [[diffusion-forcing]] / [[chen-2024-diffusion-forcing]] —
  per-position noise schedules.
- **Fast-slow concept:** [[fast-slow-policy]] — πR² is the
  most-deployed instance to date.
- **Open-loop diffusion-forcing baseline:** FASTER (horizon-aware
  schedule but no closed-loop replanning).
- **Inpaint correction siblings:** A2C2, VLASH (lightweight correction
  heads on a single future action).
- **Streaming-policy siblings:** SmolVLA's async execution, Streaming
  Diffusion Policy — solve the same problem at smaller scale.
