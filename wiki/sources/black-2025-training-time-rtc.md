---
type: source
source_type: paper
title: "Training-Time Action Conditioning for Efficient Real-Time Chunking"
authors:
  - Black, Kevin
  - Ren, Allen Z.
  - Equi, Michael
  - Levine, Sergey
year: 2025
venue: arXiv (cs.RO) — Physical Intelligence
url: https://arxiv.org/abs/2512.05964
raw_path: papers/2512.05964v2.pdf
status: ingested
tags: [robotics, manipulation, vla, flow-matching, action-chunking, real-time, asynchronous-control, train-inference-mismatch, training-recipe]
sources: []
related:
  - "[[training-time-rtc]]"
  - "[[rtc]]"
  - "[[black-2025-rtc]]"
  - "[[anon-2026-pi-r-squared]]"
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[train-inference-mismatch]]"
  - "[[kevin-black]]"
  - "[[sergey-levine]]"
  - "[[physical-intelligence]]"
created: 2026-06-08
updated: 2026-06-08
---

# Training-Time Action Conditioning for Efficient Real-Time Chunking

## TL;DR

Drop-in replacement for inference-time [[rtc]] that **moves prefix
conditioning into training** rather than enforcing it via inference-time
ΠGDM inpainting. Three minimal changes to standard flow matching: (1)
per-position AdaLN modulation, (2) fix `τ_p = 1` on the prefix and feed
GT actions, (3) mask the loss to the postfix only. At inference, the
prefix is the previously committed actions, postfix is denoised normally
— **zero ΠGDM overhead**. Matches or beats inference-time RTC at typical
delays; outperforms it cleanly at higher delays.

## Why it matters

This is the **second of three Physical-Intelligence-led real-time
chunking papers** in this wiki. Together with [[black-2025-rtc]]
(inference-time) and [[anon-2026-pi-r-squared]] (joint training-time +
fast-slow + diffusion-forcing), it brackets the design space for VLA
real-time execution.

- Resolves a compute-cost limitation of [[rtc]]: ΠGDM guidance requires
  a vector-Jacobian product per denoising step, raising per-step latency.
  Training-time conditioning eliminates this overhead.
- Establishes that **the train-inference mismatch can be closed at
  training time without losing structural elegance** — same lever
  Point4D pulled on the perception side, now on the control side.
- Lays the foundation for πR²: training-time RTC's prefix-clamping is
  the limiting case of πR²'s staircase schedule when `d` saturates.

## Key claims

- **The "soft masking" of inference-time RTC is a band-aid.** It's
  computationally expensive (per-step VJP), hyperparameter-sensitive
  (the `β` clipping constant), and adds approximation error from the
  pseudoinverse linearization. Training-time conditioning is cleaner.
- **Conditioning on the *action prefix* directly is sufficient.** The
  prefix is the `d` actions guaranteed to execute before the new chunk
  lands. Training: feed clean prefix tokens at `τ = 1`, denoise postfix
  normally. Inference: prefix = previously committed actions, postfix
  is the new chunk. No inpainting math.
- **Three-line architectural change.** (1) per-position `τ`-conditioning
  via AdaLN-zero, (2) clean prefix tokens at `τ = 1`, (3) loss mask on
  postfix only.
- **Training delay `d` is sampled** `∼ {0,...,dmax}` with exponentially
  decreasing weights — fewer training samples at large delays because
  the inpainting algorithm is more robust there anyway.
- **Outperforms inference-time RTC at d ≥ 2** with widening gap as
  delay grows (Fig 3, Dynamic Kinetix). Slightly worse at d = 0, 1
  due to fewer training samples at small delays.
- **Real-world parity at speed.** On π0.6 / 50 Hz robot: 108 ms
  end-to-end latency for training-time RTC (d ≈ 5) vs 135 ms for
  inference-time RTC (d ≈ 7).

## Methods

### Setup

- Action chunking flow matching, prediction horizon `H`, execution
  horizon `s`, inference delay `d ∈ {0, 1, ..., max_delay}`. Action
  prefix `A_{t:t+d}` = the `d` overlapping actions taken from the
  previous chunk.
- Standard FM loss: `L(θ) = E[‖v_θ(A_t^τ, o_t, τ) − (ε − A_t)‖²]`.

### Training-time RTC (§IV)

Three minimal changes:

1. **Per-position flow-matching timestep.** The `τ` in AdaLN-zero
   conditioning of a DiT block becomes per-position (`τ_p`, one per
   action timestep). Same total parameter count.
2. **Clean prefix at `τ = 1`.** Use ground-truth actions for the prefix
   positions; set their `τ = 1`; the model conditions on them but
   doesn't denoise them.
3. **Loss masking.** Compute loss only on postfix outputs.

```
# Loss (from Algorithm 1)
prefix_mask = arange(H) < delay          # delay sampled per batch
time[prefix_mask] = 1.0
x_t = time * action_chunk + (1 − time) * noise
loss = (v_θ(o, x_t, time) − (noise − action_chunk))²
loss = sum(loss · ¬prefix_mask) / sum(¬prefix_mask)
```

### Inference (§IV)

- `action_prefix` is the actions from the previous chunk that will
  execute before this denoising completes.
- For `K` denoising steps: pad the action vector with prefix at the
  front; mask `time` so prefix positions stay at 1; integrate Euler
  steps for postfix positions only. Same input/output interface as
  inference-time RTC — drop-in.

## Results (headline)

### Simulation (Dynamic Kinetix, 8-action horizon, MLP-Mixer policy)

- **Across delays d ∈ {0, 1, 2, 3, 4}** (Fig 3): training-time RTC
  curves above inference-time RTC and well above naive async; the gap
  widens with delay. Both RTC variants beat synchronous baseline.

### Real world (xArm + π0.6, two tasks)

| Task | Synchronous | Inference-time RTC | Training-time RTC |
|---|---|---|---|
| Espresso making (SR) | 0.83 ± 0.05 | 0.93 ± 0.04 | 0.93 ± 0.04 |
| Box building (SR) | 0.55 ± 0.07 | 0.60 ± 0.07 | 0.60 ± 0.07 |
| Espresso duration (s) | ~150 | ~110 | ~110 |
| Box duration (s) | ~170 | ~125 | ~125 |

Training-time RTC matches inference-time RTC on success rate and speed
**without any inference overhead** (108 ms vs 135 ms end-to-end).

## Limitations / open questions

- **Less flexible than inference-time RTC** at deployment: only supports
  conditioning on a *hard* prefix corresponding to inference delay,
  whereas inference-time RTC can softly incorporate additional
  overlapping actions beyond the prefix.
- **Training distribution of `d`** has to be picked carefully to match
  expected deployment latency. The exponential weighting at higher d
  trades better high-delay performance for slightly worse low-delay.
- **No latency-adaptive schedule.** Training-time RTC is a single point
  in design space; [[anon-2026-pi-r-squared]]'s ramped schedule
  generalizes to a continuous family.
- **Fast-slow split is orthogonal but not addressed.** Inference still
  pays full VLM cost per call; the πR² async-vision idea is
  complementary (and adopted by πR²).
- **Slightly worse at zero / one-tick delay.** Fewer training samples
  at small `d` — could be fixed by uniform sampling at the cost of
  high-delay robustness.

## Connections

- **Method page:** [[training-time-rtc]] (created in this ingest).
- **Immediate predecessor:** [[black-2025-rtc]] / [[rtc]] —
  inference-time inpainting that this paper supersedes for delay > 2.
- **Generalization:** [[anon-2026-pi-r-squared]] — training-time
  prefix clamp + diffusion-forcing ramp interior; reduces to
  training-time RTC at the `s = 0` corner.
- **Concept update:** [[action-chunking]], [[asynchronous-control]],
  [[train-inference-mismatch]].
- **Authors:** [[kevin-black]] (lead, also led inference-time RTC),
  Allen Z. Ren, Michael Equi, [[sergey-levine]] (senior).
- **Org:** [[physical-intelligence]] (all four authors).
- **Base policies cited:** π₀.₆ for real-world (this is the first
  ingest with π₀.₆); π₀.₅, π₀ family broadly.

## Citation

Black, K., Ren, A. Z., Equi, M., & Levine, S. (2025).
*Training-Time Action Conditioning for Efficient Real-Time Chunking.*
arXiv:2512.05964v2. https://arxiv.org/abs/2512.05964
