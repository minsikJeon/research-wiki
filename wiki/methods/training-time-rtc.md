---
type: method
title: Training-Time RTC (Training-Time Action Prefix Conditioning)
status: growing
tags: [robotics, manipulation, vla, flow-matching, action-chunking, real-time, asynchronous-control, training-recipe]
sources:
  - "[[black-2025-training-time-rtc]]"
related:
  - "[[rtc]]"
  - "[[pi-r-squared]]"
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[flow-matching]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-08
updated: 2026-06-08
---

# Training-Time RTC

Training-time replacement for the inference-time inpainting algorithm of
[[rtc]]. Same input/output interface — drop-in — but with zero
inference-time overhead from guidance / VJP computation.

## One-line summary

Per-position AdaLN-zero `τ` conditioning + clean prefix at `τ = 1` +
loss masked to postfix. At inference: prefix = previously committed
actions; postfix denoised normally over `K` steps.

## Inputs / outputs

Same as [[rtc]] — designed as drop-in:

- **In (training):** chunk `A_t`, observation `o_t`, sampled
  `d ∈ {0, ..., d_max}`.
- **In (inference):** observation `o_t`, action prefix
  `A_{t : t + d}` (from previous chunk's overlap), measured `d`.
- **Out:** action postfix `A_{t + d : t + H}` (the K-step denoised
  result over the postfix positions only).

## How it works

### Three minimal training-time changes

1. **Per-position flow-matching timestep.** AdaLN-zero conditioning of
   each DiT block becomes per-position: one `(γ_p, β_p)` pair per
   chunk position rather than one shared pair. Same total parameter
   count.
2. **Clean prefix at `τ = 1`.** For the `d` prefix positions, feed
   ground-truth actions and set `τ_p = 1`. The model conditions on
   them but does not denoise them.
3. **Loss masking.** Compute the flow-matching loss only on postfix
   positions (`p ≥ d`).

### Loss (Algorithm 1, paper)

```python
prefix_mask = arange(H) < delay              # delay sampled per batch
time = where(prefix_mask, 1.0, time)         # τ = 1 on prefix
x_t = time * action_chunk + (1 − time) * noise
pred_v = model(observation, x_t, time)
loss   = (pred_v − (action_chunk − noise))²
loss   = sum(loss · ¬prefix_mask) / sum(¬prefix_mask)
```

### Delay sampling

- `d ∼ {0, 1, ..., d_max}` with **exponentially decreasing** weights
  (higher delays get less training supervision, because the inpainting
  algorithm is more robust to larger prefixes anyway).
- In real-world experiments: `d_max = 10`; supports up to 200 ms
  latency on a 50 Hz robot.

### Inference (Algorithm 1, sampler)

```python
prefix_mask = arange(H) < delay
x_t = N(0, 1)
for k in range(num_steps):
    x_t = where(prefix_mask, action_prefix, x_t)   # pin prefix
    time = where(prefix_mask, 1.0, time)           # τ = 1 on prefix
    v_t = model(o, x_t, time)
    x_t = x_t + dt * v_t
    time = time + dt
return x_t
```

Drop-in replacement for the standard `K`-step DDIM/flow inference loop.
No ΠGDM, no VJP, no guidance clip.

## Where it's been applied

- **Sim:** Dynamic Kinetix (same benchmark as inference-time RTC,
  Kinetix [25]). 4-layer MLP-Mixer policy, `H = 8`.
- **Real:** π0.6 base model, two tasks — box building (cardboard
  assembly) and espresso making (grinding, tamping, extracting,
  pouring). xArm with H100 inference server.

## Known limitations

- **Hard prefix only.** Cannot softly incorporate additional overlapping
  actions beyond the prefix the way inference-time RTC can. Less
  flexible at the cost of much cheaper inference.
- **Training-distribution of `d` matters.** Exponentially-decreasing
  weights trade large-`d` robustness for small-`d` performance.
  Training-time RTC is marginally worse at `d ∈ {0, 1}`.
- **Single point in the latency-schedule space.** πR² generalizes this
  to a continuous family of schedules indexed by `d`; training-time RTC
  is essentially the `s = 0` corner of that family.
- **No fast-slow split.** Still pays full VLM cost per call.

## Related methods

- **Predecessor:** [[rtc]] / [[black-2025-rtc]] — inference-time
  inpainting, generalizes to soft overlaps but pays guidance cost.
- **Successor / generalization:** [[pi-r-squared]] — staircase
  schedule with ramped interior; training-time RTC is its limiting
  case at `s = 1/(H − 2d) → 0` (flat τ=0 interior).
- **Concept pages:** [[action-chunking]], [[asynchronous-control]],
  [[train-inference-mismatch]].
- **Base policies:** π0.6 (Physical Intelligence) — first wiki ingest
  to reference π0.6 directly.
