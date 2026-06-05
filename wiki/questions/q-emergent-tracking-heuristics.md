---
type: question
title: Do classical tracking heuristics emerge from end-to-end training?
status: open
tags: [point-tracking, interpretability, emergent-behavior]
sources:
  - "[[zholus-2025-tapnext]]"
  - "[[jung-2026-tapnext-plus-plus]]"
related:
  - "[[point-tracking]]"
  - "[[tapnext]]"
  - "[[tapnext-plus-plus]]"
created: 2026-05-24
updated: 2026-05-24
---

# Do classical tracking heuristics emerge from end-to-end training?

## The question

[[zholus-2025-tapnext]] observed that even though TAPNext has **no
tracking-specific inductive biases** (no cost volume, no iterative
refinement, no local search window, no temporal smoothness prior),
ViT attention heads spontaneously implement patterns that look exactly
like those heuristics:

- **Cost-volume-like attention head** — appearance matching to image
  patches.
- **Coordinate read-out head** — directly reading position info.
- **Motion-cluster attention head** — grouping points that move together
  (a form of implicit [[joint-point-tracking]]).

So: **can these emergent heuristics be quantified, forced earlier in
training, or used to improve data efficiency / interpretability?**

## Why it matters

Two practical reasons:

1. **Data efficiency.** TAPNext needs 500K × 48-frame Kubric clips to
   converge; TAPNext++ pushes to 1024-frame sequences. If we could
   *seed* the right attention patterns directly (instead of waiting for
   them to emerge), training could be cheaper.
2. **Interpretability for failure-mode analysis.** When a tracker fails,
   knowing *which* attention pattern broke (appearance vs. motion vs.
   coordinate) would localize the diagnosis.

## What we know

- [[zholus-2025-tapnext]] visualizes the three patterns qualitatively
  (Figs 3-4) but doesn't quantify the contribution of each.
- [[jung-2026-tapnext-plus-plus]] doesn't re-examine the patterns
  explicitly — the architecture is unchanged, so they presumably persist,
  but no analysis is provided.
- TAPNext's ablation (Table 3 of the source) shows the
  *classification coordinate head* is the single most important
  component (44.7 → 55.0 AJ vs regression). This is loosely consistent
  with the "coordinate read-out head" emergence: classification gives
  a clean signal for the read-out to lock onto.

## What we don't

- **No quantitative attribution** of each emergent pattern to final
  performance (head ablations / patching).
- **No comparison of emergence under different training regimes** —
  does long-sequence training in TAPNext++ strengthen or weaken
  these patterns? Different training data?
- **Does this generalize beyond TAPNext?** Track-On2 has an explicit
  memory module and classification head — do similar patterns appear
  there in spite of (or because of) the architectural priors? No
  visualization-style analysis published.

## Tentative position

The emergence claim is compelling but under-evidenced. The right next
step is:

1. **Head-ablation study** on a trained TAPNext / TAPNext++ — patch
   each "emergent" head with random / zero attention and measure AJ
   drop. This would quantify each pattern's contribution.
2. **Probe how patterns evolve during training** — do all three appear
   together, or in a curriculum? Could inform initialization /
   pre-training strategies.
3. **Cross-architecture comparison** — train [[track-on2]] without its
   memory module + decoded queries, see if SSM/ViT-style attention
   patterns emerge. Conversely, train TAPNext with an explicit memory
   block, see what gets duplicated.

## Next things to read

- Original TAPNext appendix B (full-resolution attention visualizations).
- Doersch et al. on TAP-Vid / TAPIR — predecessors don't have emergent
  analysis but their architectures explicitly bake in these patterns.
- Look for **interpretability papers on video transformers** — the
  techniques (probing classifiers, attention rollout, attribution)
  developed for image transformers should transfer.
