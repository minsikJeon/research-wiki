---
type: entity
entity_type: person
name: Junyi Zhang
status: growing
tags: [computer-vision, 3d-reconstruction, dynamic-scenes, long-context]
sources:
  - "[[zhang-2026-loger]]"
related:
  - "[[google-deepmind]]"
  - "[[uc-berkeley]]"
  - "[[loger]]"
  - "[[trevor-darrell]]"
created: 2026-05-29
updated: 2026-05-29
---

# Junyi Zhang

## Affiliation

UC Berkeley PhD student (Trevor Darrell group) with Google DeepMind
collaboration / internship pattern.

## Research focus

Long-context feedforward 3D reconstruction; dynamic-scene 3D from
video. Works at the intersection of (a) pretrained visual geometry
models (DUSt3R / VGGT / π³ family) and (b) efficient long-sequence
architectures (memory mechanisms, fast-weight modules).

## Sources in this wiki

- **[[zhang-2026-loger]] (lead author)** — LoGeR: Long-Context
  Geometric Reconstruction with Hybrid Memory. NeurIPS-track 2026.
  Chunked feedforward 3D with SWA + TTT hybrid memory, scaling to
  19k-frame videos.

## Other notable work (not yet primary in wiki)

- **MonST3R** [Zhang et al. 2025a, "A Simple Approach for Estimating
  Geometry in the Presence of Motion"] — cited extensively in the
  3D / 4D reconstruction batch (Point4D, LoGeR, V-DPM, Any4D, etc.).
  Should be promoted to a primary source on next reference (already
  cited 5+ times in this wiki).

## Notes (cross-source signal)

Junyi Zhang appears in multiple wiki sources as a cited author —
MonST3R is referenced by virtually every recent 4D reconstruction
paper, and LoGeR is the architectural ancestor for the "hybrid
memory" pattern that the user's planning note (Topic 3) is converging
on. **Worth tracking actively** as a peer / competitor on long-context
3D reconstruction.

Notably, J. Zhang's MonST3R/LoGeR trajectory ("DUSt3R for motion" →
"DUSt3R for long context") parallels the trajectory toward what the
user's project is targeting (3D point tracking for long context, real-
time). The two trajectories are not the same problem but share
ancestors and design vocabulary.
