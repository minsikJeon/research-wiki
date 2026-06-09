---
type: source
source_type: paper
title: "πR²: Reactive Real-time Flow Policies"
authors:
  - Anonymous (CoRL 2026 submission)
year: 2026
venue: CoRL 2026 (under review, double-blind)
url: https://reactive-real-flow.github.io
raw_path: papers/CoRL2026_p_fastslow.pdf
status: ingested
tags: [robotics, manipulation, vla, flow-matching, diffusion-forcing, action-chunking, real-time, asynchronous-control, fast-slow-policy, latency-adaptive]
sources: []
related:
  - "[[pi-r-squared]]"
  - "[[diffusion-forcing]]"
  - "[[fast-slow-policy]]"
  - "[[rtc]]"
  - "[[black-2025-training-time-rtc]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[action-chunking]]"
  - "[[asynchronous-control]]"
  - "[[train-inference-mismatch]]"
  - "[[vla]]"
  - "[[flow-matching]]"
created: 2026-06-08
updated: 2026-06-08
---

# πR²: Reactive Real-time Flow Policies

**Authorship note (private to this wiki):** anonymous CoRL 2026
submission; the source page is anonymized to respect the venue's
double-blind policy. The architecture's relationship to π0/π0.5/π0.6
(Physical Intelligence) and GR00T-N1.7 (NVIDIA) makes the affiliation
*evident from the experimental setup but not from authorship*; we don't
attempt to deanonymize.

## TL;DR

A two-mechanism modification to **any** flow-matching VLA that makes it
*real-time and closed-loop reactive*:

1. **Asynchronous slow/fast channel split.** Vision-language features
   (slow, ~60 ms) refresh in a background thread and feed the action
   head as a *cached* embedding. Proprioception (joints, EE pose,
   torques, fingertip forces — fast, μs) refreshes every control tick.
   Action denoising conditions on **fresh proprioception + stale vision**.
2. **Latency-adaptive flow schedule** (= "diffusion forcing" applied to
   action chunking). The action chunk is divided into front (clamped
   clean, in-flight) / interior (ramped noise, emitted one slot per call)
   / tail (pure noise, fresh slots). One NFE per call instead of `K`.

Result: closed-loop replanning at 25 Hz on GR00T-N1.7 / A5000 (~40 ms
per call) — roughly **4× faster** than the original model's 7 Hz, while
fine-tunable from any pretrained policy with no backbone change. Beats
Train-Time RTC by up to 50% relative on reactivity-sensitive real-world
tasks (Tidy Up Book).

## Why it matters

This is **directly the architecture for the user's Caricature 3** (slow
planner + fast controller) framed in `raw/notes/` — instantiated for a
robotics-control problem and validated on a real bimanual robot. Three
things make it particularly load-bearing:

- **Concrete realization of fast-slow policies.** Prior work (Helix,
  Gemini Robotics, GR00T-N1) advocated the split conceptually but kept
  the slow brain in a separate model. πR² shows the split can be
  *internal to a single VLA* with a one-line architectural change
  (per-token AdaLN modulation) — no separate model, no new training
  infrastructure.
- **Diffusion forcing's first practical robotics deployment.**
  [[chen-2024-diffusion-forcing]] showed per-position noise schedules
  work for video/planning; πR² shows they fix the *latency* problem in
  large VLAs while preserving the flow-matching expressiveness.
- **Sharpens the picture of RTC variants.** πR² is a proper superset of
  [[rtc]]/[[black-2025-training-time-rtc]]: RTC's "freeze prefix +
  inpaint rest" is the `s = 1/(H − 2d) → 0` corner of πR²'s ramp
  schedule. The paper makes this explicit.

## Key claims

- **Reactivity-vs-latency is the central bottleneck of modern VLAs.**
  GR00T-N1.7 on A5000: image preprocess + VLM ≈ 60 ms; DiT denoising
  (K=4) ≈ 80 ms. Per-call cost ≈ 140 ms ≈ 7 control ticks at 50 Hz —
  i.e., **>90% of committed actions execute open-loop on stale vision**.
- **Slow/fast asymmetry is exploitable.** Proprioception (joints,
  torques, fingertip forces) is "orders of magnitude faster to process
  than images or text"; **for dynamic tasks proprioception carries
  sufficient information for local reactive corrections** (responding
  to unexpected contact / compensating for slip), while vision/language
  provide global context for coarse motion plans. Disentangling the
  conditioning paths is enough to expose this asymmetry to the policy.
- **Per-position noise (Diffusion Forcing-style) collapses K denoising
  steps to 1.** With ramp `τ⋆^d(p) = clamp(0, (p−d)/(H−2d), 1)`, the
  front `d` slots are clean, the interior ramps clean→noise over
  `H−2d` slots, and the back `d` slots are pure noise. One Euler
  substep "rotates" the buffer right by `d` slots: positions `[d,2d)`
  hit τ=1 and are emitted; back gets `d` fresh-noise slots appended.
  Schedule reproduces exactly.
- **Training matches the inference regime.** Sample `d ∼ Unif{1,...,dmax}`
  per batch; clamp front `d` slots to GT (excluded from loss via
  `m_p = 1[p ≥ d]`); ramp interior + tail get noised + supervised. With
  20% probability train on standard flow (τ ∼ Unif[0,1] shared, no mask)
  so the same network can also warm-start the buffer from pure noise at
  episode start.
- **Slow-channel delay is *part of training*.** Vision/text latency
  `d_vlm ∼ Unif{0,...,dmax_vlm}` is supplied via a learned scalar
  embedding added to the slow representation; at deploy time the
  *measured* `d_vlm` uses the same embedding. The policy is taught to
  *expect* stale vision.
- **Architecturally a one-liner.** Per-position `(γ_p, β_p)` AdaLN
  conditioning (one pair per chunk position rather than one shared)
  is the only change to the DiT block. Attention, MLP, VLM backbone,
  slow/fast paths — all unchanged. **Existing pretrained VLAs fine-tune
  in.**

## Methods

### Building blocks (§3.1)

- **Action chunking flow matching:** chunk `x_1 = (a_t,...,a_{t+H−1})`,
  K denoising steps, interpolation `x_t = (1−t)ε + t·x_1`, FM loss
  `L_FM = E[‖v_θ(x_t, t) − (x_1 − ε)‖²]`. All `H` positions share one
  noise level `t`.
- **Diffusion forcing** ([[chen-2024-diffusion-forcing]]) — generalize
  to **per-position** noise `τ_p ∈ [0,1]^H`. Model predicts per-position
  velocity `v_θ(x_τ, τ, o)`.
- **Streaming diffusion** = one instantiation: linearly increasing `τ`
  across the chunk so each denoising step incorporates a fresh
  observation. πR² is another instantiation tuned for VLA latency.

### Proprioception-reactive diffusion forcing (§3.2)

VLA observation `o_t = (s_t, I_t, T_t)` = proprioception, image,
language. πR²:

- Vision+language path: VLM backbone in a **background thread**;
  finishes when complete; refreshes a cached slow feature.
- Proprioception path: small MLP every control tick; produces a fresh
  state embedding.
- DiT action head: per-call conditions on `(slow_cached, proprio_fresh)`
  and runs **1 NFE** over the chunk (vs K=4 in standard chunked
  denoising).
- Per-call action-head cost ≈ 20 ms (independent of backbone size).
- *Asynchronous vision/text processing*: slow feature is `d_vlm` ticks
  stale; the model is trained with this staleness so it's not OOD.

### Latency-adaptive staircase schedule (§3.3)

For target inference delay `d`:

```
τ_p^⋆,d = { 1                  if 0 ≤ p < d
          { 1 − (p−d)/(H−2d)   if d ≤ p < H−d
          { 0                  if H−d ≤ p ≤ H−1
```

- **Front `[0, d)`** = in-flight actions clamped clean (inpaint
  conditioning — mirrors training-time RTC).
- **Interior `[d, H−d)`** = linear ramp clean→noise, slope `1/(H−2d)`.
- **Tail `[H−d, H)`** = pure-noise slots to be filled by future calls.

**Why a ramp, not a flat τ in the interior:** at `d=0` it collapses to
streaming diffusion (linear ramp across full chunk); at `d=H/2` it
collapses to standard chunked flow with `d`-action clean prefix
(= [[black-2025-training-time-rtc]]'s scheme). πR² is the
diffusion-forcing generalization that interpolates between them.

### Inference cycle (§3.3, Fig 2)

Each call applies **one Euler substep**:
`x_p ← x_p + Δτ_p · v_θ(x, τ, o)`, with region-dependent advances
`Δτ_p` chosen so the schedule shifts right by `d` slots: positions
`[d, 2d)` reach τ=1 and are emitted; buffer rotates; `d` fresh-noise
slots appended at back. Schedule reproduces exactly when `d` holds
steady; when `d` changes, the `Δτ_p` pattern adapts.

### Compatibility (§3.3)

Plugs into any DiT-style VLA action head with **one-line change**:
AdaLN-zero conditioning becomes **per-position** (`(γ_p, β_p)` per chunk
position rather than one shared pair). Same parameter count. All other
components unchanged. Fine-tunes from any pretrained VLA.

## Results (headline)

### Simulation (Leap Cube Reorientation, MuJoCo Playground)

- **No-delay sweep over execution horizon `h ∈ {1, 2, 4, 8}`** (Fig 3,
  left): standard flow degrades from 0.79 (h=1) → 0.38 (h=8); πR² stays
  at 0.79 across h. Amortizing denoising across calls *does not*
  sacrifice quality.
- **Under VLA-level inference delay** (Fig 3, right): πR² with async
  processing dominates at `d₀ = 2, 3`; robust to visual delay
  (πR² 0.32 vs flow 0.21 / RTC 0.18).

### Real-world (xArm6 + XHand, fine-tuned GR00T-N1.7)

| Method | d (ticks) | Don't Spill SR | Tidy Up Book SR |
|---|---|---|---|
| Flow, synchronous (h=10) | 4–5 | 4/20 | 4/20 |
| Flow, naive async + TE | 4–5 | 7/20 | 7/20 |
| Flow, **Train-Time RTC** | 5 | 9/20 | 8/20 |
| **πR²** | **1–2** | **10/20** | **12/20** |

- πR² operates at the per-control-tick limit (`d=1` at 25 Hz).
- Largest gap on **reactivity-sensitive** task Tidy Up Book (≈50%
  relative gain) — picking up a book by scraping a pile requires
  continuous contact reactivity.
- Train-Time RTC is the strongest baseline, but πR² wins on every metric.

## Limitations / open questions

- **External latency not addressed.** Communication delays between
  inference server and robot client (e.g., remote-inference setups) are
  out of scope; only model-internal latency is handled.
- **Base policy architecture unchanged.** Dedicated attention heads for
  proprioception tokens (rather than the slow/fast split via cached
  features) is left for future work and might amplify reactivity further.
- **Slow-channel staleness budget bounded by training.** `d_vlm` is
  sampled `∼ Unif{0,...,dmax_vlm}`; deployment beyond `dmax_vlm` would
  be OOD. Paper doesn't quantify the degradation curve.
- **Single base VLA (GR00T-N1.7).** Π0/π0.5/π0.6 compatibility is
  claimed structurally (DiT + AdaLN-zero) but not measured.
- **No comparison with streaming-action-chunking variants** that use
  small models (SmolVLA's asynchronous execution, Streaming Diffusion
  Policy) — these solve the same problem at a different scale.
- **No long-horizon (>200 ms) jitter study.** Real-world tasks at the
  per-control-tick limit run cleanly but jitter under network spikes
  isn't characterized.

## Connections

- **Method page:** [[pi-r-squared]] (created in this ingest).
- **Underlying mechanism:** [[chen-2024-diffusion-forcing]]
  ([[diffusion-forcing]]) — per-position noise schedules; πR² is the
  first practical VLA deployment of this.
- **Action chunking & RTC family:**
  - [[zhao-2023-act]] — chunking origin.
  - [[chi-2024-diffusion-policy]] — diffusion as visuomotor policy.
  - [[black-2025-rtc]] — inference-time inpainting.
  - [[black-2025-training-time-rtc]] — training-time prefix conditioning,
    the πR² staircase's `d/H/0` corner.
- **Concept pages:** [[fast-slow-policy]] (created), [[action-chunking]]
  (updated), [[asynchronous-control]] (updated),
  [[train-inference-mismatch]] (updated).
- **Cited concurrent / related:** FASTER (diffusion-forcing-inspired
  horizon-aware schedule, but open-loop), VLASH/A2C2, VLASH —
  prefix-conditioning correction heads; all are partial solutions πR²
  unifies.
- **Cited base VLAs:** GR00T-N1.7 (NVIDIA), π0/π0.5/π0.6 (Physical
  Intelligence), OpenVLA, RT-1, RT-2, MiniVLA, SmolVLA, NaVILA.
- **Connection to perception side:** the slow/fast split mirrors
  [[zhang-2026-loger]]'s SWA+TTT (lossless local fast + compressed
  global slow) and the user's Caricature 3 architecture
  (Point4D-as-planner + lightweight head as controller). The control
  side has now caught up to (and arguably surpassed) the perception side
  in operationalizing this pattern.

## Citation

Anonymous Authors. (2026). *πR²: Reactive Real-time Flow Policies.*
Submitted to the 10th Conference on Robot Learning (CoRL 2026).
Project website: https://reactive-real-flow.github.io.
