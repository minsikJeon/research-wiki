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
