---
type: concept
title: Online vs Offline (vs Windowed) Tracking
status: growing
tags: [point-tracking, online-tracking, offline-tracking, latency]
sources:
  - "[[zholus-2025-tapnext]]"
  - "[[karaev-2024-cotracker]]"
  - "[[karaev-2024-cotracker3]]"
  - "[[aydemir-2025-track-on2]]"
  - "[[jung-2026-tapnext-plus-plus]]"
related:
  - "[[point-tracking]]"
created: 2026-05-24
updated: 2026-05-24
---

# Online vs Offline (vs Windowed) Tracking

**Conceptual ancestor:** this concept page is the TAP-specific
specialization of [[streaming-perception]] (Li, Wang, Ramanan, ECCV
2020 — [[li-2020-streaming-perception]]). That paper's streaming-
accuracy metric is the formal underpinning of the "frame / window /
video" latency tiers below.

## The axis

The most-cited framing across [[point-tracking]] papers in this wiki is
latency / context-window:

- **Frame / online / causal** — emit per-point output the moment a new
  frame arrives. No future-frame access. Latency = single forward pass.
  Examples: [[tapnext]], [[tapnext-plus-plus]], [[track-on2]].
- **Window / streaming** — consume `T` frames (typically 8–48), emit
  tracks for those T, slide. First output delayed by T frames. Examples:
  [[cotracker]], [[cotracker3]] (online variant), CoTracker2, TAPTR family.
- **Video / offline** — ingest the full video before emitting any output.
  Free to use bidirectional context. Examples: OmniMotion, Dino-Tracker,
  CoTracker3 (offline), [[tapip3d]] (sliding-window with reverse pass).

## Why it matters

- **Robotics / AR / VR / mobile** demand frame-level latency (~5–30 ms).
  Window-based methods can't meet this for the first T frames of a
  stream.
- **Bidirectional context** historically gave window/video methods a
  quality advantage on occlusion + motion ambiguity.

## What's changed (per this wiki batch)

The quality / latency trade-off is **collapsing** as of 2025–2026:

- [[zholus-2025-tapnext]] showed strictly causal SSM-based tracking can
  beat window/video methods on TAP-Vid at ~5 ms latency.
- [[jung-2026-tapnext-plus-plus]] showed the same model with long-sequence
  training also beats them on long-video benchmarks (PointOdyssey).
- [[aydemir-2025-track-on2]] showed a memory-based causal tracker can
  beat all online + offline at synthetic-only training.

**Window methods no longer have a clear quality advantage** for online
TAP, only for some 3D / occlusion-heavy regimes (and maybe not even
there).

## Open questions

- For **3D point tracking**, is the same collapse happening? [[tapip3d]]
  + [[spatialtracker-v2]] are both windowed/video. A causal 3D tracker
  is an open slot.
- How do these online architectures handle **continuous streaming** for
  hours / days? Drift behaviors over very long inputs are
  underbenchmarked.
- Are there tasks (e.g. video editing, post-hoc analysis) where the
  bidirectional offline approach retains an irreducible advantage?
