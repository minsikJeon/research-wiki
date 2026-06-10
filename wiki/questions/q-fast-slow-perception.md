---
type: question
title: Does the πR² slow/fast split have a perception analogue?
status: open
tags: [fast-slow-policy, perception, 3d-reconstruction, 3d-point-tracking, vla, robotics, design-question, latency-adaptive]
sources:
  - "[[anon-2026-pi-r-squared]]"
  - "[[anon-2026-point4d]]"
  - "[[zhang-2026-loger]]"
  - "[[elflein-2026-vgg-t3]]"
related:
  - "[[fast-slow-policy]]"
  - "[[pi-r-squared]]"
  - "[[test-time-training]]"
  - "[[3d-point-tracking]]"
  - "[[4d-reconstruction]]"
  - "[[online-vs-offline-tracking]]"
created: 2026-06-10
updated: 2026-06-10
---

# Does the πR² slow/fast split have a perception analogue?

## The question

[[anon-2026-pi-r-squared]] establishes a **slow/fast channel split**
inside a single VLA:

```
SLOW (background thread, ~60 ms):  image + text → VLM → cached slow_feat
FAST (every control tick, ~20 ms): proprio → small MLP → fresh_proprio
                                    ↓
                                   action head reads (slow_cached, fresh_proprio)
                                    + a learned embedding of d_vlm staleness
```

Critically, vision **lag is in-distribution** because `d_vlm` is sampled
during training. The policy is *taught* to expect stale vision.

This shape mirrors the user's **Caricature 3** ("slow planner + fast
controller") framing from `raw/notes/`. The question is whether the
**perception side** can use the same architectural pattern for online
3D point tracking and 4D reconstruction.

## What a perception analogue would look like

| πR² (control) | Perception analogue |
|---|---|
| Slow VLM backbone (60 ms) | Slow 3D-recon backbone (Point4D, CUT3R, VGG-T3): cached `scene_feat` |
| Fast proprio (μs) | Fast per-frame image enc / patch features (10s of ms) |
| `d_vlm` scalar embedding | `d_recon` scalar embedding for scene-feat staleness |
| Action head reads (cached scene + fresh proprio) | Track head reads (cached scene_feat + fresh frame patches) |
| πR² staircase schedule on action chunk | Staircase schedule on per-position 3D trajectories |

## Why it might work

- [[zhang-2026-loger]] and [[elflein-2026-vgg-t3]] already produce a
  **persistent compressed scene state** via TTT-based fast-weight memory.
  This is exactly the "slow planner" candidate — it's queryable, it's
  bounded-size, and it ages gracefully.
- [[anon-2026-point4d]] produces 3D-query trajectories whose chunking
  pattern maps to πR²'s position-axis.
- The training trick — "sample staleness, train for it" — is mechanism-
  agnostic; nothing about it requires the slow channel to be a VLM.

## Why it might not work (or might need modification)

1. **No proprio analogue.** The "fast modality" in robotics is
   proprioception (joints, torques). In perception, the fast channel
   would have to be patch features from the *current* frame — which
   are not orthogonal to the cached scene_feat the way joints are
   orthogonal to vision.
2. **Scene_feat staleness is observed indirectly.** `d_vlm` is a
   wall-clock measurement; scene-feat staleness has to be measured in
   *frames* or in *scene change*, which is task-dependent.
3. **Perception lacks a robot.** πR²'s motivation is "robot can't wait
   for vision"; perception's motivation is harder to articulate without
   a downstream task to align to.

## What would settle it

- **An MVP**: implement the πR² recipe on a perception backbone (e.g.,
  Point4D + light track head), train with staleness-sampled scene_feat,
  measure online accuracy at varying scene-feat refresh rates. The
  user's `2026-06-08_MiddleGround_MVP.md` is exactly this.
- **A benchmark** that rewards low per-frame latency on streaming 3D
  data — currently TAPVid-3D / Sintel evaluate on full clips offline.
  Streaming variants would surface whether async perception is needed.

## Status

Open. The user's middle-ground MVP is the concrete test case.
The expected outcome is that the recipe transfers but the
staleness measurement (`d_recon` vs `d_vlm`) needs reformulating
as scene-change rather than wall-clock.
