---
type: method
title: LaCET (Large-Chunk Elastic Test-Time Training)
status: growing
tags: [test-time-training, fast-weights, elastic-weight-consolidation, long-context, chunk-wise]
sources:
  - "[[ma-2026-fsm]]"
related:
  - "[[lact]]"
  - "[[test-time-training]]"
  - "[[fsm]]"
  - "[[loger]]"
  - "[[vgg-t3]]"
  - "[[sun-2024-ttt]]"
created: 2026-06-24
updated: 2026-06-24
---

# LaCET (Large-Chunk Elastic Test-Time Training)

Extension of [[lact]] that adds a **consolidate** step after each
chunk's fast-weight update, inspired by Elastic Weight Consolidation
(EWC) from continual learning. Solves the problem that fully plastic
multi-chunk LaCT updates lead to catastrophic forgetting and overfitting.

## One-line summary

LaCT + Fisher-weighted elastic spring pulling fast weights toward a
streaming-EMA anchor after each chunk update.

## Inputs / outputs

Same as [[lact]]: sequence of tokens → sequence of outputs, with
fast-weight state `θ` carried across chunks.

Additional state: anchor weights `θ*` and per-parameter importance
estimates `F_c`.

## How it works

### LaCET block (three operations per chunk)

```
1. Update (same as LaCT):
   θ'_{c+1} = θ_c − ∇_θ Σᵢ ηᵢ L(f_θ(kᵢ), vᵢ)

2. Consolidate (new):
   θ_{c+1} = θ'_c − λ F_c ⊙ (θ'_c − θ*_c)

3. Apply (same as LaCT):
   o_i = f_θ(q_i)
```

The consolidation term acts as an adaptive spring: parameters with
high Fisher importance are pulled strongly toward their anchors,
preserving important past information; parameters with low importance
adapt freely.

### Importance estimation (F_c)

Maintained as an EMA with decay α ∈ [0, 1]:
```
F_{c+1} = α F_c + (1 − α) φ(S_c)
```

Three variants of the statistic `S_c`:
- **MAS:** `|θ'_c − θ_c|` — magnitude of the chunk update
- **EWC:** `(θ'_c − θ_c)²` — squared gradient (original Kirkpatrick)
- **SI:** `(θ'_c − θ_c) ⊙ (θ'_c − θ*_c)` — update × drift from anchor

SI works best empirically (Table 1 of source), possibly because it
captures both the update magnitude and how far the parameter has
drifted from its anchor.

### Anchor update policies

- **Global:** `θ*` fixed to initialization. Degenerates to a weighted
  L2 regularizer. Stable but not adaptive.
- **Streaming:** `θ*` reset to current `θ` at each chunk boundary.
  Regularizes within-chunk drift only. Prone to overfitting.
- **Streaming-EMA:** `θ* ← βθ* + (1−β)θ`. Forms a low-pass filter
  over the fast-weight trajectory. **Best policy** — genuinely elastic
  behavior emerges only with this anchor evolution.

## Where it's been applied

- **[[fsm]]** (FSM-LVSM and FSM-LRM) — 4D novel view synthesis on
  Stereo4D, NVIDIA, and DL3DV benchmarks. SOTA among feed-forward 4D
  methods.

## Known limitations

- **Adds two hyperparameters:** `λ` (consolidation strength) and `α`
  (importance EMA decay). Empirically robust (SI + streaming-EMA is
  default), but no principled selector.
- **Only tested within FSM's 4D NVS setting.** Could in principle be
  applied to [[loger]]'s 3D reconstruction or [[lact]]'s language
  modeling and video diffusion, but not yet validated there.
- **Single-chunk regime:** when chunk = full sequence, consolidation
  degenerates to a second-order correction O(λ(Δθ)²) and is negligible.
  LaCET's value is specifically for multi-chunk inference.

## Related methods

- **Direct precursor:** [[lact]] — LaCET = LaCT + consolidation.
- **EWC origin:** Kirkpatrick et al. (2017) — continual learning; the
  "task A → task B" framing reinterpreted as "chunk c → chunk c+1."
- **Alternative memory mechanisms:** MAS (Aljundi et al. 2018), SI
  (Zenke et al. 2017) — FSM tests all three Fisher estimators.
- **Same-paradigm 3D/4D:** [[loger]] (chunked TTT + SWA, no elastic),
  [[vgg-t3]] (global TTT, no chunking), TTT3R (per-frame TTT).
