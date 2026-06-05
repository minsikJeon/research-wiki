---
type: source
source_type: paper
title: "Continuous 3D Perception Model with Persistent State (CUT3R)"
authors:
  - Wang, Qianqian
  - Zhang, Yifei
  - Holynski, Aleksander
  - Efros, Alexei A.
  - Kanazawa, Angjoo
year: 2025
venue: arXiv (cs.CV) — CVPR 2025 candidate
url: https://arxiv.org/abs/2501.12387
raw_path: papers/Wang et al. - 2025 - Continuous 3D Perception Model with Persistent State.pdf
status: ingested
tags: [3d-reconstruction, online, recurrent, transformer, pointmap, foundation-model, slam]
sources: []
related:
  - "[[cut3r]]"
  - "[[feed-forward-3d-reconstruction]]"
  - "[[pointmap-representation]]"
  - "[[qianqian-wang]]"
  - "[[uc-berkeley]]"
  - "[[google-deepmind]]"
created: 2026-05-24
updated: 2026-05-24
---

# Continuous 3D Perception Model with Persistent State (CUT3R)

## TL;DR

CUT3R is a **stateful recurrent transformer** that processes image
streams **online** — each new frame updates a persistent latent state
and the model emits a metric pointmap + camera pose for the new view.
Handles static + dynamic scenes, video streams + sparse photo collections.
Crucially, the state can be **queried with virtual unseen views** to
infer 3D structure for regions the model never observed — a form of
learned 3D prior. UC Berkeley × Google DeepMind.

## Why it matters

The **online / streaming** counterpart to VGGT-family batch reconstruction.
Where [[vggt]], [[mapanything]], [[depth-anything-3]] process all N
frames in one pass, CUT3R updates incrementally as frames arrive —
fundamentally important for robotics, AR, embodied perception. Closest
classical analogue: **learned SLAM**.

The "virtual view query" capability is novel: ask the state "what would
the 3D look like from this viewpoint I never showed you?" and get a
plausible pointmap. Bridges 3D reconstruction with **novel-view
synthesis** and 3D generation.

## Key claims

- **State-input interaction.** Two interconnected transformer decoders:
  - **State-update:** image tokens write to state.
  - **State-readout:** image tokens read from state.
- **Persistent state encodes the scene.** Initialized as learned tokens;
  updated on each frame; survives across the stream.
- **Online + flexible.** Handles video streams *and* unordered photo
  collections; no batch / known camera assumption.
- **Virtual view queries.** Encode a ray map for a virtual viewpoint →
  query the state → get back a pointmap + color for the unseen view.
- **Dynamic scenes natively supported** (unlike DUSt3R / VGGT trained
  on static).
- **Concurrent work:** Spann3R — also continuous reconstruction with
  spatial memory, but Spann3R's memory is a cache, not a generative
  prior.

## Methods

- **Encoder:** ViT per-frame.
- **State:** set of learnable tokens, persisted across frames.
- **Interaction (per frame):**
  1. Encode image to tokens `F_t`.
  2. Run two cross-connected transformer decoders: image tokens ↔
     state tokens. Outputs updated state `s_t` and enriched image tokens
     `F'_t`. A learnable "pose token" `z` rides along to capture
     ego-motion.
  3. Heads:
     - `Head_self`: DPT, predicts `(X̂^self_t, C^self_t)` — pointmap
       in the *current camera's* coordinate frame.
     - `Head_world`: DPT, predicts `(X̂^world_t, C^world_t)` — pointmap
       in the world (initial frame) coordinate frame.
     - `Head_pose`: MLP, predicts ego-motion `P̂_t` (relative
       transform between camera and world frames).
- **Virtual view query:** input a raymap for the virtual camera;
  state-readout produces a pointmap for that view.
- **Training:** large heterogeneous mix — single images, videos, photo
  collections, partial/full 3D annotations, static + dynamic, real +
  synthetic.

## Results (headline)

- Competitive or SOTA on monocular depth, consistent video depth, camera
  pose estimation, 3D reconstruction.
- Can infer unobserved structure from observed images (qualitative
  Fig 2, virtual-view query).
- Handles long streams; recurrence enables long-horizon scenes that
  break batch methods.

## Limitations / open questions

- Recurrence trades some peak-quality vs all-at-once batch methods
  (VGGT etc.) for streaming + online operation.
- Virtual-view inference quality drops far from observed regions.
- Heavy training data mix — not trivial to reproduce.
- Comparison with later all-at-once batch methods (MapAnything, DA3)
  not in this paper; would be informative.

## Connections

- Method: [[cut3r]] (Continuous Updating Transformer for 3D
  Reconstruction).
- Concepts: [[feed-forward-3d-reconstruction]] (recurrent variant),
  [[pointmap-representation]].
- **Closest concurrent / contemporary:** Spann3R (cache memory),
  CUT3R (compressed generative state).
- **Batch-method competitors:** [[vggt]], [[mapanything]],
  [[depth-anything-3]], [[any4d]] — CUT3R is the online answer.
- **3D-from-monocular video competitor:** MonST3R, MegaSaM,
  CasualSAM — all need per-scene optimization. CUT3R is feed-forward
  online.
- Authors: [[qianqian-wang]] (lead).
- Orgs: [[uc-berkeley]] (Efros + Kanazawa) + [[google-deepmind]].

## Citation

Wang, Q., Zhang, Y., Holynski, A., Efros, A. A., & Kanazawa, A. (2025).
*Continuous 3D Perception Model with Persistent State.* arXiv:2501.12387.
https://arxiv.org/abs/2501.12387
