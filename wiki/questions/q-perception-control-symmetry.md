---
type: question
title: Is the πR² staircase shape right for online 3D point tracking?
status: open
tags: [diffusion-forcing, point-tracking, online-tracking, fast-slow-policy, perception, robotics, design-question]
sources:
  - "[[anon-2026-pi-r-squared]]"
  - "[[chen-2024-diffusion-forcing]]"
  - "[[anon-2026-point4d]]"
related:
  - "[[diffusion-forcing]]"
  - "[[pi-r-squared]]"
  - "[[fast-slow-policy]]"
  - "[[3d-point-tracking]]"
  - "[[online-vs-offline-tracking]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-10
updated: 2026-06-10
---

# Is the πR² staircase shape right for online 3D point tracking?

## The question

[[anon-2026-pi-r-squared]] introduces a **latency-adaptive staircase
schedule** for action chunks:

```
τ_p =  1                  if 0 ≤ p < d            FRONT (clean, in-flight)
       1 − (p−d)/(H−2d)   if d ≤ p < H−d          INTERIOR (linear ramp)
       0                  if H−d ≤ p < H          TAIL (pure noise)
```

This is the control-side instance of a much more general design choice:
**which 2D `M × T` grid of [[diffusion-forcing]] noise levels is right
for a given streaming task?**

The user's MVP design (`2026-06-08_MiddleGround_MVP.md`) ports this
exact staircase to 3D point tracking. The question is whether **the
perception problem prefers a different schedule**.

## Why it's not obviously the same

Three asymmetries between robotics control and 3D-tracking perception:

| Property | Robotics control (πR²) | 3D point tracking |
|---|---|---|
| Output stream | actions consumed at the front | predictions consumed at every position |
| "Future" constraint | actions far in future are speculative | future-frame predictions are *answerable* if camera sees there |
| Backward causality | strict — robot can't unmove | none — track can be retroactively refined |
| Penalty structure | latency-sensitive | jitter-sensitive |

In particular, perception has **no equivalent of "committed actions"** —
nothing has been physically actuated. A noise-tail-pure-noise design
may waste capacity that could be spent on bidirectional refinement.

## Candidate alternative schedules

1. **πR² staircase (verbatim).** Front clamped, ramp, noise tail.
   Treat past frames as "executed" (locked) and refine forward.
2. **Causal diffusion forcing without a tail.** All positions get a
   ramp from clean (oldest) to noisy (newest), no pure-noise lookahead.
   Discards the look-ahead capacity to use compute on present.
3. **Streaming diffusion (linear ramp across full chunk).** No front
   clamp, no tail boundary. Treats every position symmetrically. May
   waste compute on past frames.
4. **Bidirectional staircase.** Front clamped + center most-refined +
   tail noised. Exploits perception's lack of strict forward causality.
   No precedent in current literature.

## What would settle it

- **Ablation:** swap schedules under fixed compute budget on TAPVid-3D
  or a streaming benchmark; measure per-frame latency vs trajectory
  accuracy and per-frame jitter.
- **Theory:** under what task metric (delay-aware vs accuracy-aware)
  does each schedule dominate? Connect to [[streaming-accuracy]] for
  the formal version.
- **Hybrid:** does combining staircase + bidirectional refinement
  beat either?

## Status

Open. The MVP doc commits to (1) for V0 because πR² has the most
direct evidence; (4) is the open research lever.
