# 03. Statistical Interpretation and Future Directions

This document provides a rigorous statistical and mathematical breakdown of the 20-pair pilot results, along with the precise architectures for future validation sweeps.

## 3.1 The Flip Rate Discontinuity
The primary empirical finding is the discontinuity in the Flip Rate between the `setup` and `+computation` stages.

- **Setup Stage**: $\mu = 0.0\%$, $n=15$
- **Computation Stage**: $\mu = 35.3\%$, $n=17$

Mathematically, the introduction of computational tokens provides a non-zero gradient vector pointing towards the Ground Truth terminal state. The hidden states within the computation prefix contain sufficient semantic momentum to steer ~35% of the generations out of their corrupted trajectories.

## 3.2 The Denominator Shift Artifact
A critical statistical confounder in the stage-wise analysis is the variance in the sample denominator $n$.

Traces that spontaneously achieve the correct terminal state without patching ("Already Flipped") must be excluded from the analysis, as their initial trajectory was never firmly committed to the wrong answer.
- At the `setup` stage, 4 traces were classified as "Already Flipped", yielding $n_{setup} = 20 - 1 - 4 = 15$.
- At the `+computation` stage, only 2 traces were "Already Flipped", yielding $n_{comp} = 20 - 1 - 2 = 17$.

This implies that 2 traces which were "easy" (i.e., highly prone to self-correction at the setup stage) were re-injected into the sample pool at the computation stage. This shifting $n$ mathematically conflates the stage-wise delta. If those 2 newly included traces also flipped at the computation stage, the absolute delta ($\Delta_{comp, setup}$) is artificially inflated.

## 3.3 Statistical Power and Transition "Noise"
The flip rate remained static between `+computation` and `+transition`:
- **Transition Stage**: $\mu = 35.3\%$, $95\% \text{ CI} = [18\%, 59\%]$, $n=17$

A naive heuristic interpretation concludes that discourse markers ("Therefore", "Next") carry $0.0$ informational weight in determining terminal states. However, from a rigorous statistical standpoint, a sample size of $n=17$ lacks the statistical power to resolve variances smaller than $\pm 20\%$. A true semantic delta of $5\%$ induced by transition tokens is strictly invisible within this confidence bound. The result is statistically *uninformative*, not definitively *null*.

## 3.4 Control Validation Blind Spots
The Control Group experiment (patching correct activations into correct traces) yielded a $100\%$ Retention Rate at $n=10$ for the `+computation` stage, cleanly validating that the Truncate & Generate methodology preserves forward pass integrity.

However, for the `setup` stage, $n=0$ (all 10 traces self-corrected). Consequently, the integrity of the Truncate & Generate methodology at the specific truncation boundary of the `setup` stage remains theoretically unvalidated.

---

## 3.5 Future Directions

The pilot experiment effectively isolates the macro-stage locus of commitment (the computation tokens). To elevate this finding to a rigorous mechanistic proof, the architecture must scale along two orthogonal vectors: **Statistical Resolution** and **Sub-Layer Localization**.

### Enhancing Statistical Resolution
To overcome the noise floors discussed in Section 3.3, the core dataset must be expanded.
Executing the pipeline on $N=80$ pairs will condense the bootstrap confidence intervals from $\sim\pm 20\%$ to $\sim\pm 8\%$. This resolution is mathematically sufficient to identify the existence (or non-existence) of micro-steering effects caused by discourse transition tokens.

### The Disturbance Control Protocol
To mathematically untangle true Semantic Commitment from the Disturbance Effect (see Section 1.6), we must implement an active disturbance baseline.

**Protocol Architecture:**
1. Truncate the Wrong Trace at index $k$.
2. Instead of utilizing the latent activations of $T_{wrong}[:k]$, inject a random activation tensor $R$ of identical shape `[batch, seq_len, hidden_dim]`, sampled from the exact norm distribution of the layer.
3. Compute the Flip Rate.

If the Random Disturbance Flip Rate tightly bounds the observed $35\%$ experimental Flip Rate, the commitment hypothesis is invalidated. If the Random Disturbance Flip Rate collapses to $0\%$, it mathematically proves that the $35\%$ recovery is driven by explicit semantic embeddings within the computational prefix.

### Per-Layer Mechanistic Sweep
The current Truncate & Generate methodology operates across the entire 48-layer stack simultaneously. To identify the precise internal structures orchestrating the commitment, we must implement a per-layer activation patching sweep.

**Algorithm:**
```python
def run_per_layer_sweep(trace_w, trace_c, stage_mask):
    for layer_idx in range(48):
        # 1. Cache clean activations from trace_c at layer_idx
        clean_cache = run_with_cache(trace_c)
        
        # 2. Patch trace_w with clean_cache ONLY at layer_idx
        patched_logits = run_with_hooks(
            trace_w, 
            hook_point=f"blocks.{layer_idx}.hook_resid_post", 
            hook_fn=patch_from_cache(clean_cache, mask=stage_mask)
        )
        
        # 3. Log delta probability of terminal answer
        log_prob(patched_logits)
```
If the delta probability spikes at specific mid-to-late layers (e.g., layers 20-35), it proves that the mathematical commitment is functionally localized, mapping directly to specific architectural depths in Qwen2.5-14B.
