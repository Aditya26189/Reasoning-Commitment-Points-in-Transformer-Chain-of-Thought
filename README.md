<div align="center">
  <h1>🧠 Chain-of-Thought Commitment Points</h1>
  <p><em>Mechanistic Interpretability of Latent Reasoning in Large Language Models</em></p>
  
  <a href="https://huggingface.co/Qwen/Qwen2.5-14B-Instruct"><img src="https://img.shields.io/badge/Model-Qwen2.5--14B--Instruct-blue.svg" alt="Model"></a>
  <a href="https://huggingface.co/datasets/DigitalLearningGmbH/MATH-lighteval"><img src="https://img.shields.io/badge/Dataset-MATH_Level_4--5-green.svg" alt="Dataset"></a>
  <a href="#"><img src="https://img.shields.io/badge/Method-Truncate_%26_Generate-orange.svg" alt="Methodology"></a>
</div>

<br/>

## 📖 Overview

When a language model reasons through a multi-step math problem, which parts of the CoT trace carry causal weight over the final answer — and at which layer does that causal influence lock in? This repository investigates the residual-stream commitment structure of Qwen2.5-14B-Instruct via macroscopic activation patching on MATH Level 4–5.


By employing a macroscopic activation patching technique called **"Truncate & Generate"**, designed to handle variable-length CoT traces without token alignment, we bypass the token alignment destruction typical of standard patching and successfully map the sequence-level and layer-level causality of mathematical reasoning on Qwen2.5-14B-Instruct.

## 📊 Key Results (Final Run)

Executed in `final/notebooke66c37d069.ipynb` on **N=11 committed-wrong trace pairs** from MATH Level 4–5 (collected from 19 candidates; 7 excluded as self-correcting at truncation boundary, 1 skipped):

| Experiment | Flip Rate | Detail |
| :--- | :---: | :--- |
| **Block A — Same-Problem Patching** | **54.5%** | 6/11 committed-wrong traces steered to GT via `computation_only` activations |
| **Block B — Cross-Problem Control** | **23.1%** | 3/13 when activations come from a *different* math problem |
| **Specificity Gap** | **+31.4 pp** | Same-problem steering is 2.4× more effective than cross-problem control |

After excluding traces that self-corrected without patching ("already flipped"), **more than half** of genuinely committed-wrong trajectories can be causally redirected by injecting correct computational hidden states. The cross-problem control is directionally consistent with **problem-specific semantic steering**, not generic attention disruption.

Note: Blocks A and B used slightly different eligible pair sets (n=11 vs n=13) due to different baseline outcomes under same-problem vs. cross-problem T&G — the cross-problem null activations inhibit natural recovery in 2 pairs; see docs/07. A paired analysis on 10 shared traces shows a directional 40pp gap (continuity-corrected McNemar's test, b=6, c=2, χ²=1.125, p≈0.29; underpowered at n=10). See [docs/07](docs/07_final_methods_and_architecture_deep_dive.md) for full limitations.

## 🚀 Key Discovery: The Functional Rift

Our most significant finding is the empirical mapping of a **"Functional Rift"** across the transformer stack via a sparse 18-layer sweep. By causally intervening on `computation` semantic tokens at each tested layer independently, we discovered a distinct tripartite architecture of commitment:

<div align="center">
  <img src="final/final_nb_img_40_1.png" alt="Per-Layer Commitment Curve showing the Functional Rift" width="800"/>
  <br><small><em>Note: Full 18-layer commitment curve (Blocks C + D), including dense transition pinning at layers 30–35.</em></small>
</div>

1. **The Plateau (Layers 0–28): Semantic Steering**
   - Intervening on computational tokens in early/mid layers flips the terminal answer **36–45%** of the time (n=11 per layer). These layers act as an open scratchpad, assembling algebraic meaning that can redirect the trajectory.
2. **The Transition Zone (Layers 28–36, tested: 28, 30, 31, 32, 33, 34, 35, 36): The Collapse**
   - Flip rate falls from **36%** at layer 28 to **9%** at layer 36 across the tested layers — the commitment collapse localizes to this region of the stack.
3. **The Cliff (Layers 44, 47): Causal Locking**
   - Flip rate hits a hard **0%** (0/11 at layers 44 and 47, the only late layers tested). Even injecting correct mathematical reasoning into the residual stream cannot redirect the output. Late layers appear causally locked onto the original wrong answer — though this could also reflect precision degradation under NF4 quantization at depth, which cannot be ruled out without a full-precision control.

This expands upon the bipartite theory of transformer layer function described by Dutta et al. (*"How to think step-by-step: A mechanistic understanding of chain-of-thought reasoning"*, Llama-2 7B) by identifying a distinct tripartite structure for complex multi-step mathematical reasoning on Qwen2.5-14B.

## 🔬 Methodology: Truncate & Generate

Traditional activation patching suffers from catastrophic alignment failure when dealing with variable-length CoT reasoning traces. We solved this by operating at the macro-sequence level:

1. **Semantic Segmentation:** Parse reasoning traces into strict functional masks (`setup`, `computation`, `transition`, `conclusion`).
2. **Truncation:** Truncate a trace leading to a *wrong* answer ($T_{wrong}$) immediately after a computational step.
3. **Injection:** Surgically overwrite hidden states at computation token positions with activations from the equivalent positional index in a *correct* trace ($T_{correct}$) sourced from either the same problem or a cross-problem control.
4. **Generation:** Allow the model to freely generate from that boundary, measuring **Flip Rate** — the fraction steered to the ground-truth answer.

The final experimental suite (Blocks A–D) includes a **Cross-Problem Control** and a **sparse 18-layer mechanistic map**, providing mechanistic pilot evidence.

## 📂 Repository Structure

```
cot/
├── final/                       # ⭐ Canonical results — start here
│   ├── notebooke66c37d069.ipynb  # Executed final notebook (Blocks A–D)
│   ├── notebook_final_pilot_cummulative.ipynb # Exploratory N=20 pilot (cumulative sweep)
│   ├── final_nb_img_40_1.png    # Per-layer commitment curve (Blocks C + D, 18-layer)
│   ├── final_nb_img_39_0.png    # Block A/B summary chart (from Kaggle export)
│   ├── commitment_raw_decomposition.png
│   └── results (1).zip          # Exported JSON + plots from Kaggle run (see below)
├── notebooks/
│   ├── v1_prototype_gsm8k/      # Early GSM8K prototypes (superseded)
│   ├── v2_math_development/     # MATH migration & T&G paradigm development
│   └── v3_current/              # Late-stage iterations (superseded by final/)
└── docs/                        # Comprehensive technical documentation
```

**`results (1).zip` contents:**

| File | Description |
|------|-------------|
| `results_main.json` | Block A: 6/11 flips, 7 already_flipped, 1 skip |
| `results_cross.json` | Block B: 3/13 flips, 5 already_flipped, 1 skip |
| `results_per_layer.json` | Merged C + Canary (sparse 13-layer subset) |
| `results_per_layer_full.json` | Merged C + D (18-layer subset) |
| `per_layer_computation_only.png` | Sparse sweep plot |
| `per_layer_pinned.png` | Block D transition pinning plot (layers 30–35) |
| `__results___files/__results___39_3.png` | (= `final_nb_img_39_0.png` in repo) |
| `__results___files/__results___40_3.png` | (= `final_nb_img_40_1.png` in repo) |
| `__huggingface_repos__.json` | Model download metadata (Qwen2.5-14B-Instruct) |

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
