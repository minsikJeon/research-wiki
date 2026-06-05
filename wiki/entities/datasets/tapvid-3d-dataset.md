---
type: entity
entity_type: dataset
name: TAPVid-3D
status: stub
tags: [point-tracking, 3d-point-tracking, benchmark]
sources:
  - "[[zhang-2025-tapip3d]]"
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[3d-point-tracking]]"
  - "[[tap-vid-dataset]]"
created: 2026-05-24
updated: 2026-05-24
---

# TAPVid-3D

## Composition

The 3D counterpart to [[tap-vid-dataset]]. Provides per-frame 3D point
ground truth across three subsets:

- **Aria** — Project Aria egocentric video.
- **DriveTrack** — automotive / driving scenes.
- **PStudio** — Panoptic Studio (CMU's volumetric capture).

The defining benchmark for [[3d-point-tracking]] as of 2025.

## Metrics

- **AJ** — 3D Average Jaccard.
- **APD3D** — Average 3D Position Accuracy (fraction of estimates within
  multiple distance thresholds).
- **OA** — Occlusion Accuracy.

## Papers using it (in this wiki)

- [[zhang-2025-tapip3d]] — primary eval. Reports AJ across all three
  subsets.
- [[xiao-2025-spatialtracker-v2]] — primary eval. 21.2 AJ / 31.0 APD3D,
  +61.8% / +50.5% over DELTA.

## Known biases / caveats

- Three subsets have very different scene characteristics; aggregate
  numbers can be misleading.
- Depth quality at eval time matters: GT depth vs estimated depth
  (UnidepthV2, MegaSaM) materially shifts results — papers report
  conditions inconsistently.
- Aria subset is egocentric → camera ego-motion dominant. Not
  representative of static-camera workflows.

## Notes

Introduced by [[skanda-koppula]] et al. 2024 (Google DeepMind). The
[[tap-vid-dataset]] page references this as its 3D successor; head-to-head
comparisons of [[tapip3d]] vs [[spatialtracker-v2]] here are an open
question — see [[cmp-tap-methods]].

**As of [[zhang-2025-d4rt]] ingest (2026-05-26),** D4RT now holds SOTA
on TAPVid-3D's 3D tracking task (Table 4 of source): AJ 0.304 / APD3D
0.410 on DriveTrack (camera coords, GT intrinsics) vs SpatialTrackerV2
0.195 / 0.275. Beats all prior 3D trackers and 2D-tracker+depth
baselines.
