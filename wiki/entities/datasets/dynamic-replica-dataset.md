---
type: entity
entity_type: dataset
name: Dynamic Replica
status: stub
tags: [point-tracking, 3d-point-tracking, benchmark, synthetic-data]
sources:
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[zhang-2025-tapip3d]]"
related:
  - "[[point-tracking]]"
  - "[[3d-point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# Dynamic Replica

## Composition

Synthetic dataset of dynamic indoor scenes built on the Replica scene
collection. Introduced by Karaev et al., CVPR 2023. Provides ground-truth
3D point tracks alongside RGB-D — used both as a 2D point tracking
benchmark and as a 3D tracking eval.

## Why it matters

One of the few datasets that bridges [[point-tracking]] and
[[3d-point-tracking]]: 2D-only methods use it as a non-Kubric
benchmark; 3D methods use it as a depth-aware eval. Heavy on occlusions
since indoor scenes have lots of furniture.

## Papers using it (in this wiki)

- [[karaev-2024-cotracker]] — major eval; cross-track attention's biggest
  occluded-point gains were measured here.
- [[karaev-2024-cotracker3]] — eval.
- [[aydemir-2025-track-on2]] — eval (Table 2).
- [[zhang-2025-tapip3d]] — 3D eval.

## Known biases / caveats

- Indoor-scene specific; outdoor / dynamic-camera generalization not
  measured here.
- Synthetic.

## Notes

The Karaev paper that introduced this dataset is also the trunk paper
for the [[nikita-karaev]] line of work. Co-introducing a benchmark with
your method is a recurring pattern in this sub-field — be alert to the
incentive bias.
