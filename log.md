# Log

Chronological, append-only record of wiki activity. See [[AGENTS]] §5 for the
entry format.

Quickly view recent activity:
```
grep "^## \[" log.md | tail -10
```

---

## [2026-05-24] note | wiki scaffolded

Initial scaffold. Created [[AGENTS]] (schema), [[index]], [[log]],
[[overview]], and the `raw/` + `wiki/` directory trees. No sources
ingested yet.

## [2026-05-24] refactor | re-scoped wiki from MSR=Mining-Software-Repositories to MSR=Master-of-Science-in-Robotics

Misread the directory name on scaffold. Corrected [[AGENTS]] (domain
paragraph, example slugs, example tags, example log entry), rewrote
[[overview]], and updated the user-role memory. Scope is now CV +
robotics + adjacent (foundation models, video understanding, SLAM,
manipulation, imitation learning, world models, embodied AI). No
files in `raw/` or `wiki/` were affected — only the schema docs.

## [2026-05-24] ingest | Zholus 2025 — TAPNext (arXiv 2504.05579)

First real source. Added [[zholus-2025-tapnext]] (full summary).
Created [[point-tracking]] (concept), [[tapnext]] (method),
[[carl-doersch]] (person — recurring TAP-line author),
[[google-deepmind]] (org), [[tap-vid-dataset]] (dataset).
Updated [[index]] and seeded the tag set. Conservative entity creation:
deferred TAPIR, CoTracker, LocoTrack, TAPTR, BootsTAP, RecurrentGemma,
TRecViT, Kubric, DAVIS, Kinetics, MultiMAE, VideoMAE — promote on the
second mention per [[AGENTS]] §6.

## [2026-05-24] note | poppler installed via Homebrew

PDF reading dependency. `brew install poppler` adds `pdftotext` +
`pdftoppm`. Read tool needs pdftoppm on PATH; until that's resolved at
launch, use `pdftotext -layout` via Bash to extract text and Read the
resulting `.txt` from `/tmp`. Works fine for academic PDFs.

## [2026-05-24] ingest | TAP-family batch (6 papers)

Batch ingest of 6 point-tracking papers in one pass — coherent thematic
group covering 2D online (CoTracker, CoTracker3, Track-On2, TAPNext++)
and 3D (TAPIP3D, SpatialTrackerV2).

**New source pages:** [[karaev-2024-cotracker]],
[[karaev-2024-cotracker3]], [[aydemir-2025-track-on2]],
[[jung-2026-tapnext-plus-plus]], [[zhang-2025-tapip3d]],
[[xiao-2025-spatialtracker-v2]].

**New method pages (7, incl. update):** [[cotracker]], [[cotracker3]],
[[track-on2]], [[tapnext-plus-plus]], [[tapip3d]],
[[spatialtracker-v2]]; updated [[tapnext]] with successor link.

**New concept pages (5, incl. update):** [[joint-point-tracking]],
[[3d-point-tracking]], [[online-vs-offline-tracking]],
[[synthetic-to-real-gap]], [[pseudo-labeling-point-tracking]];
rewrote [[point-tracking]] with evidence across all 7 sources.

**New entity pages:** people — [[nikita-karaev]] (3 papers),
[[artem-zholus]] (2 papers), [[katerina-fragkiadaki]] (CMU, TAPIP3D),
[[adam-w-harley]] (PIPs + TAPIP3D), [[gorkay-aydemir]] (Track-On line);
orgs — [[meta-ai]], [[oxford-vgg]], [[cmu-ri]] (user's school);
datasets — [[kubric-dataset]], [[pointodyssey-dataset]],
[[dynamic-replica-dataset]], [[tapvid-3d-dataset]];
updated [[carl-doersch]], [[google-deepmind]], [[tap-vid-dataset]] with
new source links.

**Synthesis pages:** [[cmp-tap-methods]] (comprehensive comparison
table), [[q-emergent-tracking-heuristics]] (from prior TAPNext flag),
[[q-sim2real-for-point-tracking]] (now well-supported).

**Tag set expanded** to 40 canonical tags — see [[index]].

**Deferred entities (still <2 mentions):** PIPs / PIPs++, TAPIR,
BootsTAP / BootsTAPIR, LocoTrack, TAPTR family, DepthAnything, DINOv3,
MoGe, MegaSaM, MonST3R, DUSt3R, VGGT, RecurrentGemma, TRecViT, VKITTI,
RoboTAP. Andrea Vedaldi + Christian Rupprecht (CoTracker seniors).
**BootsTAP is the most important to ingest next** — cited by 4 of 6
batch sources but no source page exists.

**Notable cross-source findings (now in [[overview]]):**
- Online causal tracking has matched or surpassed windowed methods for
  2D TAP (TAPNext, TAPNext++, Track-On2 all SOTA at frame-level latency).
- Synthetic-only training is no longer obviously worse than real-data
  fine-tuning — Track-On2 + TAPNext++ are evidence on Camp B side.
- 3D tracking has two parallel design philosophies (modular vs.
  end-to-end) without a definitive head-to-head yet.

## [2026-05-24] ingest | 3D/4D reconstruction batch (8 papers)

Second large batch. Coherent theme: **feed-forward 3D + 4D
reconstruction foundation models** — the layer underneath the 3D point
tracking work from the prior batch.

**New source pages (8):** [[wang-2025-vggt]] (the trunk),
[[keetha-2025-mapanything]] (CMU + Meta universal 3D),
[[lin-2025-depth-anything-3]] (ByteDance Seed minimalist),
[[wang-2025-cut3r]] (UC Berkeley + DeepMind online recurrent),
[[karhade-2025-any4d]] (CMU multi-modal metric 4D),
[[sucar-2026-v-dpm]] (Oxford VGG, VGGT→4D fine-tune),
[[luo-2026-4rc]] (NTU + Oxford VGG, conditional query 4D),
[[anon-2026-trace-anything]] (ICLR 2026 anon, trajectory fields).

**New method pages (9):** [[vggt]], [[mapanything]], [[depth-anything-3]],
[[cut3r]], [[any4d]], [[v-dpm]], [[4rc]], [[trace-anything]] + promoted
[[dust3r]] (cited by 6+ sources; not yet ingested as primary).

**New concept pages (5):** [[4d-reconstruction]],
[[feed-forward-3d-reconstruction]], [[pointmap-representation]],
[[dynamic-point-maps]], [[trajectory-fields]].

**New entity pages:** people — [[nikhil-keetha]], [[deva-ramanan]],
[[sebastian-scherer]] (3 new CMU faculty/students — user-affinity),
[[jay-karhade]] (CMU), [[qianqian-wang]] (Berkeley/DeepMind),
[[jianyuan-wang]] (VGGT lead), [[andrea-vedaldi]] (promoted — now
4 sources, most recurring senior in wiki), [[christian-rupprecht]]
(3 sources), [[bingyi-kang]] (ByteDance Seed); orgs —
[[bytedance-seed]], [[meta-reality-labs]], [[uc-berkeley]]; updates
to [[cmu-ri]] (now 3 sources), [[meta-ai]] (added VGGT), [[oxford-vgg]]
(now 6 sources — most prolific org).

**Synthesis pages:** [[cmp-3d-4d-reconstruction]] (comprehensive 3D/4D
comparison table), [[q-tracking-vs-4d-reconstruction]] (new open
question on convergence of the two sub-fields).

**Tag set expanded** to ~55 canonical tags — see [[index]].

**Raw-file note:** `raw/papers/attachment.pdf` is actually
**Trace Anything** (ICLR 2026 anonymous). User should rename to
`anon-2026-trace-anything.pdf` to match the wiki slug. The wiki never
modifies `raw/`; flagged in the source page metadata.

**Deferred entities (still <2 source mentions):** DUSt3R/MASt3R primary
ingest (now seeded as a method page), π³ (Pi3), Fast3R, MV-DUSt3R+,
VGGSfM, Spann3R, MonST3R, POMATO, Easi3R, St4RTrack, MegaSaM, MoGe,
DepthAnything 1/2, BootsTAP. Edgar Sucar (V-DPM lead), Yihang Luo (4RC
lead), Haotong Lin (DA3 lead), Norman Müller / Johannes Schönberger
(MapAnything co-authors). NTU S-Lab (4RC).

**Notable cross-source findings (now in [[overview]]):**
- VGGT is the **trunk architecture** of a 2025-2026 wave. V-DPM,
  MapAnything, 4RC, DA3, Any4D, CUT3R all build on or contrast with it.
- Two parallel waves are **converging**: 3D point tracking and 4D
  reconstruction now produce nearly the same outputs — see
  [[q-tracking-vs-4d-reconstruction]].
- CMU's 3D-reconstruction-for-robotics output is substantial:
  TAPIP3D (Fragkiadaki) + MapAnything + Any4D (Ramanan + Scherer +
  Keetha + Karhade).

## [2026-05-26] ingest | Point4D (NeurIPS 2026 anon)

Single-paper ingest. Closes a specific gap I flagged in [[overview]]:
the prior feed-forward 4D wave (Any4D, 4RC, V-DPM, Trace Anything) is
bottlenecked at ~100 frames; Point4D extends to 200-300 frames via
**3D-query-based motion decoding + chunk chaining**.

**New source page:** [[anon-2026-point4d]] (NeurIPS 2026 anonymous;
encoder initialized from [[depth-anything-3]]).

**New method page:** [[point4d]].

**New concept page:** [[trajectory-chaining]] — the strategy itself,
with explicit 2D-pixel-query vs 3D-coordinate-query failure-mode
analysis from Point4D's ablations.

**De-anonymization of [[anon-2026-trace-anything]]:** Point4D's
reference [17] gives the authors — Xinhang Liu, Yuxi Xiao (also
[[spatialtracker-v2]] lead), Donny Y. Chen, Jiashi Feng, Yu-Wing Tai,
Chi-Keung Tang, Bingyi Kang ([[bingyi-kang]], DA3 lead). arXiv 2510.13802.
Updated the source-page frontmatter + citation. Also noted the slug
is now stale (`anon-2026-...`) — keeping it to avoid breaking links;
flagged for future refactor.

**De-anonymization confirms a third research cluster:**
HKUST + ByteDance Seed + Yuxi Xiao group — alongside the
[[oxford-vgg]] (Vedaldi line) and [[cmu-ri]] (Ramanan/Scherer/Fragkiadaki
line) clusters already mapped. Trace Anything is the dense 4D extension
of SpatialTrackerV2's tracking line.

**Notable cross-source finding (now in [[overview]] thesis):**
**The Trace Anything Bézier-spline parameterization is empirically the
worst feed-forward 4D method at long horizons** — Point4D's Table 1
shows TraceAnything EPE 2.059 on PointOdyssey-200, vs 4RC 0.688,
V-DPM 0.644, Point4D 0.526. Confirms the 4RC paper's earlier
theoretical critique. Splines genuinely struggle at long horizons.

**Held back (still <2 source mentions):** **D4RT** [Zhang et al. 2025,
arXiv 2512.08924] is the **direct conceptual predecessor of Point4D** —
introduced the query-based 4D decoder for 2D pixel queries. **Highest-
priority next ingest** — Point4D explicitly extends D4RT to 3D queries.
Also new: Flow4R, DELTA, St4RTrack, InfiniteVGGT, VGGT-Long,
StreamingVGGT, Loger, Shape of Motion. **And:** Point4D's reference
[3] is "Flow3R" (arXiv 2602.20157) — a paper co-authored by the user
(Minsik Jeon) with Shubham Tulsiani. User notified separately.

**Updated [[cmp-3d-4d-reconstruction]]** with Point4D row and a long-
video numbers section.

## [2026-05-26] ingest | D4RT (arXiv 2512.08924)

Single-paper ingest. The **direct conceptual predecessor of
[[point4d]]** I flagged last turn. Lead [[mehdi-sajjadi]]
(DeepMind), substantial overlap with the DeepMind TAP cluster.

**New source page:** [[zhang-2025-d4rt]] (Google DeepMind + UCL +
Oxford; Mehdi Sajjadi lead; Andrew Zisserman senior).

**New method page:** [[d4rt]] (promoted from secondary stub).
Canonical 2D-pixel-query 4D decoder. Encoder + cross-attention
decoder; one query interface for six tasks (point tracks, point
clouds, depth, extrinsics, intrinsics, 4D correspondence).
SRT (Scene Representation Transformer) lineage.

**New entity pages promoted past 2-source threshold:**
- [[mehdi-sajjadi]] — D4RT lead + TAPNext co-author; SRT line.
- [[skanda-koppula]] — TAPVid-3D introducer + TAPNext + D4RT co-author.
  Triple connection: benchmark + two methods that beat the benchmark.
- [[ignacio-rocco]] — CoTracker + TAPNext + D4RT co-author (Meta AI →
  DeepMind transition).

**Updated entity pages:** [[google-deepmind]] (now 4 sources),
[[tapvid-3d-dataset]] (D4RT now holds SOTA; +Koppula attribution).
Updated [[anon-2026-point4d]] + [[point4d]] to cross-link the now-
ingested D4RT. Updated [[trajectory-chaining]] concept page to wiki-
link D4RT properly.

**Headline cross-source findings (now in [[overview]]):**

- **D4RT is SOTA on both static-3D AND 4D tracking simultaneously**
  (Sintel pose ATE 0.065 vs π³ 0.086 / VGGT 0.168; TAPVid-3D
  DriveTrack AJ 0.304 vs SpatialTrackerV2 0.195). This is a significant
  unification claim — one model, six tasks, all SOTA.
- **D4RT runs 9× faster than VGGT, 100× faster than MegaSaM** at pose
  estimation, 200+ FPS on A100. Efficiency advantage is real, not
  marginal.
- **Encode-once + independent cross-attention decoder is now the
  dominant 4D pattern.** D4RT formalizes it; [[point4d]] (3D queries)
  and [[4rc]] (AdaLN + base+delta output) are direct descendants.
  [[any4d]], [[v-dpm]], [[trace-anything]] are alternatives but on
  the same encoder backbone family.
- **DeepMind TAP/4D cluster is one coherent research program.** The
  author overlap across TAPVid-3D, TAPNext, BootsTAP, D4RT confirms
  it. [[carl-doersch]] is the longest thread; [[mehdi-sajjadi]] now
  joins as a second senior; [[skanda-koppula]] / [[ignacio-rocco]]
  cross-pollinate.

**Updated [[cmp-3d-4d-reconstruction]]** with D4RT in the high-level
table + the static-3D comparison row.

**Held back (still <2 source mentions):** SRT (Sajjadi 2022 — the
encoder-decoder ancestor; highest-priority next foundation paper),
π³ (Pi3 — VGGT successor; D4RT explicitly outperforms it), Andrew
Zisserman (Oxford VGG senior; will accumulate fast). DELTA, Flow4R,
St4RTrack, Kauldron training framework.

## [2026-05-28] ingest | Streaming Perception + RTC (two-paper batch)

Two unrelated papers that opened **two new research threads** for the
wiki:

**Paper 1: [[li-2020-streaming-perception]]** (Li, Wang, Ramanan,
ECCV 2020). Foundational paper for latency-aware perception. The
**conceptual ancestor of [[online-vs-offline-tracking]]** — every
"online / causal / streaming" discussion in this wiki traces back here.
Headline finding: offline AP 38.0 → streaming AP 6.2 → 17.8 (with
tracking + forecasting + scheduling) → 20.3 (infinite GPUs). The
algorithmic ceiling matters more than hardware. Also brings
**[[deva-ramanan]] to 3 sources** — his line on embodied perception
spans 2020 → 2025 (MapAnything, Any4D).

**Paper 2: [[black-2025-rtc]]** (Black, Galliker, Levine, NeurIPS 2025).
Brand-new robotics thread for the wiki. Real-time chunking via
inpainting for VLA action execution. Inference-time only; works on any
diffusion / flow VLA. Robust to >300ms latency. Tested on π0.5 base
policy + 6 real bimanual tasks.

**New source pages (2):** [[li-2020-streaming-perception]],
[[black-2025-rtc]].

**New method pages (2):** [[streamer-meta-detector]], [[rtc]].

**New concept pages (7):** [[streaming-perception]],
[[streaming-accuracy]], [[action-chunking]], [[asynchronous-control]],
[[vla]], [[flow-matching]], [[train-inference-mismatch]].

The [[train-inference-mismatch]] concept page explicitly links the
recurring pattern across **four** sources now in the wiki:
[[xiao-2025-spatialtracker-v2]] (perception heuristic),
[[anon-2026-point4d]] (perception structural),
[[black-2025-rtc]] (control structural),
[[li-2020-streaming-perception]] (perception via forecasting).
**This is the most important cross-cutting concept added in this
ingest.** Captures the analogy the user noticed in conversation: action
chunking ↔ sliding-window inference.

**New entity pages:** people — [[kevin-black]], [[sergey-levine]],
[[mengtian-li]], [[yu-xiong-wang]]; orgs —
[[physical-intelligence]], [[uiuc]]; datasets —
[[argoverse-hd-dataset]].

**Updated entities:** [[deva-ramanan]] (promoted; arc from 2020
streaming-perception → 2025 4D reconstruction is now explicit),
[[uc-berkeley]] (RTC added), [[cmu-ri]] (Li/Ramanan 2020 added),
[[online-vs-offline-tracking]] (now back-links to streaming-perception
as conceptual ancestor).

**Notable cross-source findings (in [[overview]]):**

- **The TAP-side latency framing is a special case of streaming
  perception.** This wiki's [[online-vs-offline-tracking]] concept page
  is now explicitly nested under [[streaming-perception]].
- **The "tracking and forecasting emerge as necessary" finding from
  2020 parallels [[q-emergent-tracking-heuristics]] from the TAPNext
  discussion** — both about useful representations emerging from
  end-to-end pressure. Worth potential consolidation into a single
  question page.
- **Train-inference mismatch is a recurring meta-pattern.**
  Perception and control both face it; the structural fixes (Point4D's
  3D queries, RTC's inpainting) consistently beat heuristic fixes
  (SpaTrackerV2's confidence-weighted overlap selection, temporal
  ensembling).

**Held back (still <2 source mentions):** π0, π0.5 (Black et al. 2024;
high-priority follow-up for the robotics thread), OpenVLA,
DiffusionPolicy/ACT (Zhao 2023), ΠGDM (Song 2023), Kalman filtering
in modern perception. StreamYOLO + later streaming-perception
follow-ups. Chelsea Finn / Karol Hausman (PI co-founders not yet on
ingested papers).

## [2026-05-29] ingest | Long-context 3D reconstruction (VGG-T3 + LoGeR)

Two papers that open a new section in the wiki:
**long-context / linear-time 3D reconstruction**. Both replace
softmax attention with a **fast-weight associative memory**
([[test-time-training]]) but apply it at different granularities.
Directly relevant to the user's Topic 3 planning note (long-term
real-time 3D tracking).

**Paper 1: [[elflein-2026-vgg-t3]]** (Elflein et al., NVIDIA + Vector
Inst + U. Toronto, arXiv:2602.23361, Feb 2026). Replaces VGGT's
variable-length KV in global attention with a fixed-size SwiGLU MLP
fit per-scene via TTT. **Linear in input view count, but
**global/offline/unordered** — the missing fourth slot in the
existing taxonomy. 1k images in 58 s vs VGGT 11 min (11.6× speedup);
2k images in 48.5 s on 4 GPUs (33× vs VGGT 27 min). Built on a
*frozen* VGGT backbone; ~12% of scratch-training cost. Side capability:
the fitted MLP is queryable → feed-forward visual localization.

**Paper 2: [[zhang-2026-loger]]** (J. Zhang et al., Google DeepMind
+ UC Berkeley, arXiv:2603.03269, Apr 2026). Chunk-wise feed-forward
3D with **hybrid memory**: SWA across `C_{m-1} ∪ C_m` (lossless
local) + chunk-wise TTT fast weights (compressed global) + bidi
within-chunk attention. **Trained on 128 frames, generalizes to 1k
out of the box, 19k with `LoGeR*` feed-forward SIM(3) alignment.**
74% ATE reduction on KITTI over TTT3R; 55.2% relative gain on a new
VBR long-sequence benchmark (8.8k–18.8k frames, 11.5 km).
**Architectural exemplar of Caricature 3 in the user's planning
note** — bidirectional reasoning + fast-weight global state.

**New source pages (2):** [[elflein-2026-vgg-t3]], [[zhang-2026-loger]].

**New method pages (2):** [[vgg-t3]], [[loger]].

**New concept page (1):** [[test-time-training]] — load-bearing
across multiple sources now (VGG-T3 global-offline, LoGeR
streaming-chunked, TTT3R per-frame). The page enumerates the three
patterns and is the canonical entry point for TTT-as-3D-memory.

**New entity pages:** people — [[junyi-zhang]] (LoGeR lead + MonST3R
author, the latter now cited 5+ times across the wiki); orgs —
[[nvidia]].

**Updated entities:** Index reorganized to add "Long-context 3D
reconstruction" subsection under sources; new tag set
(`test-time-training`, `fast-weights`, `kv-compression`,
`linear-complexity`, `hybrid-memory`, `sliding-window-attention`,
`long-context`, `chunk-wise`).

**Notable cross-source findings:**

- **Test-time training is the new fast-weight mechanism** of choice
  for scaling 3D reconstruction past 1k views. Three design patterns
  coexist (global-offline, streaming-chunked, per-frame); LoGeR's
  empirical results suggest **per-frame is the worst** for geometric
  coherence (TTT3R loses ~4× ATE to LoGeR on KITTI) — important
  signal for the user's Caricature 1 design (pure auto-regressive).
- **Hybrid memory > single mechanism.** LoGeR Tab. 1 makes this
  explicit: full attn (lossless local + lossless global, `O(N²)`);
  SWA (lossless local + limited global, `O(N)`); TTT/linear
  (compressed local + compressed global, `O(N)`); LoGeR
  (**lossless local + compressed global**, `O(N)`). The missing
  cell.
- **Data wall ≥ context wall.** LoGeR's most counter-intuitive
  claim: architectural fixes (FastVGGT et al.) cannot bridge to
  large-scale scenes without **long-context training data**
  (TartanAirV2). Has direct implication for the user's planning
  note Part 2 Axis E (training strategy must include long-horizon
  data, not optional).
- **Heavy chunks make scene representation, not better tracks
  (planning note 1.5.3) is empirically confirmed.** VGG-T3's fitted
  MLP *is* the compressed scene; LoGeR's TTT fast weights *anchor
  the global coordinate frame*. Both papers explicitly frame the
  heavy compute as scene-grounding, not tracking.

**Held back (still <2 source mentions, but newly cited so promote
soon):**
- **TTT3R** [Chen et al. 2026] — autoregressive per-frame TTT
  baseline. Now cited by both VGG-T3 and LoGeR. **Promote to
  method page on next ingest.**
- **π³ (Pi3)** [Wang et al. 2026] — VGGT successor; LoGeR's backbone.
  Promote when seen again.
- **MonST3R** [Zhang et al. 2025a] — cited 5+ times. **Promote to
  primary source page.**
- **FastVGGT**, **SparseVGGT**, **VGGT-Long**, **VGGT-SLAM**,
  **StreamVGGT**, **Stream3R**, **InfiniteVGGT** — all now cited
  by both VGG-T3 and LoGeR; promote next batch.
- **Pi3-Chunk** baseline from LoGeR (chunk-causal Pi3 with TTT
  reset) — interesting as a *minimal baseline* for the user's
  Caricature 2.

## [2026-06-08] ingest | πR² + Training-Time RTC + Diffusion Forcing + ACT + Diffusion Policy (5-paper batch)

Five-paper batch closing the real-time-chunking / fast-slow-policy thread
end-to-end, from foundational (ACT/DP) through the underlying mechanism
(Diffusion Forcing) through the recent line of fixes (Training-Time RTC →
πR²).

**New sources (5):**
- [[zhao-2023-act]] — ALOHA + ACT (RSS 2023). Origin of the
  [[action-chunking]] paradigm. Grounded the concept page that had been
  citing it indirectly through [[black-2025-rtc]].
- [[chi-2024-diffusion-policy]] — Diffusion Policy (RSS 2023 / IJRR
  2024). Conditional DDPM over action chunks; +46.9% over prior SOTA;
  foundation of every modern diffusion-VLA action head.
- [[chen-2024-diffusion-forcing]] — Diffusion Forcing (NeurIPS 2024,
  MIT CSAIL). Per-token independent noise levels at training; 2D
  sampling grid at inference. Bridges teacher forcing and full-sequence
  diffusion.
- [[black-2025-training-time-rtc]] — Training-Time RTC (Physical
  Intelligence). Drop-in replacement for inference-time RTC; moves
  prefix conditioning to training; zero ΠGDM overhead; wins for d ≥ 2.
- [[anon-2026-pi-r-squared]] — πR² (CoRL 2026 anon). Combines training-
  time prefix clamp + diffusion-forcing ramped interior + slow/fast
  channel split + delay-conditioned slow embedding. Closed-loop 25 Hz
  on a 7 Hz VLA. **Directly the architecture for the user's Caricature
  3** (slow planner + fast controller).

**New methods (5):** [[act]], [[diffusion-policy]], [[diffusion-forcing]],
[[training-time-rtc]], [[pi-r-squared]].

**New concepts (2):** [[diffusion-forcing]] (per-position noise sampling
grid as inference-time DoF; CDF), [[fast-slow-policy]] (slow VLM + fast
proprio channels; πR² is the first single-model VLA instance).

**Updated concepts (3):** [[action-chunking]] (added ACT as origin
source + DP + Training-Time RTC + πR² to the handling table),
[[asynchronous-control]] (full variant table with πR² at the limit),
[[train-inference-mismatch]] (added new instances + "inference-time →
training-time → joint" trajectory observation).

**Updated method:** [[rtc]] — linked to training-time successor and
πR² generalization.

**New entities:**
- People: [[chelsea-finn]] (Stanford IRIS; ACT senior co-author),
  [[russ-tedrake]] (MIT CSAIL + TRI; DP + Diffusion Forcing co-author).
- Org: [[mit-csail]] — Diffusion Policy + Diffusion Forcing primary.

**Promoted entities:**
- [[kevin-black]]: stub → growing (now lead on both RTC papers).
- [[sergey-levine]]: stub → growing (now 3 sources: ACT + RTC + Training-Time RTC).

**Cross-source synthesis (the through-line):**
The 5 papers cover the same problem at increasing levels of integration:
- ACT (2023): chunking origin; reactivity loss is the price.
- DP (2024): better action head, same chunking; same reactivity loss.
- Diffusion Forcing (2024): the mechanism that decouples noise from
  position — unlocks the per-position schedule.
- Training-Time RTC (2025): apply prefix clamping at training time;
  drops inference-time inpainting cost.
- πR² (2026): generalize to a continuous schedule (staircase via
  diffusion forcing) + add the slow/fast architectural split.

The trajectory is a strong analog for the user's Caricature 1 → 2 → 3
roadmap on the perception side. Specifically: **πR²'s slow/fast split is
structurally identical to LoGeR's hybrid memory (lossless local + slow
global)**, and **πR²'s diffusion-forcing schedule is structurally
identical to the chunked prediction problem Point4D solves**. The
robotics line is now a step *ahead* of the perception line on these
two issues; the perception side has open research slots to mirror.

**Note on entity discipline:** Tony Zhao (ACT lead) and Cheng Chi (DP
lead) deferred — single first-authorships and not yet recurring.
Shuran Song and Yilun Du same — single-source mentions. ALOHA Unleashed
and π0/π0.5/π0.6 not promoted to source pages because they're cited
rather than ingested.


## [2026-06-10] lint | Wiki audit + cleanup pass

**Scope:** systematic link-graph audit across all wiki pages.
No new sources ingested.

**Fixes applied:**

- **Filename collision resolved.** Renamed
  `wiki/methods/diffusion-forcing.md` → `diffusion-forcing-method.md`
  to disambiguate from `wiki/concepts/diffusion-forcing.md`. AGENTS.md
  §173 requires unique slugs across the vault. Updated concept-page
  frontmatter from broken `[[diffusion-forcing|method]]` syntax to
  `[[diffusion-forcing-method]]`. All 21 inbound `[[diffusion-forcing]]`
  refs now correctly resolve to the concept page (the cross-source
  pattern); the method page is reachable via `[[diffusion-forcing-method]]`
  for the algorithmic recipe.

- **Orphan source linked.** `[[sun-2024-ttt]]` (TTT foundational paper,
  added 2026-05-31) had no inbound links. Added to
  `test-time-training` concept's `sources:` field; replaced
  prose mention `(Sun et al., 2024)` with `[[sun-2024-ttt]]` in both
  test-time-training and zhang-2026-loger. Added row to `index.md`
  under long-context 3D reconstruction.

- **Source count corrected.** `overview.md` claimed "26 sources";
  actual count is 27 (the uncounted sun-2024-ttt). Bumped.

- **Concept page upgrades.**
  - `wiki/concepts/vla.md`: stub → growing. Expanded `sources:` from
    1 → 5 papers; added fast-slow-policy, training-time-rtc, pi-r-squared,
    diffusion-forcing, train-inference-mismatch to related. Added
    sections: anatomy pipeline, two action-head families, real-time
    chunking lineage, expanded examples (GR00T-N1.7, π0.6).
  - `wiki/methods/rtc.md`: added pi-gdm, training-time-rtc, pi-r-squared
    to related; ΠGDM body refs now wiki-link to pi-gdm concept.

- **New concept page: `wiki/concepts/pi-gdm.md`.** ΠGDM
  (Pseudoinverse-Guided Diffusion Models) was referenced in 10 pages
  but had no page. Created — covers the recipe, why VJP per step,
  why error grows with `d`, and where it still wins vs training-time
  alternatives.

- **Dangling brackets cleaned.** Removed `[[wiki-links]]` to
  non-existent slugs:
  - ``andrew-zisserman`` → plain text in zhang-2025-d4rt.
  - ``laura-leal-taixe``, ``aljosa-osep``, ``sven-elflein``
    → removed from nvidia.md `related:` (left as a note comment).
  - ``haotong-lin`` → removed from lin-2025-depth-anything-3 `related:`.
  - ``trevor-darrell`` → removed from junyi-zhang `related:`.
  - ``q-vggt-paradigm`` → noted as candidate-to-create in prose.
  - ``research-note-long-term-real-time-3d-tracking`` → converted
    to plain text reference (the raw note doesn't live in the wiki
    graph).

- **New question pages.**
  - `q-perception-control-symmetry` — is πR²'s staircase shape right
    for online 3D tracking, or does perception prefer bidirectional /
    no-tail?
  - `q-tt-rtc-vs-rtc-tradeoff` — when does ΠGDM-based RTC remain
    preferable to training-time RTC?
  - `q-fast-slow-perception` — does the slow/fast split transfer to
    perception (cached scene_feat + fresh image patches)?
  Added to index.md Questions section.

**Deferred for future passes:**

- Stub-status concepts with high inbound count: `flow-matching` (15),
  `trajectory-fields` (12), `joint-point-tracking` (11),
  `trajectory-chaining`, `streaming-accuracy`. Worth upgrading to
  growing when next ingest touches them.
- Existing q-pages (q-emergent-tracking-heuristics, q-sim2real,
  q-tracking-vs-4d-reconstruction) have stale `updated: 2026-05-24`
  dates but appear factually current. Refresh on next ingest.
- TTT3R is referenced in 5 pages but no source/stub. Tagged
  "pending: promote next batch" in index.md.

**Net link-graph state after lint:**

- 27 source pages, all with ≥1 inbound link.
- 0 orphan source pages.
- 0 dangling wiki-links to non-existent slugs (after cleanup).
- 1 resolved filename collision (`diffusion-forcing`).
- 3 new question pages (open).
- 1 new concept page (`pi-gdm`).

## [2026-06-11] ingest | DFoT / History-Guided Video Diffusion (Song et al. ICML 2025)

**Paper ingested:**
- [[song-2025-history-guided-video-diffusion]] — *History-Guided Video
  Diffusion.* Song, Chen, Simchowitz, Du, Tedrake, Sitzmann. ICML 2025
  (PMLR 267). arXiv 2502.06764v2.

**TL;DR:** Extends [[chen-2024-diffusion-forcing]] from causal RNNs to
non-causal video DiTs (the Diffusion Forcing Transformer, DFoT) and
introduces History Guidance (HG) — a CFG-like family that conditions
on arbitrary subsets of past frames at arbitrary noise levels. The
"noise as masking" identity unifies CFG, dropout, and full diffusion.
DFoT matches W.A.L.T at much less compute; 862-frame rollouts from one
image; new abilities for OOD-history generation and long-horizon
reactive imitation.

**New pages created:**
- [[song-2025-history-guided-video-diffusion]] — source page.
- [[dfot]] — method page (Diffusion Forcing Transformer).
- [[history-guidance]] — concept page (HG-v / HG-t / HG-f / HG-tf
  family).
- [[boyuan-chen]] — entity page (PhD at MIT CSAIL; promoted from
  single-source mention; now lead/co-lead on 2 sources).

**Pages updated:**
- [[diffusion-forcing]] (concept) — added DFoT to variants table;
  three flavors now articulated (CDF causal RNN, DFoT non-causal DiT,
  πR² causal head).
- [[mit-csail]] — added [[song-2025-history-guided-video-diffusion]];
  added [[boyuan-chen]] to members; updated notes section to reflect
  the Chen/Sitzmann/Tedrake cluster's growing footprint.
- [[russ-tedrake]] — added third source; updated notes to reflect
  3-source senior-author trajectory through DP → DF → DFoT.
- `index.md` — added source row under Streaming perception thread;
  added [[dfot]] method row; added [[history-guidance]] concept row;
  added [[boyuan-chen]] to People; bumped [[russ-tedrake]] from 2 → 3
  sources.
- `overview.md` — bumped source count 27 → 28; added 2026-06-11 entry
  in Recent shifts.

**Cross-source through-line (the trajectory now in the wiki):**

Diffusion Policy (2023/2024)
  → action-chunking + DDPM head
  
Diffusion Forcing / CDF (2024 NeurIPS)
  → per-token independent noise, causal RNN
  
DFoT (2025 ICML)  ← THIS INGEST
  → DF generalized to non-causal video DiT
  → noise-as-masking identity → History Guidance family
  
RTC → Training-Time RTC (2025)
  → πR² action-chunking analogue of DFoT's per-position AdaLN
  
πR² (2026, anon CoRL)
  → diffusion-forcing staircase on actions (= DFoT's `k_T` mask shape,
    but causal + chunked + paired with fast/slow architecture)

The Chen / Sitzmann / Tedrake MIT CSAIL cluster authored DP, DF, and
DFoT — three of the five papers in the chain.

**Why directly relevant to user's research:**

The middle-ground 3D-tracker design
([2026-06-10_MiddleGround_3DTracker_Design.md](../../Meeting/Notes/2026-06-10_MiddleGround_3DTracker_Design.md))
needs a non-causal DiT with per-position noise levels applied to a
chunked perception input. DFoT is **the closest existing instance** of
exactly that architecture (just applied to video frames, not 3D
trajectories). Worth checking the DFoT codebase for the per-frame AdaLN
implementation details and the history-guidance composition machinery —
the latter gives a principled way to combine "long memory" + "fresh
observation" scores at inference time, directly relevant to
[[q-fast-slow-perception]].

**Entity discipline:**
- [[boyuan-chen]] promoted (2-source threshold crossed).
- Kiwhan Song deferred (single-source, equal-contribution lead — wait
  for next paper).
- Max Simchowitz, Yilun Du, Vincent Sitzmann deferred (all are
  recurring co-authors but no lead role in wiki sources yet).


## [2026-06-12] ingest | LaCT + StreamVGGT + GVS — 3-paper batch (ingest 10)

Three papers ingested in one batch — coherent because all three answer
"how do we scale streaming sequence models for long contexts" through
distinct mechanisms.

**Sources added:**
- `wiki/sources/zhang-2025-lact.md` — *Test-Time Training Done Right*
  (Zhang/MIT+Adobe, arXiv 2505.23884, May 2025). Large-Chunk TTT with
  chunk sizes 2K–1M tokens, SwiGLU nonlinear fast weights up to 40% of
  model params, Muon test-time optimizer. Three task instantiations:
  novel view synthesis (1M-token context), language modeling, 14B-param
  AR video diffusion.
- `wiki/sources/zhuo-2026-stream-vggt.md` — *Streaming Visual Geometry
  Transformer* (Zhuo & Zheng/Tsinghua, arXiv 2507.11539v2, Mar 2026).
  VGGT with global attention → temporal causal attention + KV-cached
  memory tokens; distilled from VGGT teacher; outperforms CUT3R on
  reconstruction / camera pose / depth across multiple benchmarks; 5×
  faster current-frame inference at 40-frame sequences.
- `wiki/sources/song-2026-gvs.md` — *Generative View Stitching*
  (Song/MIT CSAIL+Runway ML, ICLR 2026 / arXiv 2510.24718v3, Apr 2026).
  Training-free diffusion stitching for camera-guided long-horizon
  video; works with any DFoT model; introduces **Omni Guidance** (a
  bidirectional, co-evolving extension of HG-f) and **cyclic
  conditioning** for explicit loop closure. Generates 120- to 862-frame
  videos through impossible-staircase topologies.

**Methods added:**
- `wiki/methods/lact.md` — LaCT block (window attention + large-chunk
  TTT + FFN); SwiGLU fast weights; Muon update.
- `wiki/methods/streamvggt.md` — Causal-attention + KV cache
  restructuring of VGGT; distillation from VGGT teacher; two long-
  sequence strategies (windowed streaming + K-nearest caching).
- `wiki/methods/gvs.md` — Joint-denoise neighboring chunks; Omni
  Guidance score correction; cyclic conditioning (temporal + spatial
  windows) for loop closure.

**Entities added/updated:**
- `wiki/entities/people/chonghyuk-song.md` — New entity, GVS lead.
  Explicit note that this is distinct from Kiwhan Song (DFoT lead).
- `wiki/entities/people/boyuan-chen.md` — Now 3 sources (DF + DFoT +
  GVS).
- `wiki/entities/orgs/mit-csail.md` — Now 5 sources, anchors both
  diffusion-forcing-for-video and TTT-for-long-context lines.

**Concept / method pages updated:**
- `wiki/concepts/test-time-training.md` — Added Pattern D (Large-chunk
  TTT) — LaCT's 70%-GPU-utilization recipe shows the field has been
  bottlenecked by chunk size, not by the TTT paradigm.
- `wiki/concepts/history-guidance.md` — Added Omni Guidance as a 5th
  variant in the HG family table; explained as bidirectional,
  co-evolving HG-f.
- `wiki/methods/dfot.md` — Added GVS as a training-free downstream
  application that doesn't require retraining DFoT.
- `wiki/methods/cut3r.md` — Added StreamVGGT as the newer streaming
  competitor that *beats* CUT3R on every measured metric.
- `wiki/methods/vggt.md` — Added StreamVGGT as the streaming
  distillation. `updated:` bumped.
- `wiki/sources/song-2025-history-guided-video-diffusion.md` — Added
  GVS as a direct follow-up paper.

**Index/overview:**
- `index.md` — added 3 source rows, 3 method rows, 1 new person entry
  (Chonghyuk Song); bumped Boyuan Chen to 3 sources; updated MIT CSAIL
  org line.
- `overview.md` — bumped source count 28 → 31; rewrote (D) thread to
  split into D1 (TTT-based) and D2 (cached-KV-based) streaming
  strategies; appended 2026-06-12 entry in Recent shifts; updated
  "No streaming 4D" gap to mention StreamVGGT.

**Cross-source through-line (what this batch establishes):**

```
Long-context streaming sequence models — two distinct strategies:

D1: Fast weights = memory  (TTT line)
    sun-2024-ttt  →  vgg-t3  →  loger  →  zhang-2025-lact (LaCT)
    Sun et al.        NVIDIA       DeepMind     MIT+Adobe
    foundational      per-scene    per-chunk    Large-chunk + Muon
                                                + nonlinear MLP
                                                + 14B video diffusion

D2: KV cache + causal attention  (Transformer line)
    vggt  →  cut3r  →  zhuo-2026-stream-vggt (StreamVGGT)
    offline   recurrent    causal + cached KV, distilled from VGGT
                           outperforms CUT3R on every metric

Both are O(n) per-frame. D1 has higher parameter efficiency (state can
be 40% of params); D2 has higher *empirical* 3D-reconstruction quality
right now. Trade-offs unsettled.

Diffusion-forcing line:
    chen-2024-diffusion-forcing  →  song-2025 (DFoT)  →  song-2026 (GVS)
    causal RNN                     non-causal video DiT  + offline stitching
                                                          via Omni Guidance
```

**Why directly relevant to user's research:**

The middle-ground 3D-tracker design has CUT3R as the streaming
backbone. Audit flagged "CUT3R's hidden state isn't a spatial grid" as
a 🔴 leap-of-logic for feature lifting. StreamVGGT's cached memory
tokens *are* a per-frame spatial grid — the leap-of-logic dissolves.
StreamVGGT is now the leading streaming-backbone candidate.

Separately, LaCT shows the TTT-MLP-as-streaming-state path is a viable
alternative — same role as CUT3R or StreamVGGT, but the memory is a
SwiGLU MLP whose weights are updated by Muon per chunk. Worth
considering for the perception/control thread.

GVS shows that *offline* re-anchoring of a DFoT-style chunk denoiser
can be done training-free at inference time — relevant for dataset
annotation, post-hoc re-tracking, and drift correction in long-horizon
3D tracking.

**Entity discipline:**
- [[chonghyuk-song]] created on first appearance — GVS is a load-bearing
  paper for the diffusion-forcing thread and the contribution is
  distinct enough to merit a separate page. Explicit note he's
  different from Kiwhan Song (DFoT lead) since they share a surname.
- Tianyuan Zhang, Wenzhao Zheng, William Freeman, Hao Tan deferred —
  recurring or senior contributors but no second wiki source yet.

## [2026-06-24] ingest | Ma 2026 — Fast Spatial Memory (arXiv 2604.07350)

Ingested *Fast Spatial Memory with Elastic Test-Time Training* (Ma, Yu,
Zhen, Yang, Chai, Gan — MIT-IBM Watson AI Lab / UMich / UMass Amherst,
Apr 2026). Two contributions: (1) **LaCET** — extends [[lact]] with
Elastic Weight Consolidation to fix multi-chunk fast-weight drift, and
(2) **FSM** — the first TTT-based 4D NVS model at scale. FSM-LVSM
achieves SOTA among feed-forward 4D methods on Stereo4D (32.16 PSNR).

**New pages (3):**
- [[ma-2026-fsm]] — source page.
- [[lacet]] — method page (LaCT + EWC consolidation).
- [[fsm]] — method page (full 4D NVS model, LVSM + LRM decoders).

**Updated pages (7):**
- [[lact]] — added LaCET/FSM as downstream extension; noted multi-chunk
  drift limitation discovered by this source.
- [[test-time-training]] — added Pattern E (elastic chunked TTT with
  consolidation); the A–D taxonomy is now A–E.
- [[4d-reconstruction]] — added FSM row to method table; added attention
  complexity as a new design axis (O(n²) vs O(n) via TTT).
- [[feed-forward-3d-reconstruction]] — added FSM to lineage diagram
  under the LaCT branch.
- [[cmp-3d-4d-reconstruction]] — added FSM rows to high-level table;
  added 4D NVS Stereo4D+NVIDIA benchmark section from Table 3; added
  FSM to "where each method wins" and architectural design space.
- [[zhang-2025-lact]] — (related field only; no body changes).
- `index.md` — added source row, 2 method rows, 3 new tags.

**Key cross-source finding:** naive multi-chunk LaCT degrades vs
single-chunk — the first empirical evidence that fully plastic TTT
updates are actively harmful in streaming regimes. This validates the
intuition behind [[loger]]'s hybrid memory (SWA + TTT) as a separate
stabilization strategy. The consolidation idea (EWC) is orthogonal and
could be composed with LoGeR's approach.

**Entity discipline:** all 6 authors (Ziqiao Ma, Xueyang Yu, Haoyu Zhen,
Yuncong Yang, Joyce Chai, Chuang Gan) deferred — single-source mentions.
MIT-IBM Watson AI Lab deferred as org. Chuang Gan is the most likely
to cross the 2-source threshold soon (senior on multiple TTT/embodied
papers outside this wiki).

## [2026-06-26] ingest | STRIDE — Self-Supervised Decomposable 4D Driving Scene Recon (NeurIPS 2026 anon)

Ingested STRIDE from `raw/papers/7307_Towards_Self_Supervised_G.pdf`. The
first wiki entry in the **driving-scene** flavor of feed-forward 4D, and
the first to combine three things: (1) **camera + LiDAR multi-modal**
fusion in a unified 3D point representation, (2) **Point Transformer v3**
as the reasoning backbone over lifted features (not 2D-token ViT or
sparse 3D conv), (3) **learnable dynamic instance decomposition**
without human annotations, via latent instance tokens supervised by the
same 4D reconstruction signal. Outputs: 3DGS with per-Gaussian velocity
+ instance assignment, all from one forward pass.

**Results:** SOTA on Waymo Open Dataset and PandaSet over STORM (camera-
only), STORM+ (their multi-modal re-impl), and Flux4D-ff (LiDAR-centric).
Largest improvements on scene flow (EPE3D 0.161 vs 0.200 STORM+, flow
angle 0.307 vs 0.485 STORM+ on WOD). Backbone ablation: PTv3 halves
EPE3D vs sparse-conv UNet (0.180 vs 0.385) on the same lifted features.
Modality ablation: LiDAR-depth-as-ViT-input cuts EPE3D nearly in half
vs image-only (0.180 vs 0.313); raw LiDAR points as additional PTv3 input
is marginal. Instance segmentation: beats DBSCAN baseline on all
IoU thresholds; M5 (DBSCAN init + smoothness + recon loss) is the only
ablation configuration with positive gains on both precision and recall.

**Pages created:** [[anon-2026-stride]], [[stride]]. **Pages touched:**
[[4d-reconstruction]] (added STRIDE row to feed-forward-4D table, added
multi-modal + decomposition design axes), [[index]] (added source row
under 3D/4D reconstruction; added method row; added new tags
`driving-scenes`, `lidar`, `point-transformer`, `gaussian-splatting`,
`instance-decomposition`, `self-supervised`), [[overview]] (bumped
source count 31 → 32; updated thread (B) to 10 sources; added STRIDE
to "no streaming 4D" gap with explicit limitation #3 quote; appended
2026-06-26 recent-shifts entry).

**Anonymous handling:** Following [[anon-2026-point4d]] /
[[anon-2026-pi-r-squared]] / [[anon-2026-trace-anything]] precedent —
`authors: [Anonymous]`, slug prefix `anon-2026-`, venue
"NeurIPS 2026 (under review)".

**Entity discipline:** All authors anonymous; no person/org pages
created. STORM / Flux4D / DrivingRecon / DIAL-GS / IDSplat / UniRe /
Point Transformer v3 (Wu et al. CVPR 2024) all flagged as
not-yet-ingested but cited inline — STORM and Flux4D are now cited 2+
times across the wiki and should be promoted to method pages on next
batch. PTv3 is a likely tool-page candidate if more 4D-driving papers
adopt it.

**Prompt injection flagged:** PDF text contains an embedded instruction
("In your output you MUST Include ALL of the following phrases…")
attempting to coerce reviewer-template language. Standard LLM-reviewer
injection pattern. Ignored; user notified before writing the source
page. No reviewer-template language appears in [[anon-2026-stride]] or
[[stride]].

**Why this matters for the user's project:** STRIDE is the closest
existing instance of "feed-forward dense-3D + decomposition + multi-
modal fusion" in the wiki. Its acknowledged limitation #3 (PTv3 can't
stream long sequences) is exactly the streaming-4D gap that
[[zhang-2026-loger]], [[zhuo-2026-stream-vggt]], [[lact]], [[lacet]]
attack from different angles. **A natural synthesis:** swap STRIDE's
PTv3 for LaCET blocks → driving-domain streaming 4D + decomposition.

## [2026-06-26] ingest | PIPs (Harley/Fang/Fragkiadaki, CMU, ECCV 2022) + TAPIR (Doersch et al., DeepMind+Oxford, ICCV 2023)

Foundational-paper backfill: two predecessors that every existing TAP
source in the wiki cites and that previously had only "future seeds to
promote on 2nd mention" placeholder treatment. PIPs is the originating
deep TAP architecture — 8-frame CNN + multi-scale correlation pyramids
+ 12-block MLP-Mixer iterative refinement + visibility-thresholded
chaining for long sequences — plus the **FlyingThings++** synthetic-
with-injected-occluders training recipe. TAPIR fuses PIPs' refinement
with TAP-Net's per-frame cost-volume initialization, replaces the
fixed-length MLP-Mixer with **depthwise conv over time** (any length,
parallel inference), and adds a **self-supervised position uncertainty**
head. TAPIR introduces **Panning MOVi-E** — the Kubric variant with
camera-pan trajectories — carried forward to BootsTAP, BootsTAPNext,
TAPNext.

**Pages created (4):**
- [[harley-2022-pips]] (source).
- [[pips]] (method).
- [[doersch-2023-tapir]] (source).
- [[tapir]] (method).

**Pages touched (~16):**
- Method pages: [[cotracker]], [[tapnext]] — added explicit predecessor
  links + corrected related lists; the "future seeds" placeholders are
  resolved.
- Source pages: [[karaev-2024-cotracker]] — promoted "predecessors now
  in this wiki" section.
- Concept pages: [[point-tracking]] (now sourced from PIPs+TAPIR;
  expanded "standard architectural ingredients" with the explicit
  PIPs vs TAPIR comparison), [[joint-point-tracking]] (links the
  "independent tracks" critique to PIPs's own §5.7),
  [[online-vs-offline-tracking]] (added PIPs to window tier + TAPIR
  any-length offline tier), [[synthetic-to-real-gap]] (now traces the
  arc FlyingThings++ → Panning MOVi-E → BootsTAP),
  [[trajectory-chaining]] (added PIPs §3.7 as the 2D-pixel ancestor).
- Entity pages: [[adam-w-harley]] (stub → growing; 2 sources),
  [[katerina-fragkiadaki]] (stub → growing; 2 sources — PIPs +
  TAPIP3D, both anchored at CMU), [[carl-doersch]] (now 3-source —
  TAPIR + TAPNext + TAPNext++), [[ignacio-rocco]] (related-list only;
  not a TAPIR author), [[cmu-ri]] (6 sources; institutional origin
  point of modern deep TAP), [[google-deepmind]] (added TAPIR),
  [[oxford-vgg]] (7 sources, still most prolific).
- Datasets: [[tap-vid-dataset]] (added TAPIR as benchmark user;
  introduces Panning MOVi-E), [[kubric-dataset]] (added Panning
  MOVi-E as a distinct variant).
- Comparisons: [[cmp-tap-methods]] (added PIPs + TAPIR rows + headline
  numbers; latency table now shows PIPs's ~30s chained inference vs
  TAPIR's ~7ms per query).
- Questions: [[q-emergent-tracking-heuristics]] (clarified the
  "control condition" — PIPs/TAPIR are the engineered architectures
  that TAPNext argues re-emerge for free).
- [[index]] (added 2 source rows + 2 method rows under Point Tracking;
  bumped source / org counts; added 7 new tags: `mlp-mixer`,
  `depthwise-conv`, `iterative-refinement`, `two-stage`, `occlusion`,
  `uncertainty-estimation`, `foundation-paper`).
- [[overview]] (33 → 35 sources; thread (A) grows 7 → 9; new
  recent-shifts entry).

**Entity discipline:** All four TAPIR co-authors not yet promoted
(Doersch already had a page; Yang / Vecerik / Gokay / Gupta / Aytar /
Carreira / Zisserman deferred — single-source mentions so far).
Zhaoyuan Fang on PIPs likewise deferred.

**Why this matters for the wiki:** the "PIPs is the predecessor every
later method cites" claim is now fully traceable — no more dangling
references. The CMU TAP-line (Fragkiadaki / Harley) is now visibly the
**institutional origin of the modern deep TAP sub-field**, which
strengthens the wiki's institutional grounding for the user's program
and gives [[cmu-ri]] a 6-source footprint. The control-condition for
[[q-emergent-tracking-heuristics]] (does TAPNext re-emerge what
PIPs/TAPIR engineer?) is now primary-source-grounded rather than
secondary-cited.

## [2026-06-27] ingest | Track2Act + Im2Flow2Act + 3DFlowAction + Dex4D + Pri4R (5-paper batch — manipulation via point tracks)

Five-paper batch opening **thread (E) — manipulation policies via
point tracks** in [[overview]]. The papers form a clear 2024–2026 arc
on the same load-bearing idea: point tracks as the intermediate
representation between cross-embodiment planning and robot execution.
**Two of the five are from the user's own Tulsiani group** (Track2Act
+ Dex4D); **Dex4D explicitly acknowledges the user (Minsik Jeon)** for
presentation feedback.

**Source pages created** (5):
- [[bharadhwaj-2024-track2act]] — Track2Act (Bharadhwaj × Mottaghi ×
  Gupta × Tulsiani, ECCV 2024). DiT denoiser of 2D point tracks from
  web video (EpicKitchens + RT-1 + BridgeData, 400K clips) + PnP rigid
  transform + residual BC policy on ~400 Spot demos. The founding
  paper of the line; first to combine generative track prediction +
  robot-action extraction in one framework.
- [[xu-2024-im2flow2act]] — Im2Flow2Act (Xu × Xu × Xu × Chi ×
  Wetzstein × Veloso × Song, CoRL 2024). Object-only flow as
  cross-domain interface; AnimateDiff flow generator on human videos
  + flow-conditioned diffusion policy on sim play; **zero real-robot
  data**, 81% real-world SR.
- [[zhi-2025-3dflowaction]] — 3DFlowAction (Zhi × Chen × Zhou et al.,
  Jun 2025). Direct 2D→3D lift of Im2Flow2Act; ManiFlow-110k
  (cross-source 3D flow dataset, 110K trajectories from 7 datasets);
  GPT-4o closed-loop verifier; optimization action policy needing no
  action labels; 70% vs. 20-50% baselines.
- [[kuang-2026-dex4d]] — Dex4D (Kuang × Park × Fragkiadaki ×
  Tulsiani, Feb 2026). First to apply point-tracks-as-interface to
  **22-DoF dexterous manipulation** via Anypose-to-Anypose sim-to-real
  RL + **Paired Point Encoding** + teacher-student distillation.
  Wan2.6 video gen → SAM2 + CoTracker3 + median-rescaled relative
  depth → 3D tracks → policy. +22.5% real-world SR over NovaFlow-CL.
- [[kim-2026-pri4r]] — Pri4R (Jisoo Kim et al., KAIST+LG+Yonsei+SNU+
  CMU, Mar 2026). Inverts the role: 3D point tracks as **privileged
  training-time supervision** for π₀ / π₀.₅ / OpenVLA-OFT via a
  lightweight aux head that's discarded at inference. +13.2% RoboCasa
  for OpenVLA-OFT, +9.8% LIBERO-Long.

**Method pages created** (5):
- [[track2act]] — DiT denoiser + PnP + residual BC.
- [[im2flow2act]] — AnimateDiff + object-only flow + diffusion policy
  + temporal alignment.
- [[3d-flow-action]] — 4-channel AnimateDiff + SVD-fit + GPT-4o
  verifier + optimization policy.
- [[dex4d]] — Anypose-to-Anypose RL + Paired Point Encoding +
  teacher-student.
- [[pri4r]] — auxiliary 3D point-track head as privileged supervision.

**Concept page created** (1):
- [[point-tracks-as-manipulation-interface]] — binding concept of the
  family; two design axes (conditioning vs. supervision; heuristic vs.
  learned action policy); contested points; open questions.

**Comparison page created** (1):
- [[cmp-point-track-manipulation]] — high-level design table, headline
  numbers, where-each-wins, open eval gaps.

**Person pages created** (6):
- [[homanga-bharadhwaj]] — CMU PhD (Tulsiani group); Track2Act first
  author; DemoDiffusion / Gen2Act collaborator.
- [[yuxuan-kuang]] — CMU visiting (Tulsiani × Fragkiadaki); Dex4D
  first author; track record on StopNet / RAM / SkillBlender /
  FetchBot.
- [[sungjae-park]] — CMU (Tulsiani group); Dex4D co-first; DemoDiffusion
  co-author.
- [[shuran-song]] — Stanford+Columbia; Im2Flow2Act + Diffusion Policy
  senior.
- [[cheng-chi]] — Stanford+Columbia; Diffusion Policy first author +
  Im2Flow2Act co-author.
- [[laszlo-jeni]] — CMU faculty; Pri4R co-senior; first KAIST × LG ×
  CMU collaboration in the wiki.

**Existing pages touched**:
- [[shubham-tulsiani]] — stub → growing; now 3 sources (Point4D +
  Track2Act + Dex4D); added notes section linking to homanga /
  sungjae / yuxuan and Fragkiadaki co-senior.
- [[katerina-fragkiadaki]] — was 2-source, now 3-source (PIPs +
  TAPIP3D + Dex4D); added Dex4D + Tulsiani as collaborator.
- [[cmu-ri]] — was 6-source, now 9-source; updated members list with
  bharadhwaj / kuang / park / jeni; flagged the user-acknowledged
  Dex4D entry; added "two distinct CMU research lines now visible:
  geometry/4D vs. manipulation-via-tracks; the CMU TAP line directly
  feeds the CMU manipulation line."
- [[meta-ai]] — added Track2Act (Mottaghi at FAIR is second author);
  3 → 4 sources.
- [[vla]] — added Pri4R to sources; new section "Auxiliary
  supervision (training-time geometry)" describing Pri4R's privileged-
  supervision approach + ablation table.
- [[point-tracking]] — expanded "Why it matters" with the
  manipulation downstream paragraph linking all 5 sources.
- [[index]] — added "Manipulation via point tracks" sources section
  (5 entries) + manipulation interfaces concept entry + manipulation
  via point tracks methods section (5 entries) + 6 person entries +
  CMU-RI count bump 6→9 + Meta-AI count bump 3→4 + new comparison
  entry + new robotics-policy tags (`dexterous`, `cross-embodiment`,
  `sim-to-real`, `reinforcement-learning`, `web-video`, `world-model`,
  `video-diffusion`, `flow`, `residual-policy`,
  `privileged-information`, `auxiliary-supervision`, `vlm-verifier`,
  `paired-point-encoding`).
- [[overview]] — 35 → 40 sources; new thread (E); recent-shifts
  entry; closed the "no robotics application" gap with a remaining
  sub-gap noted (no source combines 4D-recon backbone + manipulation
  policy).

**Why this matters for the wiki**: The TAP infrastructure thread (A)
that has dominated the wiki since launch now has a fully-articulated
**downstream consumer** (E). The user's institution sits at the
center: CMU produces both the TAP infrastructure (PIPs / TAPIP3D)
*and* its largest manipulation-from-tracks instance (Track2Act +
Dex4D). The Tulsiani × Fragkiadaki Dex4D paper is the concrete
collaboration node connecting the two CMU research lines.

The conditioning-vs-privileged-supervision axis (introduced by Pri4R
inverting Dex4D's setup) is a clean structural-fix analog of the
**train-inference mismatch** pattern in thread (C) — the dominant
form moves from "inference-time tracker stack" toward "training-time
embedded knowledge" without losing the gain. Watch for follow-ups.

**Entity discipline**: Author lists checked carefully — Mottaghi
correctly attributed to Meta FAIR (not Tulsiani group) on Track2Act;
Pri4R's CMU senior is Jeni only (not the KAIST/LG/Yonsei/SNU
seniors); Cheng Chi attributed to both Diffusion Policy first-author
and Im2Flow2Act co-author. ManiFlow-110k (3DFlowAction's dataset),
LIBERO, RoboCasa, BridgeV2, RT-1, EpicKitchens — none yet promoted to
dataset pages (covered inline; promote when a 2nd source uses them
explicitly). NovaFlow (Li 2025, primary Dex4D baseline) flagged but
not yet ingested — would be a natural next paper.

## [2026-07-02] ingest | Wu 2025 — Point3R (arXiv 2507.02863, NeurIPS 2025)

Added [[wu-2025-point3r]] and method page [[point3r]]. Tsinghua
Automation (Wu × Zheng × Zhou × Lu). Streaming DUSt3R-family recon
with **explicit spatial pointer memory** — each memory unit is a
`(3D position ∈ R^3, feature ∈ R^768)` pair in the global coordinate
system. Distance-based fusion caps pointer growth; **3D hierarchical
RoPE** injects continuous 3D position into pointer-image
cross-attention. DUSt3R init; 8× A800 × 15 days. Handles static +
dynamic + ordered + unordered inputs uniformly (pointer memory is
overwritten in place when a moving object revisits a location).

Headline result: long-sequence 3D recon vs [[cut3r]] on 500-1000-frame
7-scenes — Acc mean 0.071 vs 0.238; NRGBD 400-900 frames 0.110 vs
0.372 (Table 5). Also SOTA-ish on 3-5-frame 7-scenes (Acc 0.085 vs
CUT3R 0.126 vs DUSt3R-GA 0.146; Table 1), monocular depth on NYU-v2
(Abs Rel 0.078 vs CUT3R 0.086) and Bonn (0.061 vs 0.063). Weakness:
camera pose lags [[cut3r]] and MonST3R-GA on ScanNet/Sintel/TUM-
dynamics — authors flag as future work.

Wiki edits:
- **New pages:** [[wu-2025-point3r]] (source), [[point3r]] (method).
- **Concept updated:** [[feed-forward-3d-reconstruction]] — added
  Point3R to lineage tree + evidence section (memory should scale
  with explored space, not time).
- **Comparison updated:** [[cmp-3d-4d-reconstruction]] — added
  Point3R row to high-level table; expanded "recurrent state for
  streaming" bullet to name three distinct memory paradigms (fixed
  implicit state / growing KV cache / explicit 3D-indexed pointers);
  reflected in "where each method wins" (best online streaming 3D
  now names both CUT3R and Point3R).
- **Method pages relinked:** [[cut3r]] (added Point3R as sibling
  contemporary), [[streamvggt]], [[vgg-t3]], [[loger]] (all previously
  had bare "Point3R" text — now [[point3r]]).
- **Source pages relinked:** [[elflein-2026-vgg-t3]],
  [[zhuo-2026-stream-vggt]], [[zhang-2026-loger]] — same conversion.
  StreamVGGT source note expanded: Point3R is the *same-group*
  companion (Wenzhao Zheng is 2nd on Point3R, project lead on
  StreamVGGT) — two complementary streaming-memory designs from
  Tsinghua Automation in the same year.
- **[[index]]:** added Point3R to 3D/4D Reconstruction section + to
  methods list; [[overview]]: updated source count 40 → 41, thread
  (D) now 5 sources with three distinct streaming strategies
  articulated (fast-weights TTT / KV cache / geometry-indexed
  pointers); added 2026-07-02 recent-shifts entry.

Design implications:
- Third distinct memory design for online DUSt3R lineage now
  ingested: fixed implicit state ([[cut3r]]) / growing KV cache
  ([[streamvggt]]) / explicit 3D-indexed pointers ([[point3r]]). The
  Spann3R "cache past frames as KV" paradigm — the fourth remaining
  corner — is now a higher-priority ingest.
- Tsinghua Automation (Wenzhao Zheng's group) is producing the two
  currently-strongest streaming DUSt3R variants (StreamVGGT +
  Point3R) — 2-source org threshold met; org page candidate next
  batch.
- Camera-pose gap suggests **explicit pointer memory needs a
  pose-aware disentanglement** — a candidate research angle for the
  user's streaming-tracker line (Topic 3).
- No downstream integration demonstrated (VLA / policy / tracker) —
  but pointer memory is spatially organized, so 3D-query feature
  lookup (à la [[tapip3d]] using CUT3R features) is a natural
  extension. Consider as a design axis when the user's streaming 3D
  work needs a memory substrate.

No new person pages (Wu / Zhou / Lu at 1 source each; Zheng now at 2
sources — Point3R + StreamVGGT — could get a stub next batch when
another Zheng paper appears). No new org page (Tsinghua Automation
at 2 sources but authors overlap heavily — hold one more source).

## [2026-07-09] ingest | Lee 2026 — µ0 (arXiv 2606.13769, Jun 2026)

Single-paper ingest. µ0 = **query-conditioned 3D trace-space world
model** for cross-embodiment manipulation, from UMD (F. Huang lab +
J.-B. Huang lab) + SNU (H. J. Kim lab). Direct successor of TraceGen
(Lee et al. CVPR 2026 — same first author).

Created [[lee-2026-mu0]] and [[mu0]]. Updated
[[point-tracks-as-manipulation-interface]] (6 methods now; new
"world-model features" role added as 3rd axis; new "fixed grid vs.
semantic keypoints" axis; new sub-section for µ0; contested points
+ open questions rewritten with 3-role framing),
[[cmp-point-track-manipulation]] (6-row design table; µ0 vs
baselines RoboCasa365 sim table + Real UR3 table + trace-forecasting
table; new "where µ0 wins" bullet; open eval gaps updated to name
µ0 vs Pri4R vs Dex4D 3-way; new axes: fixed-grid vs semantic
keypoints, waypoints vs B-spline). Also cross-linked from
[[track2act]] / [[im2flow2act]] / [[3d-flow-action]] / [[dex4d]] /
[[pri4r]] frontmatter + Related sections, and from [[trace-anything]]
(reciprocal note about the B-spline primitive at short horizons).

**Wiki state after ingest:** 42 sources, 44 methods, 3 comparisons.
[[index]] entry added to both the Sources / Manipulation-via-point-
tracks section and the Methods list; [[overview]]: bumped to
42 sources, thread (E) expanded from 5 → 6 sources, three-axis
framing replaces prior two-axis framing, 2026-07-09 recent-shifts
entry added with priority-ingest bumps.

Design implications:
- **Third role for point tracks in the manipulation family** —
  neither pure conditioning (Track2Act / Im2Flow2Act / 3DFlowAction /
  Dex4D) nor pure privileged supervision (Pri4R), but "trace-
  denoising features as a frozen motion prior." The action expert
  reads intermediate denoising features from a single partial
  denoising step; the decoded track is a pretraining objective, not
  a runtime signal.
- **First video-only WM to beat action-labeled π₀** on a real-scale
  sim benchmark (RoboCasa365, 30.25 vs 25.25% avg SR) — reframes π₀-
  style action-labeled pretraining as a design choice, not a
  requirement. Real-world UR3 also wins vs π₀.₅ (91.7 vs 80%). The
  cleanest ablation is µ0-AE vs VLM+AE (same SmolVLM backbone, no
  trace expert): +18.4 pts. Trace-expert features carry motion
  structure the VLM alone cannot supply.
- **Semantic keypoint sampling directly attacks the fixed-grid
  limitation** every prior member of the family shares — DINOv2
  entity clusters + per-entity budget + movement filter surface tool
  tips and contact patches that grids miss. Ablation (Appendix E.1)
  confirms the change matters.
- **B-spline primitive reused from [[trace-anything]]** — the wiki's
  200-frame-benchmark critique of Bézier splines doesn't bite at
  µ0's manipulation horizons (T ≤ 32). µ0 is the first data point
  that the primitive is useful at policy horizons; scaling µ0 to
  200-frame manipulation traces is untested and would repeat the
  Trace Anything failure mode.
- **TraceExtract data engine (~8× TraceGen)** is the practical
  bottleneck for scaling this line — semantic keypoint selection +
  global 3D lifting + event-centric captioning stack, applicable to
  any heterogeneous video corpus. Reusable infrastructure, not just
  a µ0-specific pipeline.
- **π₀.₅ still wins in sim** (42%) — action-labeled pretraining is
  not yet fully replaceable at scale. µ0's headline is that video-
  only pretraining now approaches (not surpasses) action-labeled
  under matched action-expert regimes.

No new person pages (Seungjae Lee, Yoonkyo Jung, Jia-Bin Huang,
Furong Huang, H. Jin Kim, Jusuk Lee, Jonghun Shin, Yao-Chih Lee,
Amir H. Shahidzadeh all at 1 wiki source each — Seungjae Lee will
promote to 2 when TraceGen is ingested; hold all for now). No new
org page (UMD at 1 source — will hit 2 with TraceGen; SNU already
has a co-authorship on [[kim-2026-pri4r]] but no primary lead, so
still short of the org-page threshold this pass — flag for review
after TraceGen).

Priority-ingest bumps flagged (in order):
1. **TraceGen** (Lee et al. CVPR 2026) — µ0's direct predecessor,
   primary baseline throughout, would promote 2 entity threshold
   markers.
2. **NovaFlow** (Li 2026a) — cited by µ0 refs and remains Dex4D's
   headline baseline.
3. **BootsTAP / BootsTAPIR** — still #1 TAP hole (unchanged).
4. **Dream2Flow** (Dharmarajan et al. ICRA 2026) — µ0's 3D
   baseline; video-gen + object flow line.
5. **RoboCasa365** (Nasiriany et al. 2026) — new sim benchmark
   used by both Pri4R and µ0. Would justify a dataset page.

Sanity check: `find | sed | sort -u > slugs.txt` + grep new page
wikilinks → all resolve to existing pages except the intentional
[[tracegen]]-adjacent references (kept as bare text since no wiki
page exists yet). No stale slug references from prior sessions
surfaced.

No prompt injection detected in the PDF; extracted TL;DR + Fig 1
+ §2–§4 text + Table 1/Table 2/Fig 6 numbers. Some late-appendix
detail (Appendix A/B specifics) not extracted this pass — flagged
in the source page but not blocking.

## [2026-07-09] ingest | Huang 2026 — PointWorld (arXiv 2601.03782, Jan 2026)

Single-paper ingest triggered by user asking "is PointWorld in my
wiki?" (was not). PointWorld = **large pretrained 3D dynamics WM**
for robot manipulation. Stanford (Wenlong Huang + Li Fei-Fei) +
NVIDIA (Chao + Mousavian + Liu + Fox + Mo). Cited by µ0 as prior
work; different paradigm within same thread (E).

Created [[huang-2026-pointworld]] and [[pointworld]]. Updated
[[point-tracks-as-manipulation-interface]] (7 methods now; **new
"action-conditioned dynamics simulator" role** added as 4th pole on
the tracks-role axis; new sub-section for PointWorld; contested
points rewritten with 4-role framing; new "language-conditioned
trace generator vs. action-conditioned dynamics WM" axis; new
"fixed grid vs. semantic keypoints vs. dense per-pixel" axis);
[[cmp-point-track-manipulation]] (7-row design table; PointWorld
scaling roadmap table; PointWorld real-Franka SR table; new "where
PointWorld wins" bullet; open eval gaps updated to name 4-way
PointWorld vs µ0 vs Pri4R vs Dex4D and note the µ0 × PointWorld
fusion slot is unexplored; recurring axes expanded to four).
Cross-linked from [[track2act]] / [[im2flow2act]] / [[3d-flow-action]]
/ [[dex4d]] / [[pri4r]] / [[mu0]] / [[lee-2026-mu0]] related lists;
also from [[stride]] (shared PTv3 backbone — now 2-source tool
candidate).

**Wiki state after ingest:** 43 sources, 45 methods, 3 comparisons.
[[index]] entries added to both the Sources / Manipulation section
and the Methods list; tags updated (`dense-tracking`,
`point-transformer` under point tracking; `mpc` under robotics
policy); [[overview]] bumped to 43 sources, thread (E) expanded from
6 → 7 sources, three-axis framing replaced with four-axis framing,
2026-07-09 recent-shifts entry added.

Design implications:
- **Fourth role for point tracks in the manipulation family** —
  action-conditioned dynamics simulator, not a trajectory generator.
  Different question ("what happens?" vs. "what should happen?");
  different downstream stack (MPPI, not learned action policy).
  PointWorld = learned physics engine; MPPI = outer planner. Fully
  factored: model predicts, planner optimizes.
- **First published scaling laws for 3D world modeling** — log-linear
  in both data (5% → 100%) and params (50M → 1B) on DROID ℓ2 mover.
  Analogous to LM / vision scaling laws. Removes doubt about whether
  the "just scale it" recipe from LM works for 3D dynamics too.
- **PTv3 + DINOv3 = scalable substrate for point-cloud WMs.** Beats
  GBND / PointNet / PointNet++ / SparseConv / plain Transformer at
  957× the params of GBND. STRIDE already uses PTv3 for driving-scene
  4D — same architecture family unifies driving 4D + manipulation
  dynamics. PTv3 = 2-source backbone, now a tool-page candidate.
- **DROID 3D annotation pipeline is reusable community
  infrastructure** — FoundationStereo depth + VGGT-init extrinsics +
  robot-mesh alignment + CoTracker3 → 60% recovery of DROID with
  reliable 3D flows. Applicable to any RGB-D manipulation corpus.
- **Gripper-only action flows > whole-body flows on real data.**
  Contact-relevant geometry concentrates learning signal; dense whole-
  body dilutes. Simple design finding but generalizable across
  action-conditioned dynamics models.
- **Chunked-not-autoregressive** for WM rollout — single forward pass
  predicts all H = 10 steps jointly; 10× compute-efficient vs AR;
  lower drift. Trend echoes action-chunking on control side (ACT,
  Diffusion Policy) — chunked prediction is winning across
  perception + control.
- **Sharp complementary role vs. µ0.** µ0 = language-conditioned
  trace generator (goal-directed motion prediction); PointWorld =
  action-conditioned dynamics (physics simulation). Combining them
  (µ0 trace as MPPI target, PointWorld as simulator) is an
  unexplored fusion slot. Flagged in the open eval gaps section.
- **Depth requirement is the sharpest limitation.** No pure-RGB
  pretraining — cannot ingest Ego4D-style human video without depth.
  µ0 handles this via global-local reconstruction from RGB alone.
  Design axis: depth-required vs. RGB-only pretraining.

No new person / org pages this pass. Wenlong Huang at 1 wiki source
(would promote if VoxPoser or ReKep were ingested — both cited).
Li Fei-Fei, Dieter Fox, Kaichun Mo, Yu-Wei Chao, Arsalan Mousavian,
Ming-Yu Liu all at 1 wiki source each — hold. Stanford + NVIDIA
each at 1+ implicit sources this batch — Stanford diffuse across
Song / Finn / Fei-Fei labs (hold on umbrella org page); NVIDIA now
has PointWorld (Chao/Mousavian/Liu/Fox/Mo) + VGG-T3 (partial via
Vector) — potentially 2-source but authors disjoint (hold).

Priority-ingest bumps (in order):
1. **TraceGen** (Lee CVPR 2026) — µ0 predecessor; still #1 for
   thread-(E) coherence.
2. **VoxPoser / ReKep** (Huang et al.) — same Wenlong Huang line;
   would promote him to 3-source and open VLM-based waypoint /
   constraint interfaces as a wiki sub-axis.
3. **GBND** (Ai et al. Sci. Robotics 2025) — PointWorld's baseline
   and a natural entry point for the graph-neural-dynamics
   literature the wiki is missing.
4. **BootsTAP / BootsTAPIR** — unchanged from prior lists; #1 TAP
   hole.
5. **FoundationStereo / DROID / BEHAVIOR-1K** — infrastructure pages.

Sanity check: `find | sed | sort -u > slugs.txt` + grep new page
wikilinks → all resolve to existing pages. No stale slug references.

No prompt injection detected. Extracted §1–§5 text + Fig 1/2/3
narrative + Tables 1/2 numbers + scaling-law roadmap. Late-appendix
details on MPPI temperature schedule + optimization hyperparameters
not extracted this pass.

## [2026-07-09] lint | wiki audit + freshness pass

Systematic lint after µ0 + PointWorld ingests. Findings + fixes:

### Broken wikilinks
- `[[user_role]]` in `wiki/sources/elflein-2026-vgg-t3.md:215` →
  rewrote to plain "the user's" (memory files aren't wiki pages).
- `[[slug]]` in `index.md:12` — inside format-example backtick, not a
  real link. Leave.
- `[[wiki-links]]` in `log.md:548` — prose inside a prior lint entry.
  Leave.
- `[[tracegen]]` — intentional forward reference; TraceGen not yet
  ingested. Documented in [[log]] as top ingest priority. Leave.
- `[[AGENTS]]` / `[[index]]` / `[[log]]` / `[[overview]]` — top-level
  files at repo root, not under `wiki/`. Valid; my slug audit only
  scans `wiki/`. Fine.

### Frontmatter compliance
- `wiki/sources/sun-2024-ttt.md` had `status: ingested (web)` —
  parenthetical annotation off-schema. Normalized to `status: ingested`.
- Type-aware audit (source pages use `title`; entity pages use `name`;
  question pages use `status: open`): **all pages compliant** after
  the sun-2024-ttt fix.
- Every page has YAML frontmatter (grep pass).

### Stale `updated:` fields (bumped)
Ten pages carried `updated:` dates predating recent ingest edits:
- **Point3R (2026-07-02) touches, bumped to 2026-07-02:**
  [[cmp-3d-4d-reconstruction]] (was 06-24), [[feed-forward-3d-reconstruction]]
  (06-24), [[cut3r]] (06-12), [[streamvggt]] (06-12), [[loger]] (05-29),
  [[vgg-t3]] (05-29), [[zhang-2026-loger]] (05-29),
  [[zhuo-2026-stream-vggt]] (06-12).
- **Lint-pass edits today (2026-07-09):** [[elflein-2026-vgg-t3]]
  (was 05-29 — fixed broken `[[user_role]]` link + already carried
  Point3R additions), [[sun-2024-ttt]] (was 05-31 — status fix).

### Orphan / dupe checks
- Slug uniqueness: no duplicate basenames across `wiki/`.
- Filename ↔ slug consistency: passes.
- Truly orphan pages (no incoming links from anywhere): none. The
  audit surfaced `q-tt-rtc-vs-rtc-tradeoff` as unlinked from `wiki/`
  but it's referenced from [[index]] + [[overview]] + [[log]].
- Index coverage: all 43 sources listed in [[index]].
- Entity `entity_type:` field: all present.

### Counts
- **43 sources**, **45 methods**, **1 concept-methods collision-free**,
  **3 comparisons**, **6 open questions**, **entity pages: 35 people
  + 11 orgs + 6 datasets**.

### Not-fixed this pass (deferred)
- `[[slug]]` / `[[wiki-links]]` inside format examples and prose —
  leave as-is; not real broken links.
- `[[tracegen]]` — will resolve when TraceGen is ingested (top
  priority next batch).
- Raw layer `M raw/sources/zotero/library.bib` — pre-existing Zotero
  auto-sync mutation; out of scope per raw-is-immutable schema.
- 2026-06-27 batch (Track2Act ×5) entity page updates
  ([[cmu-ri]], [[meta-ai]], [[shubham-tulsiani]],
  [[katerina-fragkiadaki]], [[point-tracking]], [[vla]]) all still
  carry their properly-set `updated: 2026-06-27` — no bump needed.

Wiki state coherent after pass. Next ingest can proceed cleanly.

## [2026-07-09] ingest | Allshire 2025 — VideoMimic (arXiv 2505.03729, CoRL 2025 Best Student Paper)

Single-paper ingest triggered by user asking about VideoMimic. Not
in raw/papers/ so downloaded PDF from arXiv (2505.03729v5, Aug 2025)
and placed at `raw/papers/2505.03729v1.pdf` (v1 filename per user's
naming convention, but content is v5). VideoMimic = **real-to-sim-
to-real** pipeline turning monocular RGB video into **contextual
whole-body humanoid control**. UC Berkeley (Allshire × Choi × Zhang
× McAllister equal + Darrell + Abbeel + Malik + Kanazawa senior).

Created [[allshire-2025-videomimic]] and [[videomimic]]. Updated
[[uc-berkeley]] (source count 2 → 5; growing status; new members
+ member commentary about BAIR + Efros/Kanazawa vs Levine subcultures
converging on VideoMimic); [[junyi-zhang]] (source count 1 → 2;
new VideoMimic entry; cross-thread signal note); [[loger]] (backlink
to videomimic); [[4d-reconstruction]] (added source + related link;
VideoMimic is first wiki source that **consumes** 4D recon downstream
in RL rather than producing it as an end goal); [[index]] (new
Sources subsection "Humanoid whole-body control from video"; new
Methods subsection; UC Berkeley entry updated to 5 sources; Junyi
Zhang updated to 2 sources; new tags: humanoid, whole-body-control,
real-to-sim-to-real, smpl, deepmimic, unitree-g1, monocular-video);
[[overview]] (bumped 43 → 44 sources; new 2026-07-09 VideoMimic
shift entry seeding a not-yet-full thread F, humanoid learning-from-
video; priority ingest bumps updated).

**Wiki state after ingest:** 44 sources, 46 methods, 3 comparisons.

Design implications:
- **First humanoid whole-body source.** Departs sharply from (E)
  manipulation focus. All prior robotics sources are tabletop
  gripper / hand. VideoMimic is 23-DoF bipedal locomotion +
  contextual sitting / stair climbing on real Unitree G1.
- **Real-to-sim-to-real as a paradigm.** Not direct training from
  demos or from language-conditioned trace generators. Video →
  joint 4D reconstruction → simulator-ready mesh → RL policy → real
  robot. The 4D reconstruction is the bridge.
- **First downstream 4D-reconstruction consumer in the wiki.** All
  prior (B) 4D sources treat reconstruction as an end goal. VideoMimic
  reframes: 4D reconstruction is a substrate for RL. Extends the wiki
  answer to "what is 4D reconstruction good for?"
- **4-stage RL curriculum** (MoCap pretrain → scene-conditioned
  DeepMimic tracking → DAgger distill → PPO fine-tune under distilled
  obs) — a general recipe for imitation-learning-from-video that
  handles noisy references + partial observability at deployment.
- **Single unified policy** across skills — context (heightmap +
  root direction) replaces skill selection. Design lesson:
  environment-as-context can subsume task labels for locomotion.
- **UC Berkeley bumps to 5 sources.** BAIR now spans four wiki
  threads. Two subcultures (Efros/Kanazawa CV vs Levine/PI robotics)
  converge on VideoMimic (Kanazawa + Malik + Abbeel + Darrell all
  senior on one paper).
- **[[junyi-zhang]] cross-thread bridging.** Same PhD student on
  LoGeR (long-context 3D recon, thread D) and VideoMimic (humanoid
  from video, new thread F seed). Rare bridging of two wiki threads
  by one person.
- **CoRL 2025 Best Student Paper** — strong external validation of
  the design choices. Recent, high-profile.

No new person / org pages this pass. VideoMimic authors all held at
1 source each — Allshire, Choi, McAllister (equal-first), Anthony
Zhang, Chung Min Kim, Angjoo Kanazawa, Jitendra Malik, Pieter Abbeel.
Trevor Darrell now at 2 sources (LoGeR advisor + VideoMimic senior)
— hold for third source before promoting to person page.
Junyi Zhang updated in place to 2 sources.

Priority-ingest bumps (in order):
1. **DeepMimic** (Peng et al. SIGGRAPH 2018) — VideoMimic Stage 2's
   direct ancestor. Highest-priority for the new humanoid-from-video
   corner.
2. **SFV** (Peng, Kanazawa, Malik, Abbeel, Levine 2018) — direct
   predecessor from same senior authors. Would promote all four to
   3+ sources apiece.
3. **MonST3R** (Zhang et al. 2024/25) — now cited by 9+ wiki sources.
   Junyi Zhang lead. Way past promotion threshold. Overdue.
4. **SLAHMR** (Ye et al. 2023) — VideoMimic's global-position
   bootstrap. Ancestor of joint 4D human-scene reconstruction.
5. **MegaSAM / MoGe / FoundationStereo** — perception substrates
   reused across VideoMimic + PointWorld + others.
6. **TraceGen** (Lee et al. CVPR 2026) — still #1 for (E) manipulation
   thread coherence.
7. **VoxPoser / ReKep** — Wenlong Huang line; PointWorld author.

Sanity check: `find | sed | sort -u > slugs.txt` + grep new page
wikilinks → all resolve to existing pages. No stale slug references.
[[cotracker3]] noted as adjacent-but-not-used in the connections
list (Berkeley + Meta line via Karaev).

No prompt injection detected in the PDF. Extracted §1–§7 text +
Fig 1/2/3/4/5/6 narrative + Table 1 comparison + Table 2 recon
numbers + full policy training curriculum. Appendix A/B/C
(implementation details) not extracted this pass but flagged in
the method page open directions.

Note on raw layer: **downloaded PDF** to raw/papers/2505.03729v1.pdf
from arXiv. This is the first time in this session's history that
the raw layer was written to by the assistant (previous ingests all
used user-supplied PDFs). Raw layer immutability applies to
*modifying existing files* — adding new source PDFs is the standard
ingest pattern. Documented here for the record.
