---
type: method
title: Flow3R
status: stub
tags: [scene-flow, visual-geometry-learning, feed-forward, factored-representation, user-affiliated]
sources: []
related:
  - "[[feed-forward-3d-reconstruction]]"
  - "[[shubham-tulsiani]]"
  - "[[cmu-ri]]"
created: 2026-05-26
updated: 2026-05-26
---

# Flow3R

**Authors:** Zhongxiao Cong, Qitao Zhao, Minsik Jeon (user), Shubham
Tulsiani ([[shubham-tulsiani]]). arXiv:2602.20157, 2026.

> ⚠️ **Not yet ingested as a primary source** — this page is seeded
> from a secondary citation in [[anon-2026-point4d]] (reference [3]).
> The Flow3R PDF isn't in `raw/papers/`. The user is a co-author;
> ingest priority is high.

## One-line summary (from title)

**Factored flow prediction for scalable visual geometry learning.**
Positioned in [[anon-2026-point4d]]'s related-work as a peer of
[[vggt]], DUSt3R, [[depth-anything-3]] in the feed-forward
visual-geometry-prediction line — but with a **factored** approach to
flow prediction, plausibly along the same principle that
[[mapanything]] / [[any4d]] argued for (factor outputs into
viewpoint-invariant + time/motion components).

## Where it likely fits

Two threads in the wiki are directly relevant:

- [[feed-forward-3d-reconstruction]] — Flow3R sits in this lineage.
- [[3d-point-tracking]] / [[4d-reconstruction]] — "flow" in the title
  + "scalable visual geometry" suggests this is about scene flow as the
  scalable bridge between 3D and 4D, similar to [[any4d]]'s allocentric
  scene flow output but with a different factorization.

## Open questions until ingest

- What exactly is the factorization? (Plausibly view + time, or rigid +
  non-rigid, or static-base + per-frame-delta.)
- Does Flow3R beat or complement [[any4d]] / [[4rc]] / [[point4d]] on
  the same benchmarks? Point4D cites Flow3R only in its intro listing,
  not in its eval tables.
- How does the user's contribution position the work — methodology,
  experiments, dataset, theory?

## To do on ingest

When the Flow3R PDF lands in `raw/papers/`:

1. Write a full source page using the standard template.
2. Update this method page from stub → growing.
3. Update [[shubham-tulsiani]] to reflect the now-primary citation.
4. Place Flow3R in [[cmp-3d-4d-reconstruction]]'s table.
5. Cross-link to [[any4d]] / [[4rc]] / [[v-dpm]] / [[mapanything]] /
   [[trace-anything]] depending on the closest design choice.
