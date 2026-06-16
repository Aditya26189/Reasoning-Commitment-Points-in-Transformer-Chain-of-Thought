# 04. Deep Logical Audit & Final Experimental Architecture

This document synthesizes the rigorous logical audit conducted on the methodology and details the final, highly controlled experimental architecture designed to yield top-tier publishable results.

---

## 4.1 The Truncate & Generate Logic Audit

Before scaling the experiment to $N=40$, a line-by-line logical audit was performed to ensure the Truncate & Generate methodology mathematically measures what we claim it measures: causal commitment. 

### 4.1.1 Correct Masking of Terminal Answers
**The Risk:** The most severe risk to the experiment's validity was the possibility of `computation_only` masking capturing the final output (e.g., `\boxed{-21}`). If the final answer token was classified as computation, the "patching" step would simply copy the correct answer into the wrong trace's output stream. This would guarantee a 100% flip rate artificially, invalidating the entire paper.

**The Audit Validation:** The `classify_line` algorithm was heavily audited and proven robust. By checking the `conclusion` regex patterns *first*, any line containing `\boxed{}` or `####` is strictly quarantined into the `conclusion` mask. Thus, the `computation_only` mask is pure: it contains only the algebraic intermediate reasoning steps.

### 4.1.2 Baseline Determinism
In the per-layer analysis (`run_per_layer`), the unpatched baseline is evaluated once per pair rather than recomputed across layers. The audit confirmed this is mathematically valid: because the truncation point, the input tokens, and the generation parameters (greedy decoding) are absolutely identical, the baseline is deterministic. Re-computing it per layer would waste massive compute without changing the result.

### 4.1.3 Trace Alignment Symmetry
When extracting activation tensors from the Correct Trace ($T_{correct}$) to patch into the Wrong Trace ($T_{wrong}$), the alignment must be mathematically sound. 
The audit confirmed that `store_pass` grabs activations using the absolute token index relative to the prompt: `c_p_len + w_rel`. This maps the relative position of the mathematical scaffolding in the wrong trace directly to the equivalent structural position in the correct reasoning trace.

---

## 4.2 The "Functional Rift" vs. Monotonic Decline

The initial $N=12$ per-layer patching scan revealed a specific shape in the Commitment Curve:
- **Layers 0–32**: A plateau of 25% – 42% flip rates.
- **Layers 36–40**: A collapse zone.
- **Layers 44–47**: A hard 0% flip rate.

### 4.2.1 Statistical Significance at Late Layers
A naive reading might dismiss the late-layer $0\%$ flip rate as noise due to the small $N=12$ sample. However, if the true latent flip rate were $30\%$, the probability of observing exactly zero flips across 12 trials is $0.7^{12} \approx 1.4\%$. A hard zero at these endpoints is statistically significant evidence of causal locking.

### 4.2.2 The Dutta et al. Alignment
We originally conceptualized this as a "monotonic decline," but the audit reframed this pattern as a **plateau followed by a cliff**. This shape maps perfectly onto the "functional rift" described in recent mechanistic interpretability literature (e.g., Dutta et al.).
- **The Plateau (Early/Mid Layers):** These layers are responsible for transmitting and assembling input-relevant semantic information. Patching here can steer the sequence.
- **The Cliff (Late Layers):** These layers operate past the functional rift; their sole job is to rigidly project the in-context prior into the terminal vocabulary distribution. Patching here fails because the model has already irrevocably committed to the trajectory.

### 4.2.3 Crucial Mechanistic Check (Causally Locked vs. Disrupted)
To definitively prove the Functional Rift hypothesis at layers 44 & 47, we must examine the literal text output generated when the model *fails* to flip:
1.  **Causally Locked:** The model outputs the *original wrong answer*. This proves the late layers are fully committed to the wrong trajectory regardless of injected reasoning.
2.  **Disrupted:** The model outputs mathematical garbage/nonsense. This implies the late-layer patching is simply destroying the attention heads.
*Checking the raw string outputs for late-layer non-flips is a mandatory step before finalizing the mechanistic claim.*

---

## 4.3 The Six-Block "Bulletproof" Architecture

Top-tier conferences demand ablation studies that preemptively destroy reviewer counter-arguments. The final Kaggle execution script has evolved from a single run into a highly rigorous 6-block suite:

1.  **Block A: Main Experiment ($N=40$)**
    - **Goal:** Establish the baseline semantic steering capability of `computation_only` patches from organic, same-problem traces.
2.  **Block B: Cross-Problem Control ($N=40$)**
    - **Goal:** To prove the flips in Block A aren't just "attention disturbance." By patching computation tokens from an entirely *different* math problem, we test if arbitrary activation spikes cause flips. If Block B $\approx 0\%$ while Block A $\approx 35\%$, we prove the steering is genuinely semantic.
3.  **Canary Block:**
    - **Goal:** A dynamic timing gate (running only layers 32 and 36 on $N=30$) to ensure the full per-layer sweep will not exceed the 2-hour Kaggle execution limit.
4.  **Block C: Full Per-Layer Scan ($N=30$)**
    - **Goal:** To map the exact location of the functional rift across all 48 transformer layers.
5.  **Block D & E: Shuffled and Quantization Controls (Optional/Offline)**
    - **Goal:** Block E randomly shuffles the positional indices of the patched tokens to prove that ordered algebraic logic matters. Block D tests whether the NF4 4-bit quantization weights are injecting artifactual noise.
6.  **Block F: Correct-to-Correct Retention**
    - **Goal:** To prove that the Truncate & Generate methodology doesn't inherently destroy the forward pass. Patching a correct trace with itself should yield $100\%$ accuracy.

### 4.3.1 Execution Order Dependency
The $N=40$ behavioral run (Blocks A & B) must be executed **first**. 
Because the segmentation bounds were recently tightened (the `\boxed{}` fix), the distribution of which tokens qualify as "computation" has shifted. This changes which pairs pass the threshold for analysis. The per-layer scan (Block C) is computationally expensive and must only be run *after* the behavioral baseline is mathematically confirmed on the new, clean distribution.
