---
type: question
title: When does RTC remain preferable to Training-Time RTC?
status: open
tags: [robotics, vla, real-time-control, action-chunking, inference-time-algorithm, training-recipe]
sources:
  - "[[black-2025-rtc]]"
  - "[[black-2025-training-time-rtc]]"
related:
  - "[[rtc]]"
  - "[[training-time-rtc]]"
  - "[[pi-r-squared]]"
  - "[[pi-gdm]]"
  - "[[asynchronous-control]]"
  - "[[train-inference-mismatch]]"
created: 2026-06-10
updated: 2026-06-10
---

# When does RTC remain preferable to Training-Time RTC?

## The question

[[training-time-rtc]] eliminates [[rtc]]'s per-step ΠGDM vector-Jacobian
product, reducing per-call latency 135 ms → 108 ms on π0.6 with **same
real-world success rates**. The TT-RTC paper claims it should be a
**drop-in replacement** when fine-tuning is available.

But ΠGDM-based [[rtc]] retains several deployment advantages:

1. **No retraining required** — drop-in to any pretrained flow policy.
2. **Arbitrary observation operators** — supports soft prefix +
   intermediate-region exponential decay + arbitrary linear
   constraints; TT-RTC supports only a hard prefix of length `d`.
3. **β is a per-task knob** — tunable at deployment without retraining;
   TT-RTC's `d_max` is set at training time.

**Question:** is there a deployment regime where RTC remains
strictly preferable?

## Candidate regimes where RTC may still win

| Regime | Why RTC wins |
|---|---|
| Untrainable / frozen checkpoint | No fine-tune budget; ΠGDM is the only option |
| Multi-tenant inference | Many checkpoints share infra; per-policy fine-tune impractical |
| Tasks with varying constraint structure | TT-RTC's hard prefix is too rigid; ΠGDM's β/W flexibility wins |
| Very small `d` (`d=0,1`) | TT-RTC's exponential `d` sampling under-trains low-`d` — fig 3 shows small RTC win there |
| Per-deployment latency varies | Hard to retrain TT-RTC for every robot's measured `d` |

## What would settle it

- **An ablation paper** measuring the RTC → TT-RTC gap *under
  redeployment* — i.e., when the deployed-`d` differs from training-`d`
  by more than the exponential schedule allows.
- **A theory paper** characterizing the linearization-error vs
  exponential-`d`-bias trade-off as a function of `d` and chunk size
  `H`.
- **πR²'s extension** ([[anon-2026-pi-r-squared]]) sidesteps this by
  training a continuous family — but at higher fine-tune cost. The
  three-way comparison RTC vs TT-RTC vs πR² along the
  fine-tune-cost ↔ deployment-flexibility frontier is open.

## Status

Open. Practical answer for product VLAs (single policy, fixed robot)
seems to be **TT-RTC is strictly better**; for prototyping and
multi-policy infra **RTC is still useful**.
