---
type: method
title: CUT3R (Continuous Updating Transformer for 3D Reconstruction)
status: stub
tags: [3d-reconstruction, online, recurrent, transformer, pointmap, virtual-view-query, slam]
sources:
  - "[[wang-2025-cut3r]]"
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
created: 2026-05-24
updated: 2026-07-02
---

# CUT3R (Continuous Updating Transformer for 3D Reconstruction)

**Stateful recurrent** online 3D reconstruction. Each new frame updates
a persistent latent state; the state can be **queried with virtual
unseen views** to infer 3D structure for unobserved regions. The
streaming counterpart to VGGT-family batch reconstruction.

## One-line summary

ViT encodes each new frame → two interconnected transformer decoders
write to & read from a persistent state token bank → DPT heads emit
per-frame pointmap (self + world frame) + ego-motion → state can also
be queried with a virtual ray map.

## Inputs / outputs

- **In:** image stream (video or unordered photos), no camera info.
- **Out per frame:**
  - `(X̂^self_t, C^self_t)` — pointmap + confidence in current-camera
    coords.
  - `(X̂^world_t, C^world_t)` — pointmap + confidence in world (frame-1)
    coords.
  - `P̂_t` — relative ego motion.
- **Virtual view query:** input a ray map for any virtual viewpoint →
  state-readout produces pointmap + color for that view (even unseen).

## How it works

1. **Encoder:** ViT per-frame; emits image tokens `F_t`.
2. **State:** learned tokens, initialized as scene-agnostic priors;
   evolves over time.
3. **State-input interaction:** two interconnected transformer decoders.
   Image tokens + pose token `z` *update* the state and *read* from it;
   the decoders cross-attend per block.
4. **Heads (per frame):**
   - `Head_self` = DPT → self-frame pointmap.
   - `Head_world` = DPT → world-frame pointmap.
   - `Head_pose` = MLP on enriched pose token → ego motion.
5. **Virtual view:** for unseen view query, ray map → state-readout →
   pointmap. No additional training, just inference-time query.

## Where it's been applied

- Monocular depth, consistent video depth, camera pose, 3D reconstruction.
- Dynamic + static scenes.
- Online inference from streaming video; also unstructured photo
  collections.

## Known limitations

- Recurrence trades peak quality vs batch methods (VGGT, MapAnything) for
  streaming.
- Virtual-view inference quality degrades far from observed regions.
- Heavy training mix (single images + videos + photo collections + 3D
  annotations).
- No head-to-head with later VGGT-family models in the paper.

## Related methods

- **Closest concurrent / contemporary:** Spann3R (cache-style memory
  vs CUT3R's generative state), [[point3r]] (explicit 3D-indexed
  pointer memory — third distinct memory paradigm; beats CUT3R on
  long-sequence 3D reconstruction).
- **Batch counterparts:** [[vggt]], [[mapanything]],
  [[depth-anything-3]] — CUT3R is the online answer to all of these.
- **Streaming competitor (newer):** [[streamvggt]] — uses a growing
  KV cache + causal attention distilled from VGGT, rather than CUT3R's
  fixed-size token bank. **Beats CUT3R** on 3D reconstruction (7-Scenes,
  NRGBD, ETH3D), camera pose (ScanNet), and depth estimation across
  benchmarks. The trade-off: StreamVGGT's memory grows linearly with N
  while CUT3R's is constant.
- **Online dynamic SLAM contrast:** MonST3R, MegaSaM, CasualSAM —
  all need per-scene optimization. CUT3R is online + feed-forward.
- **CUT3R features as foundation** for downstream tasks is an open
  direction (not deeply explored in the paper).
