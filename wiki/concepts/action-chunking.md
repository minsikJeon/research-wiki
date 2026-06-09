---
type: concept
title: Action Chunking (for Robot Policies)
status: growing
tags: [robotics, manipulation, vla, policy, action-chunking, real-time]
sources:
  - "[[zhao-2023-act]]"
  - "[[chi-2024-diffusion-policy]]"
  - "[[black-2025-rtc]]"
  - "[[black-2025-training-time-rtc]]"
  - "[[anon-2026-pi-r-squared]]"
related:
  - "[[asynchronous-control]]"
  - "[[vla]]"
  - "[[rtc]]"
  - "[[training-time-rtc]]"
  - "[[pi-r-squared]]"
  - "[[act]]"
  - "[[diffusion-policy]]"
  - "[[diffusion-forcing]]"
  - "[[flow-matching]]"
created: 2026-05-28
updated: 2026-06-08
---

# Action Chunking (for Robot Policies)

## Definition

A control paradigm where the policy outputs **a chunk of `H` future
actions** per inference call, not a single action: `π(A_t | o_t)` where
`A_t = [a_t, ..., a_{t+H−1}]`. Only the first `s ≤ H` actions are
executed before re-querying with a new observation. Originated with
[[act]] ([[zhao-2023-act]], RSS 2023) and is now standard in dexterous
manipulation VLAs.

The original motivation in ACT was **compounding-error suppression** for
imitation learning: predicting `H` actions at once reduces the effective
horizon from `T` to `T/H`, so small per-step errors don't compound as
disastrously. Reactivity loss is then traded back via temporal ensembling
(ACT), inpainting (RTC), or diffusion forcing (πR²).

## Why it matters

Two related benefits and two related problems:

**Benefits:**
- **Temporal consistency.** A coherent multi-step strategy survives
  observation noise.
- **Throughput amortization.** One slow VLA forward pass produces `s`
  controller steps of actions, amortizing inference cost.

**Problems:**
- **Reactivity loss.** A long execution horizon `s` means stale
  responses to new observations.
- **Mode jumps at chunk boundaries.** Adjacent chunks may pick different
  "strategies" from the learned distribution, producing discontinuities.
  Out-of-distribution accelerations are particularly damaging because
  they shift the dynamics distribution at train vs. test.

## How it's typically handled

- **Long execution horizon (s ≈ H/2).** Default in many works
  (ACT, π0, π0.5, DP). Trades reactivity for stability.
- **Temporal ensembling ([[act]]).** Average overlapping action
  predictions from multiple chunks with exponential weights. Reduces
  acceleration but produces invalid actions outside the learned
  distribution.
- **Inference-time chunking with inpainting ([[rtc]] /
  [[black-2025-rtc]]).** Pose the boundary problem as inpainting:
  freeze the prefix that's already committed, inpaint the rest
  consistent with the previous chunk + new observation. No retraining;
  pays a per-step ΠGDM cost.
- **Training-time prefix conditioning ([[training-time-rtc]] /
  [[black-2025-training-time-rtc]]).** Move the prefix conditioning
  into training: per-position AdaLN-zero `τ`, clean prefix at `τ = 1`,
  loss masked to postfix. Inference is then a standard `K`-step flow
  decoder — no ΠGDM. Outperforms inference-time RTC at `d ≥ 2`.
- **Latency-adaptive diffusion-forcing schedule
  ([[pi-r-squared]] / [[anon-2026-pi-r-squared]]).** Generalize the
  prefix clamp to a **staircase**: clean front (`d` slots) + ramped
  interior (clean → noise) + pure-noise tail (`d` slots). Plus async
  VLM processing (the slow channel) so inference cost per call drops
  to ~1 NFE on the action head + cached vision. Closes the
  reactivity-vs-latency gap most fully so far.

## Connection to perception-side problems

Action chunking + asynchronous inference has the *same shape* as the
**sliding-window inference with cross-window initialization** problem
we mapped in [[spatialtracker-v2]]:

| Perception                          | Robot control                       |
|-------------------------------------|-------------------------------------|
| Window of frames                    | Chunk of actions                    |
| Overlap region between windows      | Overlap between adjacent chunks     |
| Re-initialize new window's tracks   | Re-initialize new chunk's actions   |
| Train on clean GT queries           | Train on clean ground-truth chunks  |
| Inference uses noisy carry-over     | Inference uses prev-chunk overlap   |
| → [[train-inference-mismatch]]      | → [[train-inference-mismatch]]      |

RTC handles the mismatch explicitly by **freezing the prefix that
inference latency guarantees will execute, then inpainting the rest**.
SpaTrackerV2 handles the analogous problem by **picking the most-
confident overlap frame and synthesizing a fresh query** — implicit and
heuristic, not structural. The contrast is one of the more interesting
cross-domain patterns in the wiki.

## Open questions

- **Optimal execution horizon `s` is task-dependent.** [[rtc]]'s
  Figure 5 shows only RTC and BID benefit from shorter `s`; standard
  chunking penalizes too-short `s`.
- **Inpainting vs. ensembling vs. autoregression.** Three families of
  chunk-boundary solutions; the empirical comparison in RTC is partial.
  A unified evaluation across diffusion / flow / transformer policies
  is still open.
- **Chunking + asynchrony in non-generative VLAs.** RTC requires a
  generative policy. Whether transformer-decoder VLAs need a different
  inference-time mechanism is unsettled.
