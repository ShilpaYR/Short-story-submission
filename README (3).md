# Test-Time Scaling in Reasoning LLMs — CMPE 297 Short Story

**Student:** Shilpa Yelkur Ramakrishnaiah · San José State University · CMPE 297, Fall 2026

A short-story review of **[Test-Time Scaling in Reasoning LLMs: Inference Regimes, Evaluation, and Reproducibility](https://arxiv.org/abs/2608.04001)**
by Mohsen Hariri, Weicong Chen, Nahal Shahini, et al. (Case Western Reserve University), arXiv:2608.04001, posted 4 August 2026.

| Deliverable | Link |
| --- | --- |
| 📄 Paper (PDF) | https://arxiv.org/pdf/2608.04001 |
| ✍️ Medium article | https://medium.com/@shilpa.yelkurramakrishnaiah/more-thinking-fewer-guarantees-how-to-read-test-time-scaling-claims-80ab5f85d2a7 |
| 💻 GitHub repo | https://github.com/ShilpaYR/Short-story-submission |
| 📊 Slides (PDF) | [`TTS_short_story.pdf`](TTS_short_story.pdf) |
| 📊 Slides (PowerPoint) | [`TTS_short_story.pptx`](TTS_short_story.pptx) |
| 🎥 Video walkthrough (YouTube) | https://www.youtube.com/watch?v=EL4JvS3GcBs |

---

## Why this paper

Reasoning models such as DeepSeek-R1, Qwen3 Thinking and gpt-oss improve when they spend more compute at inference time. They can think longer, sample more answers, or search over partial solutions. But "test-time scaling" now names very different algorithms, with different costs and failure modes, so published results rarely compare. This survey:

1. gives one formal framework and taxonomy for all of them;
2. proposes how to **evaluate the whole inference system**, covering compute accounting, metrics and uncertainty;
3. separates two meanings of **reproducibility**;
4. maps the open-weight reasoning-model ecosystem and backs its claims with a **1,403,520-attempt** public trace release.

It fits the course's focus on LLM development, evaluation and LLMOps.

## The framework: one tree, one budget

Every method is modeled as *budgeted inference over the model's prefix tree*. The prompt is the root, partial reasoning traces are nodes, and finished responses are leaves. The decoding policy decides which leaves get sampled. Every token, verifier call and controller step costs part of a budget **B**. The quantity to report is answer quality as a function of B.

![Three regimes of test-time scaling](fig1_taxonomy.png)

## Taxonomy and key techniques

| Regime | What grows | Key techniques | Main failure mode |
| --- | --- | --- | --- |
| **Single-trajectory** (sequential) | One chain, extended or stopped by a controller | s1 budget forcing ("Wait"), Qwen3 thinking budgets, gpt-oss reasoning effort | Length caps: 16.12% of high-effort gpt-oss-20b answers hit the 81,920-token cap |
| **Leaf-level** (parallel) | N complete answers + a reducer | Self-consistency, Best-of-N with RM/verifier, MBR, DeepConf, Adaptive-Consistency, ESC | Proxy over-optimization; verifier false positives; O(N²) MBR cost |
| **Prefix-level** (search) | Scored partial prefixes, expanded or pruned | Beam / best-first search, MCTS (RAP), Tree-of-Thoughts, PRM-guided search, DEFT | Pruning loss: a correct leaf whose ancestors are pruned can never be recovered |

Hybrids are common, for example a search followed by a vote over the leaves it reaches.

## Evaluation protocol (the LLMOps part)

**Compute accounting.** C_total = C_gen + C_eval, where C_eval = C_signal + C_control + C_decision. Count warm-up and discarded tokens. Equal sample counts are not equal compute. Report latency and memory *beside* the budget.

**One metric family.** S(k, t) is the share of prompts with at least t correct answers among k attempts.

| Metric | Slice of S(k, t) | Question |
| --- | --- | --- |
| Pass@k | S(k, 1) | Can the model ever find it? |
| pass^k | S(k, k) | Is it right every time? |
| Maj@k | bounded by S(k, ⌊k/2⌋+1) | Would majority vote work? |
| G-Pass@k (τ) | S(k, ⌈τk⌉) | Right in at least a τ share of tries? |
| Geom@k | √(S(k,1)·S(k,k)) | Balance of discovery and stability |

Uncertainty comes from a prompt bootstrap that recomputes the reducer on each replicate, or from a Beta(1,1) Bayesian posterior.

**Reproducibility.** *Exact replay* means the same candidates, scores and decisions; it needs logged raw outputs and random streams. *Distributional reproducibility* means independent reruns give statistically compatible estimates. Precision, kernels and batching can change even greedy outputs, so report them.

## Key results

**Finding ≠ picking.** On AIME/HMMT/BrUMO (20 configurations × 80 attempts), median Pass@k rises from 56.49% (k=1) to 82.08% (k=80). The median share solved in *all* 80 attempts is only 15%.

![Discovery vs selection](fig2_discovery_vs_selection.png)

**The selection gap.** On 186 MathArena problems, reference-free selection trails Pass@80 by 8–16 points. A reference-based verifier reaches ROC AUC 0.983 against rule-based grading, while a reference-free pointwise judge reaches 0.871.

![Selection gap](fig3_selection_gap.png)

**Discovery is not reliability.** On SuperGPQA (3,600 questions, 72 fields), gpt-oss-20b at high effort reaches Pass@80 81.94%, mean accuracy 45.03% and pass^80 only 9.47%. Field accuracy ranges from 21.33% (Aquaculture) to 76.78% (Mathematics).

![Discovery vs stability](fig4_discovery_vs_stability.png)

Rankings are cheap to estimate: k=1 rankings already agree with the 80-sample ranking at Spearman 0.967.

## My two cents

- **Strengths:** a unifying framework; a reporting checklist practitioners can adopt directly; a large public trace bank for testing new selectors cheaply.
- **Limitations:** dense notation; experiments limited to math and knowledge QA on open-weight models (no agents, code or tool use); cost measured in tokens and calls rather than FLOPs or dollars; the selection gap is diagnosed but not closed.
- **Biggest open problem:** reliable *reference-free* verifiers. The bottleneck has moved from generating answers to recognizing them.

## Repository contents

| File | What it is |
| --- | --- |
| `README.md` | This summary, with all links and references |
| `TTS_short_story.pdf` / `TTS_short_story.pptx` | The 15-slide deck |
| `fig1_taxonomy.png` … `fig4_discovery_vs_stability.png` | Figures used above and in the Medium article |

All numbers in the figures come from Hariri et al. (2026), Section 4.

## References

1. **Main paper** — Hariri, M., Chen, W., Shahini, N., et al. (2026). *Test-Time Scaling in Reasoning LLMs: Inference Regimes, Evaluation, and Reproducibility.* [arXiv:2608.04001](https://arxiv.org/abs/2608.04001)
2. Zhang, Q., et al. (2025). *A Survey on Test-Time Scaling in Large Language Models: What, How, Where, and How Well?* [arXiv:2503.24235](https://arxiv.org/abs/2503.24235)
3. Muennighoff, N., et al. (2025). *s1: Simple test-time scaling.* [arXiv:2501.19393](https://arxiv.org/abs/2501.19393)
4. Wang, X., et al. (2023). *Self-Consistency Improves Chain of Thought Reasoning in Language Models.* ICLR 2023. [arXiv:2203.11171](https://arxiv.org/abs/2203.11171)
5. Snell, C., Lee, J., Xu, K., Kumar, A. (2024). *Scaling LLM Test-Time Compute Optimally can be More Effective than Scaling Model Parameters.* [arXiv:2408.03314](https://arxiv.org/abs/2408.03314)
6. Yao, S., et al. (2023). *Tree of Thoughts: Deliberate Problem Solving with Large Language Models.* [arXiv:2305.10601](https://arxiv.org/abs/2305.10601)
7. Hao, S., et al. (2023). *Reasoning with Language Model is Planning with World Model (RAP).* [arXiv:2305.14992](https://arxiv.org/abs/2305.14992)
8. Lightman, H., et al. (2023). *Let's Verify Step by Step.* [arXiv:2305.20050](https://arxiv.org/abs/2305.20050)
9. Fu, Y., Wang, X., Tian, Y., Zhao, J. (2025). *Deep Think with Confidence (DeepConf).* ICLR 2026. [arXiv:2508.15260](https://arxiv.org/abs/2508.15260)
10. Aggarwal, P., et al. (2023). *Let's Sample Step by Step: Adaptive-Consistency for Efficient Reasoning.* [arXiv:2305.11860](https://arxiv.org/abs/2305.11860)
11. Li, Y., et al. (2024). *Escape Sky-high Cost: Early-stopping Self-Consistency for Multi-step Reasoning.* ICLR 2024. [arXiv:2401.10480](https://arxiv.org/abs/2401.10480)
12. Liu, J., et al. (2025). *Are Your LLMs Capable of Stable Reasoning? (G-Pass@k).* Findings of ACL 2025. [arXiv:2412.13147](https://arxiv.org/abs/2412.13147)
13. Balunović, M., Dekoninck, J., Petrov, I., Jovanović, N., Vechev, M. (2025). *MathArena: Evaluating LLMs on Uncontaminated Math Competitions.* NeurIPS 2025 D&B. [arXiv:2505.23281](https://arxiv.org/abs/2505.23281)
14. M-A-P Team (2025). *SuperGPQA: Scaling LLM Evaluation across 285 Graduate Disciplines.* [arXiv:2502.14739](https://arxiv.org/abs/2502.14739)
15. DeepSeek-AI (2025). *DeepSeek-R1.* [arXiv:2501.12948](https://arxiv.org/abs/2501.12948)
16. OpenAI (2025). *gpt-oss-120b & gpt-oss-20b Model Card.* [arXiv:2508.10925](https://arxiv.org/abs/2508.10925)
17. Qwen Team (2025). *Qwen3 Technical Report.* [arXiv:2505.09388](https://arxiv.org/abs/2505.09388)
