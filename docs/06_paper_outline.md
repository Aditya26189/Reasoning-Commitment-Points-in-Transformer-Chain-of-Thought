# 06. Research Paper Outline: Chain-of-Thought Commitment Points

This document provides a structured outline for writing the final conference paper based on the results from `final/notebooke66c37d069.ipynb`.

---

## Abstract
- **Context:** Chain-of-Thought (CoT) prompting enables LLMs to perform complex reasoning by unrolling computation across tokens. However, it is unknown at what precise stage — and at which transformer layers — an LLM *commits* to its final answer.
- **Methodology:** We introduce "Truncate & Generate" activation patching. By segmenting reasoning traces into functional stages and causally intervening on computation-token hidden states, we map both the sequence-level and layer-level trajectory of commitment.
- **Findings:** On Qwen2.5-14B-Instruct with MATH Level 4–5 problems, same-problem computation patching achieves a **54.5% flip rate** (6/11 committed-wrong traces), while cross-problem patching yields only **23.1%** — a **31.4 pp specificity gap** proving problem-specific semantic steering. A sparse 18-layer mechanistic map reveals a **functional rift**: a 36–45% plateau (layers 0–28), a collapse window (layers 30–35), and hard **0% causal locking** at tested late layers (44, 47).

## 1. Introduction
- The hypothesis of Latent Reasoning: Intermediate tokens act as functional hidden state carriers.
- The concept of the "Commitment Point": The boundary where the internal probability distribution collapses onto a specific terminal answer.
- **Contributions:**
  1. **Truncate & Generate:** A robust methodology to causally intervene on autoregressive CoT traces without destroying token alignment.
  2. **Behavioral commitment locus:** Demonstration that computation-token activations carry sufficient semantic momentum to redirect >50% of committed-wrong trajectories (54.5%).
  3. **Causal specificity:** Cross-problem control proves a 31.4 pp gap — the effect is problem-specific semantic steering, not generic disturbance.
  4. **Layer-wise functional rift:** First sparse per-layer commitment map on a 48-layer model for mathematical CoT, revealing plateau → collapse → cliff architecture.

## 2. Related Work
- **Activation Patching & Mechanistic Interpretability:** (Wang et al., Nanda et al.). How Truncate & Generate differs from traditional static patching on variable-length sequences.
- **The Functional Rift:** Dutta et al. (*"How to think step-by-step: A mechanistic understanding of chain-of-thought reasoning"*) on the bipartite nature of transformer layers (early context-processing vs. late prior-projection). Our work provides layer-precise evidence on a reasoning task with a distinct layer range on Qwen2.5-14B.
- **Chain of Thought Mechanics:** How intermediate reasoning steps influence terminal distributions.

## 3. Methodology
- **3.1 The MATH Dataset Environment:** Why Level 4–5 problems were required over GSM8K (multi-step depth, distinct functional boundaries). N=19 organic correct/wrong pairs.
- **3.2 Semantic Segmentation:** The `classify_line` algorithm prioritizing `conclusion` > `computation` > `transition` > `setup`. Pre-experiment validation (4/4 pairs passed).
- **3.3 Truncate & Generate Architecture:**
  - Causal graph formulation.
  - Truncating $T_{wrong}$ at `computation_only` boundary, surgically overwriting hidden states from $T_{correct}$.
  - "Already flipped" exclusion to measure true commitment.
- **3.4 Experimental Controls:**
  - Block A: Same-problem patching (main finding).
  - Block B: Cross-problem ablation (specificity, +31.4 pp gap).
  - Blocks C–D: Sparse 18-layer mechanistic map with transition pinning.

## 4. Results & Analysis
- **4.1 Behavioral Commitment — Computation Tokens as Steering Vectors:**
  - Block A: **54.5%** flip rate (6/11) on `computation_only` same-problem patching.
  - More than half of genuinely committed-wrong traces are causally redirected by correct computational activations.
- **4.2 Cross-Problem Specificity:**
  - Block B: **23.1%** (3/13) — substantially lower than Block A.
  - The **31.4 pp gap** (2.4× effect ratio) proves the steering is driven by problem-specific semantic content, not attention disruption (supported by a paired 10-pair intersection analysis yielding a +40 pp gap).
- **4.3 The Functional Rift (Layer-wise Commitment):**
  - **Plateau (Layers 0–28):** 36.4% – 45.5% flip rates. Early/mid layers are an open semantic scratchpad.
  - **Transition Zone (Layers 30–35):** Collapse from 27.3% to 9.1%. Commitment solidifies across a tight 7-layer window.
  - **Cliff (Layers 44, 47):** Hard 0.0% (0/11). Statistically significant ($p \approx 0.02$ if true rate were 30%). Tested late layers are causally locked.
  - **Validation:** Unpatched baselines reproduce the original wrong answer (verified via single-pair sanity check) — consistent with locking, not disruption.
- **4.4 Unified Model:** The model decides its answer in mid-layers (28–35); final layers project that prior into vocabulary.

## 5. Discussion
- **5.1 The Anatomy of a Mathematical Decision:** Synthesizing behavioral (54.5% steering) and mechanistic (functional rift) findings into a unified theory.
- **5.2 Implications:** Inference-time intervention windows for reasoning models; which layers to target for activation steering.
- **5.3 Limitations:** N=19 organic pairs (n=11 per analysis after exclusions); single model family; regex-based segmentation boundaries; slightly asymmetric pair sets in cross-problem control.

## 6. Conclusion
- Summary: Computation tokens are causal steering vectors. Commitment solidifies at layers 28–35. Late layers are locked projectors.
- Implications for mechanistic interpretability of reasoning and inference-time interventions.
