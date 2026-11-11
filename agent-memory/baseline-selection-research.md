# Open-Source Agent Memory Systems — Baseline Selection Survey

Working document. It records a literature and repository review for
choosing baseline memory system(s) for a research paper on agent memory. The
research was done on 2026-11-11, and every number below is as reported on
that date. Self-reported vendor numbers are labelled as such throughout:
whether a result was measured by the authors themselves or under a shared
harness is the main thing this document tracks.

## 1. Requirements (as specified)

The baseline must be:

1. **A top performer on most benchmarks**, not just one.
2. **Reliable in its evaluation.** Its published numbers should be
   reproducible and measured under a stated, fair protocol, unlike Mem0's
   (see §4.9).
3. **Easy to adapt.** The code should be usable and modifiable for our
   setting.

## 2. Recommendation

| Role | System | Why |
|---|---|---|
| **Primary performance baseline** | **Hindsight** ([`vectorize-io/hindsight`](https://github.com/vectorize-io/hindsight), MIT) | Highest, broadest open-source results (LongMemEval-S, LoCoMo, BEAM 100K–10M, PersonaMem); the most transparent evaluation of the top systems; easy to run as a black box. |
| **Academic baseline we can modify** | **LightMem** ([`zjunlp/LightMem`](https://github.com/zjunlp/LightMem), MIT, ICLR 2026) | Strongest prior academic system in a third-party rerun; peer-reviewed; small, modular core (~6.5K LOC); reproduction scripts and released outputs. |
| **Mandatory controls** | Full-context and BM25 / hybrid (BM25 + dense) RAG | Under independent, controlled evaluation these beat most named memory systems (§3.2). Reviewers will expect them. |
| Optional extra | HippoRAG 2 ([`OSU-NLP-Group/HippoRAG`](https://github.com/OSU-NLP-Group/HippoRAG), MIT, ICML 2025) | Best structured method on MemoryAgentBench; ranked among the top methods in a cross-scenario study (§3.3). |

These criteria pull against each other. Hindsight wins on criterion 1 and
mostly on 2, but it is a production server (~190K LOC Python on Postgres),
so it is easy to *run* but hard to *modify*. LightMem is the opposite: lower
absolute scores, but peer-reviewed and small enough to ablate. Using both
covers all three criteria.

## 3. State of evaluation (why headline numbers are not comparable)

### 3.1 No neutral leaderboard covers the top systems yet

All the 90%+ LongMemEval / LoCoMo numbers (Hindsight, EverMemOS, Mastra,
Agent Zero, Mem0 2026) are self-reported. Each comes from a different
backbone, judge, prompt and sometimes data subset. Cross-paper tables
usually copy competitors' self-reported numbers rather than re-running
them. The one independent, shared-harness leaderboard (AML, §3.4) has
not reported results for Hindsight, Mastra or EverMemOS as named entries
that I could verify.

### 3.2 Simple baselines are strong under controlled evaluation

| Evidence | Finding |
|---|---|
| MemoryAgentBench (ICLR 2026, arXiv 2507.05257 v4, Table 3; GPT-4o-mini backbone for RAG/memory agents) | Overall: long-context GPT-5-mini **60.6**, GPT-4o-mini long-context 42.2, HippoRAG-v2 **41.6**, BM25 **41.5**, Zep 24.0, **Mem0 21.1**, Cognee 20.6. |
| LeanMem rerun (arXiv 2608.03463; GPT-4.1-mini) | On LongMemEval-S, **FullText 77.4** beats every prior memory system (best: LightMem 76.7). |
| LightMem repo, LoCoMo (gpt-4o-mini backbone and judge) | FullText **73.8**, NaiveRAG 63.6, A-MEM 64.2, MemoryOS 58.3, **Mem0 (OSS) 36.5**, Mem0 (API) 61.7. |
| LightMem paper, LongMemEval-S (gpt-4o-mini) | NaiveRAG 61.0 > Mem0 53.6, MemoryOS 44.8, LangMem 37.2 (LightMem: 64.3–68.6). |
| Mem0 vs long-context (arXiv 2603.04814) | Long-context GPT-5-mini beats a Mem0-based system by ~33–35 points on LongMemEval and LoCoMo. Mem0 is competitive only on PersonaMem v2. |
| AML Cycle 1 (Aug 2026, §3.4) | The open-source text-memory track was won by a **vanilla RAG** system (InvMem, 45.06). |
| AMB (Vectorize's own harness, §4.1) | A plain hybrid-search baseline gets 74.0 on LongMemEval-S and 79.1 on LoCoMo. |

### 3.3 No single architecture dominates

Three controlled studies agree that no single architecture wins
everywhere:

- **Cross-scenario generality** (arXiv 2606.04315). Eight systems, five
  scenarios, 12 task columns. Average rank (lower is better): DCI-Lite
  agentic grep harness 3.67, long context 4.00, HippoRAG 4.17,
  DCI-Lite+Sum 4.28, **LightMem 5.11**, Mem-T 6.06, AMA-Agent 6.61,
  SimpleMem 6.83, PlugMem 6.94, MemRL 7.33.
- **Harness the Memory** (arXiv 2608.15008). Eleven memory substrates
  under one harness, three backbones, four benchmark suites. The best
  substrate flips between dialogue QA and agentic tasks.
- **Are We Ready for an Agent-Native Memory System?** (arXiv 2606.24775,
  SJTU, with co-authors from MemTensor, the company behind MemOS). Twelve
  systems. The leader shifts by workload.

### 3.4 Agent Memory Leaderboard (AML), the closest to neutral

- An independent platform. Participants expose `Add` / `Search`; AML fixes
  the answer model, scoring, `top_k=100` and a 117,760-token input
  budget. Private held-out question sets.
- **Cycle 1** (published 2026-08-12): 136 team applications, 67 systems
  completed evaluation.
  - Text memory, open-source division: InvMem (vanilla RAG) 45.06,
    ReFind 44.97, ActiveMemoryIndex 44.84.
  - Text memory, commercial division: MemoraX v0.5 58.02, MemOS v2.0.1
    45.89, NTES-MEMORY-SMART 44.21.
- **Cycle 2** results are due mid-November 2026. **Check them before the
  submission.**
- The full Cycle 1 table is on Hugging Face, which this environment could
  not reach, so I have only the published top-3 entries.

### 3.5 LoCoMo-specific problems

- Per the [locomo-audit](https://github.com/dial481/locomo-audit) project,
  99 of the 1,540 scored questions have score-corrupting errors, and the
  standard judge accepted 62.81% of intentionally wrong answers. These
  figures come from secondary summaries; I did not re-verify them.
- Most papers drop category 5 (adversarial/unanswerable). The Zep–Mem0
  dispute was largely about this choice (Zep's 84% recounted to 58.44%,
  later revised to 75.14%).
- LoCoMo conversations are short (~16–26K tokens), so it is near
  saturation for strong long-context models. Report it, but don't lean
  on it.

## 4. Candidates

### 4.1 Hindsight (Vectorize) — recommended primary baseline

- **Paper:** "Hindsight is 20/20: Building Agent Memory that Retains,
  Recalls, and Reflects" (arXiv 2512.12818). Vectorize, with
  collaborators from The Washington Post and Virginia Tech.
- **Design:** four memory networks (world facts, agent experiences,
  entity summaries, evolving beliefs). Recall fuses dense, BM25, graph
  (entity / causal links) and temporal retrieval.
- **Paper results** (memory LLM GPT-OSS-20B or 120B; judge GPT-OSS-120B,
  temperature 0; LongMemEval's per-question-type judge prompts):

  | Config | LongMemEval-S | LoCoMo |
  |---|---|---|
  | Full-context, OSS-20B | 39.0 | — |
  | Hindsight, OSS-20B | 83.6 | 83.18 |
  | Hindsight, OSS-120B | 89.0 | 85.67 |
  | Hindsight, OSS-120B memory + Gemini-3 Pro answerer | **91.4** | **89.61** |

  Competitor numbers in the paper are copied: Zep+GPT-4o 71.2 and
  Supermemory+GPT-4o 81.6 from Supermemory's report; on LoCoMo,
  Memobase 75.78, plus Backboard's unreproduced claim of 90.00.
- **AMB harness results** ([`vectorize-io/agent-memory-benchmark`](https://github.com/vectorize-io/agent-memory-benchmark);
  a Gemini model answers and judges; per-question outputs published):

  | Dataset | Hindsight | Hybrid-search baseline | Hindsight avg. context tokens |
  |---|---|---|---|
  | LongMemEval-S (500) | **94.6** | 74.0 | 43,624 |
  | LoCoMo (1,540) | **92.0** | 79.1 | 36,235 |
  | LoCoMo-fixed, agent mode (1,502) | 85.5 | — | — |
  | BEAM 100K / 500K / 1M / 10M | 73.4 / 71.1 / **73.9** / **64.1** | — | 17.7K–27.3K |
  | PersonaMem 32K | 86.6 | 84.4 | 15,812 |
  | LifeBench (en) | 71.5 | 61.0 | 34,324 |

  For scale: Mem0's own blog reports 64.1% on BEAM-1M and 48.6% on
  BEAM-10M.
- **Other evidence:**
  - DolphinBench (arXiv 2609.24971), an action-based benchmark attributed
    to Mem0 in Mem0's own benchmark overview. In one harness/model setup,
    Hindsight scored 1 pp above Honcho at lower cost but higher latency.
    No system won in every setup.
  - Agent Zero Memory's paper lists Hindsight 3rd on LongMemEval and 4th
    on LoCoMo, but every number in those tables is copied from other
    sources.
- **Reliability caveats:**
  1. Vectorize built both the system and the AMB harness.
  2. The README's "independent reproduction" was done by the paper's own
     collaborators, and no standalone third-party rerun has been published.
  3. It feeds ~36–44K retrieved tokens per question to the answerer
     (LongMemEval-S histories average ~115K), so part of the gain is a big
     context plus a strong reader. **Control or report the retrieved-token
     budget** when comparing.
- **Code / adaptability:**
  - MIT licence. 25+ LLM providers, including `ollama` and any
    OpenAI-compatible endpoint, so it can run on the **same backbone as our
    system**.
  - Python client with `retain` / `recall` / `reflect`. Embedded mode
    (`pip install hindsight-all`, `HindsightServer(...)` context manager)
    needs no separately managed server. Postgres underneath (embedded
    `pg0` by default).
  - The core engine (`hindsight-api-slim/hindsight_api/engine/`) is ~190K
    LOC. The largest parts are `memories/` (~18K), `providers/` (~15K),
    `retain/` (~14K) and `reflect/` (~8K). Fine as a black box,
    heavyweight to modify or ablate.

### 4.2 LightMem (ZJUNLP) — recommended academic baseline

- **Paper:** "LightMem: Lightweight and Efficient Memory-Augmented
  Generation" (arXiv 2510.18866), **ICLR 2026**. MIT licence.
- **Third-party rerun** (LeanMem, arXiv 2608.03463; GPT-4.1-mini judge;
  LoCoMo categories 1–4; 5-run averages). Accuracy:

  | Method | LoCoMo (4.1-mini) | LME-S (4.1-mini) | LoCoMo (Qwen3-8B) | LME-S (Qwen3-8B) |
  |---|---|---|---|---|
  | FullText | 56.92 | 77.40 | 50.83 | — |
  | NaiveRAG | 64.09 | 67.60 | 51.04 | 65.00 |
  | MemoryOS | 65.39 | 67.80 | 63.04 | 63.40 |
  | A-Mem | 67.58 | 71.40 | 69.80 | 69.00 |
  | SimpleMem | 76.86 | 74.60 | 71.14 | **74.60** |
  | **LightMem** | **79.33** | **76.73** | **78.57** | 69.80 |
  | *LeanMem (the paper's own method)* | *84.87* | *91.80* | *84.41* | *77.40* |

  LightMem is the best prior method in 3 of 4 settings. It is also cheap:
  ~107–150K construction tokens per conversation, vs ~1.2–1.35M for A-Mem.
- **Own results:** LongMemEval-S 64.3–68.6 (gpt-4o-mini) and up to 70.2
  (Qwen3-30B-A3B). The lower absolute numbers come from the small
  backbones.
- **Cross-scenario study** (§3.3): average rank 5.11, the best of the
  "semantic memory" systems; LoCoMo 62.0 vs long context 61.5.
- **Code / adaptability:**
  - Core package `src/lightmem/` is ~6.5K LOC: `configs/` (~0.9K),
    `factory/` (~3.4K, pluggable LLM / embedder / retriever),
    `memory/` (~2.3K).
  - Main API: `LightMemory.from_config`, `add_memory`, `retrieve`, plus
    `online_update` / `offline_update` (offline consolidation).
  - Reproduction scripts in `experiments/locomo/` and
    `experiments/longmemeval/`. Run outputs and judge logs are released on
    Google Drive.
  - Includes a baseline toolkit (`src/lightmem/memory_toolkits/`, ~37K
    LOC: Mem0, A-MEM, LangMem, ...) and the companion harness
    [`zjunlp/MemBase`](https://github.com/zjunlp/MemBase), which covers
    Mem0, A-MEM, EverMemOS and LangMem on LoCoMo and LongMemEval.

### 4.3 EverMemOS / EverOS (EverMind)

- **Self-reported** (repo `benchmarks/README.md`): LoCoMo **94.42** and
  LongMemEval **94.00**, but with a **different model stack per
  benchmark**. LoCoMo: DeepSeek-v4-flash decider + GPT-4.1-mini answerer.
  LongMemEval: Qwen3.6-27B decider + Gemini-3.6-flash answerer. Older
  materials report 92.3 / 93.05 on LoCoMo and 83.0 on LongMemEval, so the
  numbers have moved around.
- **Independent** (StreamMemBench, arXiv 2606.14571; DeepSeek-V4-Flash):
  IEU 35.02 / FUR 48.94, mid-pack. A-Mem got 34.92 / 64.98 and raw RAG
  27.95 / 60.96.
- Apache-2.0. A good harness design (resumable ADD → SEARCH → ANSWER →
  JUDGE with a `run_spec.json` that records the models served). ~174K LOC
  of Python.
- **Not chosen:** model stacks are cherry-picked per benchmark, and the
  independent results do not back up the headline numbers.

### 4.4 Mastra Observational Memory

- **94.87%** on LongMemEval-S with gpt-5-mini (self-reported). An
  observer/reflector compresses history into a dated log that stays in
  context.
- **Not chosen:** only one benchmark reported; TypeScript, inside the
  Mastra framework; reads are query-independent, so every turn pays for
  the whole log.

### 4.5 Agent Zero Memory

- Claims **95.60** on LongMemEval and **93.60** on LoCoMo (arXiv
  2608.29606), and accuracy within 92.2–95.6 across eight backbones.
- **Not chosen:** the comparison tables are all copied self-reported
  numbers, and I could not find released code.

### 4.6 MemOS (MemTensor)

- 2nd in AML Cycle 1, commercial text track (45.89). It also leads LoCoMo
  exact match in the SJTU study, which has MemTensor co-authors.
- Independently it **collapses on StreamMemBench** (IEU 2.94 / FUR 3.96,
  vs ~19–35 IEU for the other systems).
- **Not chosen:** inconsistent across independent evaluations.

### 4.7 HippoRAG 2 (OSU NLP) — optional extra baseline

- ICML 2025, MIT, small (~12K LOC Python).
- Best structured method on MemoryAgentBench: overall 41.6, LME(S*) 50.7.
  Average rank 4.17 in the cross-scenario study (3rd of 10).
- More a continual-learning RAG system than a conversational memory
  layer. Reviewers know it well, so it is a cheap credibility add.

### 4.8 A-Mem, SimpleMem, MemoryOS

- **A-Mem** (NeurIPS 2025): best agentic memory on StreamMemBench (FUR
  64.98) and strong evidence recall (Recall@10 85.9 in the SJTU study).
  But it is very expensive (~1.2–1.35M construction tokens per
  conversation).
- **SimpleMem** (arXiv 2601.02553, MIT): close to LightMem in the LeanMem
  rerun. Its own headline LoCoMo results are reported as **F1**, a weak
  metric for free-form answers. Its newer variants (Omni-SimpleMem,
  EvolveMem) are tuned by an automated research loop against the
  benchmarks themselves.
- **MemoryOS** (EMNLP 2025): consistently below LightMem, SimpleMem and
  A-Mem in third-party reruns.

### 4.9 Mem0 — excluded, as suspected

- **Independent results are poor:** MemoryAgentBench overall 21.1 (below
  BM25 at 41.5); LightMem repo LoCoMo **36.5 for the OSS library** vs 61.7
  for the hosted API; ~33–35 points behind long-context GPT-5-mini on
  LongMemEval and LoCoMo (arXiv 2603.04814).
- **Reproduction problems:** GitHub issue
  [#3944](https://github.com/mem0ai/mem0/issues/3944): ~0.20 LLM score on
  LoCoMo via the platform, traced to memories being stamped with the
  current date instead of the dataset timestamps. Issue #2800: local
  reproduction failures. Then there is the Zep–Mem0 scoring dispute
  (§3.5).
- The 2026 self-reported numbers (LoCoMo 92.5, LongMemEval 94.4) come from
  a new algorithm and Mem0's own harness. No independent rerun yet.

## 5. Recommended evaluation protocol

1. **Run every system ourselves under one harness.** Same LLM for memory
   construction and for answering; same judge; same embedding model where
   the system allows it. Absolute scores swing with the answerer: LightMem
   gets 68.6 with gpt-4o-mini, Hindsight 91.4 with a Gemini-3 Pro
   answerer.
2. **Judges:** use LongMemEval's official per-question-type judge prompts.
   Report a second judge from a different model family, plus agreement.
3. **Control or report the retrieved-context budget** (tokens per query).
   Also report construction tokens, query tokens and latency, as
   LeanMem, LightMem and AMB do.
4. **Benchmarks:**
   - LongMemEval-S (and M if feasible)
   - LoCoMo, categories 1–4, with the label-error caveat (consider
     reporting on a corrected version too)
   - BEAM 1M / 10M, for scale beyond the context window
   - one agentic / stress suite, e.g. MemoryAgentBench
5. **Always include** full-context and BM25 / hybrid RAG.
6. **Multiple runs** (≥3) with paired significance tests, as LeanMem does.
7. **Release per-question outputs** for every system, as Hindsight's AMB,
   EverOS and LightMem do. This is now the bar for a credible memory paper.

Existing harnesses to reuse rather than writing from scratch:

- AMB (open, but vendor-built)
- `zjunlp/MemBase`
- EverOS `benchmarks/` (good staged / resumable design)
- MemoryAgentBench

## 6. Open questions / caveats

- [ ] Check **AML Cycle 2** results (due mid-November 2026) for an
      independent ranking that may include Hindsight / EverOS / Mem0.
- [ ] The full AML Cycle 1 table (Hugging Face) could not be fetched from
      this environment; only top-3 entries are recorded above.
- [ ] LoCoMo audit figures (§3.5) come from secondary summaries and were
      not re-verified.
- [ ] Decide whether to run Hindsight's `reflect` (agentic answer) mode or
      only `recall` + our own answerer. For a controlled comparison,
      `recall` + a shared answerer is cleaner; report `reflect` separately
      if used.
- [ ] Budget: Hindsight's and A-Mem's write paths call the LLM on every
      message. Estimate the API / GPU cost of LongMemEval-S (500 histories
      × ~115K tokens) before committing to all baselines.

## 7. Sources

**Systems**
- Hindsight paper — https://arxiv.org/abs/2512.12818
- Hindsight repo — https://github.com/vectorize-io/hindsight
- AMB harness and results — https://github.com/vectorize-io/agent-memory-benchmark
- Critique of Hindsight's reproduction claim — https://zby.github.io/commonplace/notes/related-systems/hindsight
- LightMem paper — https://arxiv.org/abs/2510.18866
- LightMem repo — https://github.com/zjunlp/LightMem
- MemBase harness — https://github.com/zjunlp/MemBase
- EverMemOS / EverOS — https://github.com/EverMind-AI/EverMemOS
- EverMind evaluation write-up — https://evermind.ai/blogs/a-unified-evaluation-framework-for-ai-memory-systems
- Mastra Observational Memory — https://mastra.ai/research/observational-memory
- Agent Zero Memory — https://arxiv.org/abs/2608.29606
- SimpleMem — https://arxiv.org/abs/2601.02553
- HippoRAG — https://github.com/OSU-NLP-Group/HippoRAG

**Independent and controlled evaluations**
- MemoryAgentBench — https://arxiv.org/abs/2507.05257
- LeanMem (third-party rerun) — https://arxiv.org/abs/2608.03463
- StreamMemBench — https://arxiv.org/abs/2606.14571
- Cross-scenario generality — https://arxiv.org/abs/2606.04315
- Harness the Memory — https://arxiv.org/abs/2608.15008
- Are We Ready for an Agent-Native Memory System? — https://arxiv.org/abs/2606.24775
- Mem0 vs long-context — https://arxiv.org/abs/2603.04814
- DolphinBench — https://arxiv.org/abs/2609.24971
- Ground Truth First — https://arxiv.org/abs/2607.21962

**Leaderboards and lists**
- Agent Memory Leaderboard (AML) — https://github.com/AML-memory/agent-memory-leaderboard and https://agentmemoryleaderboard.ai/
- Awesome-Agent-Memory — https://github.com/TeleAI-UAGI/Awesome-Agent-Memory

**Evaluation disputes and guides**
- Mem0 reproduction issue #3944 — https://github.com/mem0ai/mem0/issues/3944
- LoCoMo audit — https://github.com/dial481/locomo-audit
- Mem0 benchmark overview (2026) — https://mem0.ai/blog/ai-memory-benchmarks-in-2026
- Cognee benchmark guide — https://www.cognee.ai/agent-memory-evaluation-datasets-benchmarks
- Comparison of five memory systems (Medium) — https://medium.com/@wasowski.jarek/i-compared-5-ai-agent-memory-systems-across-6-dimensions-none-wins-6a658335ed0a
