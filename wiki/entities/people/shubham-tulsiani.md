---
type: entity
entity_type: person
name: Shubham Tulsiani
status: stub
tags: [computer-vision, 3d-reconstruction, scene-flow, visual-geometry-learning]
sources:
  - "[[anon-2026-point4d]]"
related:
  - "[[cmu-ri]]"
  - "[[flow3r]]"
  - "[[point4d]]"
  - "[[feed-forward-3d-reconstruction]]"
created: 2026-05-26
updated: 2026-05-29
---

# Shubham Tulsiani

## Affiliation

[[cmu-ri]] — Assistant Professor at Carnegie Mellon's Robotics Institute.
**The user's advisor.**

## Research focus relevant to this wiki

Visual geometry learning, 3D reconstruction, scene flow, single- and
multi-view 3D inference. The Tulsiani group sits squarely in the
[[feed-forward-3d-reconstruction]] lineage covered by this wiki (DUSt3R
→ VGGT → DA3 → 4D extensions), with particular emphasis on **factored
representations** and **scalable** learning targets.

## Sources in this wiki

- [[anon-2026-point4d]] — **Point4D** (NeurIPS 2026 under double-blind
  review). The user (Minsik Jeon) is **lead author** under Tulsiani's
  advising; the source page treats authorship as anonymous in its
  formal frontmatter to mirror the venue review process. This is the
  first primary Tulsiani-group source in the wiki.

## Notes

- **Point4D** ([[point4d]]) — long-range 4D motion reconstruction via
  3D-coordinate queries + chunk chaining. Lead author: the user.
  Encoder initialized from [[depth-anything-3]]; thesis: 3D queries
  decouple point identity from image-plane visibility, enabling
  direct re-querying across chunks under occlusion.
- **Flow3R** ([[flow3r]]) — *Factored flow prediction for scalable
  visual geometry learning*. Cong, Zhao, Jeon, Tulsiani,
  arXiv:2602.20157, 2026. Co-authored with the user. Cited by
  [[anon-2026-point4d]] reference [3] as a peer in the "feed-forward
  visual geometry prediction" lineage (VGGT / DUSt3R / MASt3R / DA3
  family).
- **Group research line (inferred from Flow3R + Point4D):** factored,
  scalable representations for feed-forward visual geometry + scene
  motion. Both papers commit to *feed-forward* (vs. per-scene
  optimization) and to representations that are *factored* across
  geometry / motion / camera. Likely thread for future Tulsiani-group
  ingests.

When a Tulsiani-group paper arrives in `raw/papers/`, this should be a
high-priority ingest. Other CMU-RI faculty already in the wiki
([[deva-ramanan]], [[sebastian-scherer]], [[katerina-fragkiadaki]])
are likely collaborators or seminar peers. Watch for joint papers.
