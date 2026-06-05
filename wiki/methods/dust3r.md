---
type: method
title: DUSt3R / MASt3R (pointmap lineage)
status: stub
tags: [3d-reconstruction, feed-forward, transformer, pointmap, pairwise]
sources: []
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[vggt]]"
created: 2026-05-24
updated: 2026-05-24
---

# DUSt3R / MASt3R (pointmap lineage)

The pairwise feed-forward 3D reconstruction line that **introduced
pointmaps** as the unified representation for shape + camera. Cited by
every paper in the 3D/4D reconstruction batch as the conceptual
predecessor. **Not yet ingested** as a primary source — this entry seeds
the entity from secondary mentions.

## One-line summary

Given an image pair, directly predict per-pixel 3D pointmaps in a shared
coordinate frame — bypassing classical SfM's matching → triangulation →
BA pipeline. MASt3R is the metric-scale follow-up.

## Inputs / outputs

- **In:** 2 RGB images.
- **Out:** per-pixel pointmaps `P_0, P_1 ∈ R^{3×H×W}` in the coordinate
  frame of image 0; implicitly encodes camera intrinsics + extrinsics +
  shape.

## How it works (sketch)

- Twin ViT encoder + cross-attention decoder.
- Predict pointmaps in a shared frame; this **inherently** encodes
  cameras (the camera-1 frame is identity; camera-2 pose is recoverable
  from the two pointmaps).
- For >2 images: **pairwise inference + post-hoc global alignment**
  (optimization).

## Where it's been applied

Originally on 3D reconstruction from photo collections. The pointmap
representation has since been picked up by:
- [[vggt]] — multi-view extension dropping pairwise + alignment.
- [[mapanything]] — adds metric scale + multi-modal + factored repr.
- [[v-dpm]] — dynamic point maps.
- [[cut3r]] — online recurrent pointmap prediction.
- [[4rc]] — base geometry as pointmap + relative motion.
- [[trace-anything]] — explicitly contrasts pointmap-per-frame with
  trajectory fields.

## Known limitations

- **Pairwise only** → needs O(N²) inference + global alignment for
  multi-view.
- **Static scenes only** in the original.
- **Up-to-scale** (DUSt3R); MASt3R is the metric follow-up.

## Related methods

- **Pointmap descendants:** see "where it's been applied" above.
- **Pairwise post-hoc-alignment alternatives:** MonST3R (dynamic),
  POMATO, Spann3R (continuous), CUT3R, VGGT (multi-view in one pass).

## Notes

This entry exists because DUSt3R / MASt3R are referenced by 6+ of this
wiki's sources but not yet ingested as primary papers. Promote to a
full ingest when the user adds the DUSt3R / MASt3R PDFs to
`raw/papers/`.
