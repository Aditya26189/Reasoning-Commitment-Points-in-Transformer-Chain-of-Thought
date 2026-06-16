# 06. Research Paper Outline: Chain-of-Thought Commitment Points

This document provides a highly structured outline for writing the final top-tier conference paper based on the results from the `notebook_publishable.ipynb` experimental suite.

---

## Abstract
- **Context:** Chain-of-Thought (CoT) prompting enables LLMs to perform complex reasoning by unrolling computation across tokens. However, it is unknown at what precise stage an LLM *commits* to its final answer.
- **Methodology:** We introduce "Truncate & Generate" activation patching. By segmenting reasoning traces into functional stages (`setup`, `computation`, `transition`) and causally intervening on the latent state, we map the trajectory of commitment.
- **Findings:** We identify a macroscopic commitment locus at the computational token boundary. Furthermore, a per-layer mechanistic sweep reveals a "functional rift" (plateau followed by a cliff), demonstrating that early/mid layers process semantic steering while late layers rigidly project in-context priors into terminal outputs.

## 1. Introduction
- The hypothesis of Latent Reasoning: Intermediate tokens act as functional hidden state carriers.
- The concept of the "Commitment Point": The boundary where the internal probability distribution collapses onto a specific terminal answer.
- **Contributions:**
  1. A robust methodology (Truncate & Generate) to causally intervene on autoregressive CoT traces without destroying token alignment.
  2. The isolation of computational tokens as the primary carrier of semantic steering in mathematical reasoning.
  3. The discovery of a layer-wise functional rift demonstrating how commitment structurally solidifies within the transformer block.

## 2. Related Work
- **Activation Patching & Mechanistic Interpretability:** (e.g., Wang et al., Nanda et al.). How our Truncate & Generate macro-methodology differs from traditional static patching.
- **The "Functional Rift":** Citing Dutta et al. regarding the bipartite nature of transformer layers (early context-processing vs. late prior-projection).
- **Chain of Thought Mechanics:** How intermediate reasoning steps influence terminal distributions.

## 3. Methodology
- **3.1 The MATH Dataset Environment:** Why Level 4-5 problems were required over GSM8K (multi-step depth, distinct functional boundaries).
- **3.2 Semantic Segmentation:** Explanation of the `classify_line` algorithm prioritizing `conclusion` > `computation` > `transition` > `setup` to ensure clean masks.
- **3.3 Truncate & Generate Architecture:** 
  - Formulating the causal graph.
  - Slicing $T_{wrong}[:k]$, injecting activations from $T_{correct}[:k]$.
  - The deterministic baseline check to prevent denominator shift artifacts.
- **3.4 Experimental Controls (The 6-Block Suite):**
  - Cross-Problem ablation (ruling out disturbance effects).
  - Shuffled position control (ruling out unordered activation spikes).
  - NF4 artifact check (ruling out quantization noise).

## 4. Results & Analysis
- **4.1 Behavioral Commitment (The Stage-wise Locus):**
  - Reporting the $N=40$ Flip Rates for `setup` vs. `computation_only`.
  - Proving the discontinuity: The injection of computational activations successfully steers the terminal state.
- **4.2 The Cross-Problem Baseline:**
  - Demonstrating that patching from a different math problem yields $~0\%$ flip rate, mathematically proving the effect is *semantic steering*, not *attention disruption*.
- **4.3 The Mechanistic Sweep (Layer-wise Commitment):**
  - **The Plateau (Layers 0-28):** Flips successfully triggered, demonstrating semantic steering in early/mid layers.
  - **The Transition Zone (Layers 30-35):** The precise 4-layer window where the flip rate collapses from 27% to 9%.
  - **The Cliff (Layers 44-47):** The hard $0\%$ flip rate.
  - **Validation:** Textual evidence that late layers are causally locked (outputting the original wrong answer) rather than disrupted (outputting nonsense).

## 5. Discussion
- **5.1 The Anatomy of a Decision:** Synthesizing the behavioral and mechanistic findings into a unified theory of how Qwen2.5-14B makes mathematical decisions.
- **5.2 Limitations:** Sample size constraints, reliance on specific regex masking boundaries, potential artifacts of Truncate & Generate on KV-cache continuity.

## 6. Conclusion
- Summary of the functional rift.
- Implications for future reasoning models and inference-time interventions.
