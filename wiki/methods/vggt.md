---
type: method
title: VGGT (Visual Geometry Grounded Transformer)
status: growing
tags: [3d-reconstruction, feed-forward, transformer, pointmap, depth-estimation, camera-pose, foundation-model]
sources:
  - "[[wang-2025-vggt]]"
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[v-dpm]]"
  - "[[mapanything]]"
  - "[[depth-anything-3]]"
  - "[[4rc]]"
created: 2026-05-24
updated: 2026-05-24
---

# VGGT (Visual Geometry Grounded Transformer)

**Foundational** feed-forward transformer for static 3D reconstruction.
Takes N RGB images, outputs camera intrinsics + extrinsics + depth maps +
point maps + 2D tracking features in one second. The ancestor architecture
for the 2025-2026 wave of 3D/4D reconstruction foundation models.

## One-line summary

DINO ViT patchify → L blocks of alternating (frame-wise / global) self-
attention → DPT heads for depth + point map + tracking features + camera
head; no test-time optimization needed.

## Inputs / outputs

- **In:** 1 to hundreds of RGB images of the same scene.
- **Out (per frame):** camera `g_i = (q, t, f)` (quaternion + translation
  + focal); depth `D_i`; point map `P_i` (in world frame = first
  camera); tracking features `T_i` ∈ R^{C×H×W}.
- **Plus:** 2D tracks for query points via a separate tracking head.

## How it works

1. **Patchify:** DINO ViT splits each image into tokens.
2. **Camera token:** prepended per frame; outputs camera params.
3. **Backbone:** L blocks alternating
   - **Frame-wise self-attention** (per-image, like ViT).
   - **Global self-attention** (across all frames, all tokens).
   - First frame is special — used as world-coord reference; the
     architecture is permutation-equivariant for frames 2..N.
4. **Heads:**
   - Camera head (MLP) → `g_i`.
   - DPT depth head → `D_i`.
   - DPT pointmap head → `P_i` in world frame.
   - DPT tracking-features head → `T_i`.
5. **Tracking module** (jointly trained): query points + tracking
   features → 2D tracks across frames.
6. **Over-complete by design:** depth + camera + point map are redundant
   (point map derivable from depth + camera). Both heads improve
   accuracy.

## Where it's been applied

- Camera pose estimation, multi-view depth, dense point cloud
  reconstruction, 3D point tracking (all SOTA at publication).
- **As backbone / frozen features:** for downstream point tracking and
  novel-view synthesis tasks.

## Known limitations

- **Static scenes only.** Dynamic content out of distribution.
  Addressed by [[v-dpm]] (DPM heads), Pi3 (dynamic depth only), and
  most of this batch.
- **First-frame reference asymmetry** — fixed; π³ (Pi3) removes this.
- **Pointmap head is over-complete** and slightly worse than
  depth+camera derivation at inference; kept for training-signal value.

## Related methods

- **Predecessors:** DUSt3R, MASt3R (pairwise pointmaps + post-hoc
  alignment), VGGSfM (differentiable BA). VGGT is the multi-view
  extension that drops post-hoc opt.
- **Direct extensions in this wiki:**
  - [[v-dpm]] — adds Dynamic Point Map heads for 4D.
  - [[4rc]] — VGGT encoder + conditional decoder for query-anywhere 4D.
  - [[mapanything]] — VGGT-style multi-view + factored repr + metric
    scale + multi-modal inputs.
- **Competitors:**
  - [[depth-anything-3]] strips to depth+ray; +35.7% pose / +23.6% geo
    over VGGT.
  - [[mapanything]] adds metric scale + flexibility.
- **Used by / as backbone:** [[spatialtracker-v2]] (alternating
  attention pattern).
