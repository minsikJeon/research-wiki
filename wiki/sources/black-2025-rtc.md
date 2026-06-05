---
type: source
source_type: paper
title: "Real-Time Execution of Action Chunking Flow Policies"
authors:
  - Black, Kevin
  - Galliker, Manuel Y.
  - Levine, Sergey
year: 2025
venue: NeurIPS 2025
url: https://arxiv.org/abs/2506.07339
raw_path: papers/2506.07339v2.pdf
status: ingested
tags: [robotics, manipulation, vla, flow-matching, diffusion-policy, action-chunking, real-time, inference-time-algorithm, inpainting, asynchronous-control]
sources: []
related:
  - "[[rtc]]"
  - "[[action-chunking]]"
  - "[[vla]]"
  - "[[flow-matching]]"
  - "[[asynchronous-control]]"
  - "[[train-inference-mismatch]]"
  - "[[kevin-black]]"
  - "[[sergey-levine]]"
  - "[[physical-intelligence]]"
  - "[[uc-berkeley]]"
created: 2026-05-28
updated: 2026-05-28
---

# Real-Time Execution of Action Chunking Flow Policies (RTC)

## TL;DR

RTC poses the **real-time execution problem for action-chunking VLAs as
an inpainting problem at inference time.** While the policy is generating
the next chunk, the previous chunk is still executing — so the first `d`
(inference-delay) actions of the new chunk are **frozen** (already
committed) and the remaining `H − d` actions are **inpainted** to be
consistent with that frozen prefix, plus the overlap with the previous
chunk. Works on any diffusion / flow-based VLA out of the box; **no
retraining**. Tested on 12 new Kinetix dynamic tasks + 6 real bimanual
manipulation tasks with π0.5 base policy.

## Why it matters

This is a **brand-new research thread in this wiki** — robotics policy
execution, not perception. Plus it's a clean instance of the exact
**train/inference mismatch** pattern we mapped on the perception side
(e.g., [[spatialtracker-v2]]'s sliding-window inference vs. clean-query
training): action chunking policies are trained to predict full chunks
of `H` actions starting from "now", but at deployment under latency,
the actual operating regime is "first `d` actions are already
committed, fill in the rest while being consistent with them".

RTC's contribution is structural: instead of patching the symptom at
training, treat the runtime regime as inpainting with a learned generative
prior. **Inference-time-only structural change** — same lever Point4D
pulled in 3D point tracking, applied to robot control.

Also relevant for the user's MSR program directly: bimanual dexterous
manipulation is exactly what robotics labs care about.

## Key claims

- **Action chunking has latency-induced discontinuities.** Naive
  synchronous inference creates pauses at chunk boundaries; naive
  asynchronous inference produces out-of-distribution jumps (Fig 2).
  Temporal ensembling [Zhao et al., ACT] reduces acceleration but
  produces invalid actions.
- **Inpainting solves the cross-chunk continuity problem.** Frame
  the new chunk as: hard-freeze the `d` actions that will already be
  executed when this chunk lands; soft-mask the `H − s − d`
  intermediate actions (exponentially decaying guidance weight); freely
  generate the final `s` actions.
- **ΠGDM-based guidance.** Pseudoinverse-guidance correction added to
  the flow velocity field at every denoising step; with a learned
  velocity field `v`, the corrected field is `v + min(β, ...)`. The
  clipping constant `β` matters for stability with few denoising steps
  — they ablate this.
- **Soft masking improves cross-chunk continuity** vs. hard masking
  alone (Fig 4): the intermediate region uses exponentially decaying
  weights `c_i = (e^{i−1} − 1) / (H − s − d + 1)`.
- **Universal: any diffusion / flow VLA**, no retraining. Diffusion
  policies are convertible to flow at inference time.
- **20% faster than synchronous** baseline on real match-lighting task,
  smoother than temporal ensembling (Fig 1).
- **Robust to >300ms inference delay** — corresponding to >30% of
  prediction horizon.

## Methods

- **Action chunking policy** π(A_t | o_t), prediction horizon `H`,
  execution horizon `s`. Conditional flow matching at training.
- **Definitions:** Δt = controller period; δ = inference time;
  `d := ⌊δ/Δt⌋` = inference delay in controller timesteps.
- **Asynchronous inference loop** (Algorithm 1):
  - `GetAction` consumes `a_{t−1}`, provides `o_t` (every Δt).
  - `InferenceLoop` runs in background; forecasts next `d` from a
    rolling buffer of past inference times.
  - Frozen prefix: `a_{0:d}` from previous chunk get weight 1.
  - Intermediate: `a_{d:H−s}` get exponentially decaying weight.
  - Free: `a_{H−s:H}` get weight 0 (purely generated).
- **ΠGDM velocity correction** (Eq 2-4):
  `v_ΠGDM = v + min(β, ((1−τ)/(τ·r_τ²))) · ((Y − Â₁)^⊤ diag(W) ∂Â₁/∂A_τ)`
  where `Â₁` is the one-step denoised estimate, `W` is the soft mask,
  `Y` is the previous-chunk target.
- **Execution horizon constraint:** `d ≤ s ≤ H − d` for feasibility.

## Results (headline)

- **Kinetix sim (12 dynamic tasks):** RTC and BID (best baseline) are
  the only methods that benefit from shorter execution horizons; RTC
  outperforms all baselines under varying inference delay
  `d ∈ {0, ..., 4}` with `H = 8`.
- **Real-world bimanual (6 tasks, π0.5 base):** RTC achieves significantly
  improved task throughput on all 6 tasks; high success rate on
  match-lighting at 300ms+ latency (paper highlights this as their
  flagship demonstration).
- **Inference delay tolerance:** robust to delays >30% of prediction
  horizon.

## Limitations / open questions

- **Inference-time inpainting adds compute.** ΠGDM guidance involves a
  vector-Jacobian product per denoising step, so peak GPU compute per
  step rises. Net throughput still wins because of asynchrony, but the
  per-step latency increase isn't free.
- **Requires a generative policy** (flow / diffusion). Won't apply
  out-of-box to non-generative policies (e.g. transformer-decoder VLAs).
- **Soft-mask decay schedule** is hand-designed; the
  `H − s − i` exponential form is one of several reasonable choices.
- **Hyperparameter `β` clipping** is non-trivial; without it the
  algorithm is unstable at typical denoising-step budgets (3–5 steps
  for control).
- **Sim benchmark is new** (Kinetix); apples-to-apples comparison
  against external numbers is limited. Real-world benchmark is custom
  to the authors' robot setup.

## Connections

- **Method:** [[rtc]].
- **Concept introduced / champion:** [[action-chunking]] (the chunked
  execution paradigm goes back to ACT/Zhao 2023; RTC champions
  inference-time inpainting as the right way to handle latency).
- **Concept: [[asynchronous-control]]** — RTC formalizes the
  asynchronous-inference regime more cleanly than prior work.
- **Concept: [[train-inference-mismatch]]** — RTC is **the closest
  analogue in robotics** to the train-inference mismatch we noted in
  [[spatialtracker-v2]] (clean GT queries during training vs.
  noisy synthesized queries from previous-window predictions at
  inference). RTC handles it explicitly via inpainting + soft masking;
  STV2 does not.
- **Base model:** π0.5 VLA (Black et al. 2024) — not yet ingested.
  Belongs to the broader VLA family — see [[vla]] (new concept stub).
- **Training paradigm: [[flow-matching]]** — conditional flow matching;
  converted from diffusion at inference time. New concept stub.
- **Cited baselines:** ACT/temporal ensembling (Zhao et al. 2023, ref
  [68]), BID (best alternative in their sim), DP/OpenVLA (refs
  [22, 30]).
- **Authors:** [[kevin-black]] (lead), Manuel Y. Galliker, [[sergey-levine]]
  (senior; UC Berkeley / Physical Intelligence).
- **Orgs:** [[physical-intelligence]] (new), [[uc-berkeley]] (already
  in wiki via CUT3R).

## Citation

Black, K., Galliker, M. Y., & Levine, S. (2025). *Real-Time Execution
of Action Chunking Flow Policies.* NeurIPS 2025. arXiv:2506.07339v2.
https://arxiv.org/abs/2506.07339
