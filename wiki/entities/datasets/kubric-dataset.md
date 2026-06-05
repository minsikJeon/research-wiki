---
type: entity
entity_type: dataset
name: Kubric
status: stub
tags: [point-tracking, synthetic-data, training-data]
sources:
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[zholus-2025-tapnext]]"
  - "[[jung-2026-tapnext-plus-plus]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[xiao-2025-spatialtracker-v2]]"
related:
  - "[[point-tracking]]"
  - "[[tap-vid-dataset]]"
  - "[[synthetic-to-real-gap]]"
created: 2026-05-24
updated: 2026-05-24
---

# Kubric

## Composition

Open-source synthetic video data generator from Google (Greff et al.,
CVPR 2022). Produces 3D-rendered videos with **automatic ground-truth
point tracks**, segmentation, depth, optical flow, normals — anything
the renderer knows. Used by basically every modern [[point-tracking]]
method as the primary training source because real point-track
annotation is prohibitively expensive.

## Variants observed in this wiki

- **TAP-Vid-Kubric** (vanilla): 11K videos × 24 frames. The default
  training set for the [[tap-vid-dataset]] line.
- **TAPNext's extension** ([[zholus-2025-tapnext]]): 500K videos × 48
  frames, with camera panning + motion blur added.
- **Kubric-1024** ([[jung-2026-tapnext-plus-plus]]): 1024-frame extension
  with diverse trajectory generation (spherical-shell start/end +
  sinusoidal noise + ground-plane projection).

## Why it matters

The training-data foundation of the whole sub-field. Discussions of
[[synthetic-to-real-gap]] all reduce to "how well does
Kubric-trained representation transfer to real video?" — that's the
question.

## Papers using it (in this wiki)

Effectively all 7 sources ingested so far. Listed in `sources:`
frontmatter.

## Known biases / caveats

- Rendering aesthetic — simple shapes, uniform lighting, limited textures
  vs. real-world distribution.
- Trajectory generation is procedural; statistics differ from human-
  recorded scenes.
- 2D-only by default (3D versions like Kubric-1024 still rely on
  rendering-time GT depth).
