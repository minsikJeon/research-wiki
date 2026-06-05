---
type: question
title: Is real-data fine-tuning necessary for SOTA point tracking?
status: open
tags: [point-tracking, sim2real, pseudo-labeling, synthetic-data]
sources:
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
  - "[[zholus-2025-tapnext]]"
related:
  - "[[point-tracking]]"
  - "[[synthetic-to-real-gap]]"
  - "[[pseudo-labeling-point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# Is real-data fine-tuning necessary for SOTA point tracking?

## The question

Two camps have emerged as of 2025-2026:

- **Camp A — fine-tune on real with pseudo-labels:** BootsTAPIR
  (15M videos, heavy recipe), [[cotracker3]] (15K videos, simple recipe).
- **Camp B — synthetic-only is enough if training is right:**
  [[aydemir-2025-track-on2]] (memory + classification + long synthetic
  clips), [[jung-2026-tapnext-plus-plus]] (1024-frame Kubric training of
  unchanged TAPNext architecture).

Both camps report SOTA results on overlapping benchmarks. **Which is
right?**

## Why it matters

Practical implications:

- If **Camp B is right:** real-data collection / labeling pipelines for
  point tracking are unnecessary. Smaller labs can match Big Tech.
- If **Camp A is right:** the long-term winners will be groups with
  unlabeled video data at scale (Meta, Google, ByteDance).
- If **both are right in different regimes:** we need a principled way
  to decide which to use per deployment.

For a robotics application (the user's likely deployment context):
real data is expensive to label but unlabeled robot-perspective video
is plentiful. The right strategy depends on which camp applies to
robot-vs-Internet visual distributions.

## What we know

- **Camp A's strongest evidence:** [[karaev-2024-cotracker3]] beats
  BootsTAPIR with 1000× less data using teacher diversity (Table 1, 2 of
  source). Simple recipes can extract surprising mileage from real data.
- **Camp B's strongest evidence:** [[jung-2026-tapnext-plus-plus]] beats
  BootsTAPNext (real-data fine-tuned variant of [[tapnext]]) on long
  sequences with **only** Kubric-1024 + roll augmentation +
  occluded-point supervision. The architecture is identical to TAPNext —
  so the gain is purely from synthetic training improvements.
- **Both camps share a common predecessor finding:** [[zholus-2025-tapnext]]
  showed synthetic-only TAPNext was already SOTA on TAP-Vid; the real-data
  fine-tuned BootsTAPNext only modestly improved over that baseline.

## What we don't

- **No matched-compute comparison.** "1000× less data" claims don't
  account for the differing model sizes, training steps, or backbone
  pretraining.
- **No deployment-distribution analysis.** Most evals are on TAP-Vid
  (Internet videos) or PointOdyssey (synthetic). Robotics / egocentric /
  medical regimes are underrepresented.
- **No additive study.** Does combining "long synthetic training" with
  "lightweight real-data pseudo-labels" stack? Or do they compete for
  the same model capacity?

## Tentative position

The honest read: **Camp B is sufficient for current benchmarks; Camp A
remains useful for distribution-shifted deployments.**

For the user's context (CMU MSR, likely robotics-adjacent):

- Start with the best **Camp B** recipe (TAPNext++ training pipeline) —
  it's the simplest, requires no real-data collection, and currently
  leads on long-sequence robustness.
- Only add pseudo-labeled real data if a specific deployment
  distribution shows residual gap.

## Next things to read / ingest

- **BootsTAP / BootsTAPIR** (Doersch et al., 2024) — the canonical
  pseudo-labeling pipeline; cited by every paper in the batch but not
  yet ingested. Should be next priority.
- **PIPs / PIPs++** (Harley et al.) — predecessors that established the
  synthetic-training default.
- **Any robotics-specific point-tracking paper** to see how the two
  camps perform under non-Internet distributions.
- A future systematic **matched-compute comparison** paper would
  resolve much of this — but doesn't exist yet, would be a thesis-scale
  project.
