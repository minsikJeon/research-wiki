---
type: source
source_type: paper
title: "Learning to (Learn at Test Time): RNNs with Expressive Hidden States"
authors:
  - Sun, Yu
  - Li, Xinhao
  - Dalal, Karan
  - Xu, Jiarui
  - Vikram, Arjun
  - Zhang, Genghan
  - Dubois, Yann
  - Chen, Xinlei
  - Wang, Xiaolong
  - Koyejo, Sanmi
  - Hashimoto, Tatsunori
  - Guestrin, Carlos
year: 2024
venue: arXiv (cs.LG)
url: https://arxiv.org/abs/2407.04620
raw_path: external (not in raw/papers/)
status: ingested (web)
tags: [sequence-modeling, rnn, transformer, test-time-training, fast-weights, long-context, language-modeling]
sources: []
related:
  - "[[test-time-training]]"
  - "[[loger]]"
  - "[[vgg-t3]]"
created: 2026-05-31
updated: 2026-05-31
---

# Learning to (Learn at Test Time): RNNs with Expressive Hidden States

## TL;DR

**Foundational paper for [[test-time-training]].** The central thesis:
make the RNN hidden state a *parametric model* `f_W` whose weights are
updated by self-supervised gradient descent on the input stream.
This combines RNN's linear-time efficiency with Transformer-quality
long-context handling.

## The core argument (why TTT exists)

Sequence modeling has a fundamental trade-off:

| Architecture | Per-token cost | Memory | Long-context |
|---|---|---|---|
| Transformer | `O(N)` | Linear (KV cache) | Strong |
| RNN / Mamba | `O(1)` | Constant (hidden state) | **Weak** — Mamba's perplexity plateaus after 16k tokens |

The paper frames this as: **RNNs' advantage (linear compute) only matters in long-context settings, but RNNs fail there because their compression is too lossy.**

**TTT's bet:** if compression is the problem, use the **best compression mechanism we know** — self-supervised learning. *"Self-supervised learning can compress a massive training set into the weights of a model, which often exhibits deep understanding about the semantic connections among its training data."* Apply that mechanism at inference time, per scene/sequence, with the input stream as the training set.

## Key claims

- **Hidden state = a model.** The RNN's hidden state `s_t` is replaced by `W_t`, the weights of an inner model `f`. Update rule: `W_t ← W_{t-1} − η ∇_W L(f_W(k_t), v_t)` where `(k_t, v_t)` are projections of the current token.
- **Update rule = self-supervised gradient step.** Each token "trains" the inner model. The outer model learns to choose K, V, Q projections that make this self-supervised task useful for the downstream loss.
- **TTT-Linear and TTT-MLP.** Two instantiations — inner model = linear layer or 2-layer MLP. Both are drop-in replacements for attention.
- **Beats Mamba on long context.** TTT-Linear and TTT-MLP keep reducing perplexity at 32k+ context where Mamba flattens.
- **Competitive with Transformer.** At 125M–1.3B param scale, comparable perplexity with linear compute.

## Why this matters beyond LLMs

The reason TTT shows up in 3D reconstruction ([[loger]], [[vgg-t3]], TTT3R) is that the *same* RNN-vs-Transformer trade-off exists for long video:

- VGGT (global attention) = Transformer = `O(N²)`, OOMs at 1k frames.
- CUT3R (recurrent hidden state) = RNN = constant memory but lossy beyond ~hundreds of frames.
- LoGeR / VGG-T3 / TTT3R = TTT applied to 3D = linear-time + expressive per-scene memory.

The 3D community is essentially porting TTT from language modeling to multi-view geometry — same motivation (long-context with bounded memory), same mechanism (per-scene fast-weights via gradient descent on a self-supervised loss).

## Connections

- Concept page: [[test-time-training]] — the cross-source pattern in this wiki.
- 3D applications: [[loger]] (per-chunk TTT + SWA), [[vgg-t3]] (per-scene TTT global compression), TTT3R (per-frame TTT — promote next batch).
- Earlier fast-weight ancestors: Schlag et al. 2021 (linear attention as fast weight programmer); Katharopoulos et al. 2020 (linear attention).
- Successor / variants: LaCT [Zhang et al. 2025]; Titans [Behrouz et al. 2024].

## Citation

Sun, Y., Li, X., Dalal, K., Xu, J., Vikram, A., Zhang, G., Dubois, Y., Chen, X., Wang, X., Koyejo, S., Hashimoto, T., & Guestrin, C. (2024). *Learning to (Learn at Test Time): RNNs with Expressive Hidden States.* arXiv:2407.04620. https://arxiv.org/abs/2407.04620
