<div align="center">
  <h1>🧠 Chain-of-Thought Commitment Points</h1>
  <p><em>Mechanistic Interpretability of Latent Reasoning in Large Language Models</em></p>
  
  [![Model](https://img.shields.io/badge/Model-Qwen2.5--14B--Instruct-blue)](https://huggingface.co/Qwen/Qwen2.5-14B-Instruct)
  [![Dataset](https://img.shields.io/badge/Dataset-MATH_Level_4--5-green)](https://huggingface.co/datasets/DigitalLearningGmbH/MATH-lighteval)
  [![Methodology](https://img.shields.io/badge/Method-Truncate_&_Generate-orange)]()
</div>

<br/>

## 📖 Overview

How does a Large Language Model (LLM) utilizing Chain-of-Thought (CoT) prompting arrive at a mathematical conclusion? Does it "know" the answer from the first generated token, or does the reasoning process dynamically compute the result in the intermediate layers?

This repository investigates the **Commitment Point** of LLMs—the precise mechanistic moment where the model's internal probability distribution irrevocably collapses onto a specific terminal answer.

By employing a novel macroscopic activation patching technique called **"Truncate & Generate"**, we bypass the token alignment destruction typical of standard patching and successfully map the sequence-level and layer-level causality of mathematical reasoning on Qwen2.5-14B-Instruct.

## 📊 Key Results (Final Run)

Executed in `final/notebooke66c37d069.ipynb` on **N=19 organic correct/wrong trace pairs** from MATH Level 4–5:

| Experiment | Flip Rate | Detail |
| :--- | :---: | :--- |
| **Block A — Same-Problem Patching** | **54.5%** | 6/11 committed-wrong traces steered to GT via `computation_only` activations |
| **Block B — Cross-Problem Control** | **23.1%** | 3/13 when activations come from a *different* math problem |
| **Specificity Gap** | **+31.4 pp** | Same-problem steering is 2.4× more effective — exceeds our 25 pp design threshold |

After excluding traces that self-corrected without patching ("already flipped"), **more than half** of genuinely committed-wrong trajectories can be causally redirected by injecting correct computational hidden states. The cross-problem control confirms this is predominantly **problem-specific semantic steering**, not generic attention disruption.

## 🚀 Key Discovery: The Functional Rift

Our most significant finding is the empirical mapping of a **"Functional Rift"** across the transformer stack via a sparse 18-layer sweep. By causally intervening on `computation` semantic tokens at each tested layer independently, we discovered a distinct tripartite architecture of commitment:

<div align="center">
  <img src="final/final_nb_img_40_1.png" alt="Per-Layer Commitment Curve showing the Functional Rift" width="800"/>
</div>

1. **The Plateau (Layers 0–28): Semantic Steering**
   - Intervening on computational tokens in early/mid layers flips the terminal answer **36–45%** of the time (n=11 per layer). These layers act as an open scratchpad, assembling algebraic meaning that can redirect the trajectory.
2. **The Transition Zone (Layers 30–35): The Collapse**
   - We isolated the commitment drop to a precise window — flip rate falls from **27%** at layer 30 to **9%** at layer 35. The latent distribution is collapsing onto the terminal state.
3. **The Cliff (Layers 44, 47): Causal Locking**
   - Flip rate hits a hard **0%** (0/11 at layers 44 and 47, the only late layers tested). Even injecting correct mathematical reasoning into the residual stream cannot redirect the output. Late layers rigidly project the in-context prior into the original wrong answer.

This confirms and extends the bipartite theory of transformer layer function (Dutta et al.) on a complex multi-step mathematical reasoning task.

## 🔬 Methodology: Truncate & Generate

Traditional activation patching suffers from catastrophic alignment failure when dealing with variable-length CoT reasoning traces. We solved this by operating at the macro-sequence level:

1. **Semantic Segmentation:** Parse reasoning traces into strict functional masks (`setup`, `computation`, `transition`, `conclusion`).
2. **Truncation:** Truncate a trace leading to a *wrong* answer ($T_{wrong}$) immediately after a computational step.
3. **Injection:** Surgically overwrite hidden states at computation token positions with activations from the equivalent step in a *correct* trace ($T_{correct}$).
4. **Generation:** Allow the model to freely generate from that boundary, measuring **Flip Rate** — the fraction steered to the ground-truth answer.

The final experimental suite (Blocks A–D) includes a **Cross-Problem Control** and a **sparse 18-layer mechanistic map** with transition pinning, providing publication-grade causal evidence.

## 📂 Repository Structure

```
cot/
├── final/                       # ⭐ Canonical results — start here
│   ├── notebooke66c37d069.ipynb  # Executed final notebook (Blocks A–D)
│   ├── final_nb_img_40_1.png    # Per-layer commitment curve
│   ├── commitment_raw_decomposition.png
│   └── results (1).zip          # Exported JSON results from Kaggle run
├── notebooks/
│   ├── v1_prototype_gsm8k/      # Early GSM8K prototypes (superseded)
│   ├── v2_math_development/     # MATH migration & T&G paradigm development
│   └── v3_current/              # Late-stage iterations (superseded by final/)
└── docs/                        # Comprehensive technical documentation
```

> **`final/` supersedes all notebook versions.** The `notebooks/` folders preserve the research iteration history.

## 📖 Deep Dive Documentation

1. [Theory and Architecture](docs/01_theory_and_architecture.md)
2. [Methodology and Codebase](docs/02_methodology_and_codebase.md)
3. [Results and Future Directions](docs/03_results_and_future_directions.md)
4. [Deep Logical Audit & Final Architecture](docs/04_deep_logical_audit_and_final_architecture.md)
5. [Experiment Evolution History](docs/05_experiment_evolution_history.md)
6. [Conference Paper Outline](docs/06_paper_outline.md)
7. [**Final Methods & Results (canonical)**](docs/07_final_methods_and_architecture_deep_dive.md)
8. [Draft: Abstract & Results](docs/08_paper_draft_abstract_and_results.md)

## ⚙️ Reproducibility

To reproduce the final functional rift finding:

1. Ensure access to a GPU with at least 16GB VRAM (e.g., NVIDIA T4) to load Qwen2.5-14B-Instruct in NF4 quantization.
2. Open `final/notebooke66c37d069.ipynb`.
3. Run all cells sequentially. The notebook is self-contained.
4. Full execution of Blocks A–D takes approximately 1.5–2 hours on a single T4.

---
*This research investigates the mechanistic interpretability of latent reasoning in large language models.*
