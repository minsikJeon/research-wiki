---
type: concept
title: Feed-Forward 3D Reconstruction
status: growing
tags: [3d-reconstruction, feed-forward, transformer, foundation-model]
sources:
  - "[[wang-2025-vggt]]"
  - "[[keetha-2025-mapanything]]"
  - "[[lin-2025-depth-anything-3]]"
  - "[[wang-2025-cut3r]]"
  - "[[karhade-2025-any4d]]"
  - "[[sucar-2026-v-dpm]]"
  - "[[luo-2026-4rc]]"
related:
  - "[[pointmap-representation]]"
  - "[[4d-reconstruction]]"
  - "[[vggt]]"
  - "[[mapanything]]"
  - "[[depth-anything-3]]"
created: 2026-05-24
updated: 2026-05-24
---

# Feed-Forward 3D Reconstruction

## Definition

Recover the 3D structure of a scene (geometry + cameras) from one or
more images in a **single neural-network forward pass**, with no
test-time optimization. Replaces the classical SfM/MVS pipeline
(feature matching → triangulation → bundle adjustment) with a learned
end-to-end model.

## Why it matters

- **Speed:** seconds vs minutes-to-hours for classical SfM.
- **Robustness:** works in regimes where classical SfM fails (sparse
  views, weak textures, ambiguous baselines).
- **Generality:** one model handles many tasks (uncalibrated SfM,
  calibrated MVS, monocular depth, ...).
- **Foundation-model substrate:** feed-forward 3D models are the
  backbone for 4D extensions ([[4d-reconstruction]]) and downstream
  perception tasks (tracking, novel-view synthesis, 3D generation).

## Lineage (as of May 2026, in this wiki)

```
DUSt3R / MASt3R (pairwise + post-hoc alignment)     ← [[dust3r]]
        ↓
   VGGT (multi-view, alternating attn, all-in-one)  ← [[vggt]]
        ├── MapAnything (factored + metric + multi-modal)  ← [[mapanything]]
        ├── V-DPM (fine-tuned to dynamic / 4D)             ← [[v-dpm]]
        ├── 4RC (encode-once + conditional query)          ← [[4rc]]
        └── DA3 (strips to depth+ray only)                 ← [[depth-anything-3]]
              ↑ +35.7% pose / +23.6% geo vs VGGT
        ├── CUT3R (recurrent online variant)               ← [[cut3r]]
        └── Any4D (4D sibling of MapAnything)              ← [[any4d]]
```

## Core principles emerging from the batch

1. **No 3D inductive biases needed** ([[vggt]]). Vanilla transformers
   + alternating attention + lots of data > clever architecture.
2. **Factored representations beat coupled.** Decomposing into
   depth + rays + pose + metric scale ([[mapanything]], [[any4d]]) trains
   better on partial-annotation data than coupled pointmaps.
3. **Joint over-complete prediction helps** ([[vggt]]): predicting
   depth + camera + pointmap + tracks together is better than any subset,
   even when one quantity derives from another.
4. **But less can be more** ([[depth-anything-3]]): just depth + ray maps
   beats VGGT's multi-task head zoo.
5. **Post-hoc optimization can be eliminated.** Pre-VGGT methods
   (DUSt3R) required global alignment; VGGT-family don't.
6. **Foundation models in 3D follow the same recipe as in NLP / 2D
   vision:** large transformer + lots of varied data + strong
   pretraining (DINO, etc.) → general capability.

## Evidence across sources

- [[wang-2025-vggt]]: VGGT > DUSt3R/MASt3R/VGGSfM across all metrics;
  features generalize to non-rigid tracking + NVS.
- [[keetha-2025-mapanything]]: one model > 12 specialist feed-forward
  models, with metric scale and multi-modal inputs.
- [[lin-2025-depth-anything-3]]: minimalist DA3 > VGGT by 35.7% pose /
  23.6% geo. Settles the "minimum sufficient prediction target" question.
- [[wang-2025-cut3r]]: online recurrence works for streaming 3D + virtual
  view inference.
- [[karhade-2025-any4d]] + [[sucar-2026-v-dpm]] + [[luo-2026-4rc]]:
  extend the paradigm to dynamic 4D.

## Open questions

- **Streaming + dynamic:** [[cut3r]] handles streaming, [[any4d]] handles
  dynamic. Combined streaming + 4D is open.
- **Scale ceiling:** how big do these models need to grow before they
  saturate? Comparable to LLM scaling laws?
- **Cross-task transfer:** can a single foundation model serve 3D + 4D +
  tracking + segmentation + generation? Or do specialized backbones win?
- **Robotics deployment:** most evals are on academic benchmarks. Open:
  how do these models perform on diverse real robot streams?
- See [[q-tracking-vs-4d-reconstruction]] for the longer-form open thread.
  (A dedicated `q-vggt-paradigm` question page is a candidate to create.)
