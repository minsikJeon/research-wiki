---
type: entity
entity_type: person
name: Shubham Tulsiani
status: growing
tags: [computer-vision, 3d-reconstruction, scene-flow, visual-geometry-learning, manipulation, point-tracking]
sources:
  - "[[anon-2026-point4d]]"
  - "[[bharadhwaj-2024-track2act]]"
  - "[[kuang-2026-dex4d]]"
related:
  - "[[cmu-ri]]"
  - "[[flow3r]]"
  - "[[point4d]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[track2act]]"
  - "[[dex4d]]"
  - "[[point-tracks-as-manipulation-interface]]"
  - "[[homanga-bharadhwaj]]"
  - "[[sungjae-park]]"
  - "[[yuxuan-kuang]]"
  - "[[katerina-fragkiadaki]]"
created: 2026-05-26
updated: 2026-06-27
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
  formal frontmatter to mirror the venue review process.
- [[bharadhwaj-2024-track2act]] — **Track2Act** (ECCV 2024). Senior
  author; first author Homanga Bharadhwaj. Founding paper of the
  "predict point tracks from web videos → drive robot manipulation"
  line, ancestor to Im2Flow2Act / 3DFlowAction / Dex4D / Pri4R.
- [[kuang-2026-dex4d]] — **Dex4D** (Feb 2026). Co-senior author with
  [[katerina-fragkiadaki]]; first authors Yuxuan Kuang + Sungjae Park.
  Direct dexterous-manipulation descendant of Track2Act, also
  citing Tulsiani-group DemoDiffusion. **Acknowledges the user
  (Minsik Jeon) for presentation feedback.**

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
high-priority ingest. The wiki now has **three primary Tulsiani-group
sources** spanning two research lines:
- *Geometry / 4D:* [[anon-2026-point4d]], [[flow3r]] (placeholder).
- *Manipulation / point-track-conditioned policies:*
  [[bharadhwaj-2024-track2act]], [[kuang-2026-dex4d]] —
  [[katerina-fragkiadaki]] co-senior on Dex4D, making this a joint
  Tulsiani × Fragkiadaki line.

Other CMU-RI faculty already in the wiki ([[deva-ramanan]],
[[sebastian-scherer]], [[laszlo-jeni]]) are likely collaborators or
seminar peers. Watch for joint papers.
