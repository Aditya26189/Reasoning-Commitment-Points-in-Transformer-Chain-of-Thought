# 02. Methodology and Codebase Architecture

This document details the algorithmic pipeline utilized to execute the Truncate & Generate experiment, focusing on data ingestion, cumulative stage masking, and the rigorous parsing architecture required for mathematical reasoning evaluation.

## 2.1 Trace Pairing Pipeline (`collect_pairs_v3`)
The `collect_pairs_v3` module is responsible for mining the mathematical state space for divergent trace paths.

### Phase 1: Greedy Correct Anchor
- **Operation**: The model generates a trace at $T=0.0$ (greedy decoding).
- **Validation**: The terminal answer is extracted and compared to the dataset GT.
- **State**: If valid, this trace is cached as the `Correct Trace`. If invalid, the problem is skipped (as we cannot establish a ground-truth baseline path).

### Phase 2: Sampled Wrong Divergence
- **Operation**: The model is prompted with $T=1.2$ to $T=2.0$ (high entropy sampling).
- **Condition**: Generation continues in a loop (up to `greedy_budget`) until the terminal answer strictly *diverges* from the GT.
- **State**: The first divergent trace is cached as the `Wrong Trace`.

## 2.2 Stage Segmentation Heuristics
To evaluate commitment systematically, we cannot truncate at random indices. We utilize a regex-based heuristic engine (`get_stage_masks`) to classify every line of the trace into an architectural stage.

1. **`setup`**: Identified via absence of computational operators. Captures problem restatement and variable initialization.
2. **`computation`**: Identified via strict regex sets containing arithmetic operators (`+`, `-`, `\times`, `=`) and numeric transformations.
3. **`transition`**: Identified via linguistic discourse markers (`Therefore`, `Hence`, `Next`, `Step X`).
4. **`conclusion`**: Identified via structural markers (`####`, `\boxed`).

## 2.3 Cumulative Masking Logic
The `build_cumulative_masks` function computes the monotonically increasing token index boundaries for truncation.

Given a trace string, it is tokenized into `w_ids`. The line-level stage classifications are projected onto the token indices.
- **Stage 1 (`setup`)**: The truncation index $k$ is set to the last token belonging to a `setup` line *before* the first `computation` line.
- **Stage 2 (`+computation`)**: $k$ is set to the last token of the final `computation` line *before* the `conclusion`.
- **Stage 3 (`+transition`)**: $k$ includes all `setup`, `computation`, and `transition` lines up to the `conclusion`.

### Exclusion of the Conclusion Stage
The `conclusion` stage is strictly excluded from the experimental sweep. By definition, the conclusion stage encapsulates the terminal answer string. Truncating *after* the conclusion provides the generation pass with $0$ actionable tokens to alter the outcome, reducing the intervention to a trivial identity function.

---

## 2.4 Data Auditing and Normalization Architecture
The transition from reasoning conceptually about tokens to evaluating mathematical truth requires a highly robust parsing and auditing architecture. The MATH dataset, unlike simpler integer-based datasets (e.g., GSM8K), encodes ground truths in complex LaTeX representations.

### The `extract_gt` Parser
The naive approach to extracting terminal answers relies on regex targeting the `####` delimiter. However, Qwen2.5-14B frequently encapsulates its final output within `\boxed{}` LaTeX commands.

A standard regex `\\boxed\{(.*)\}` is mathematically unsound due to the prevalence of nested braces (e.g., `\boxed{\frac{1}{2}}`). To resolve this, `extract_gt` employs a depth-aware stack parsing algorithm:
1. It locates the literal substring `\boxed{`.
2. It initializes a brace depth counter `d = 1`.
3. It iterates linearly through the string, incrementing `d` for `{` and decrementing for `}`.
4. The extraction terminates precisely when `d == 0`, guaranteeing mathematically contiguous capture regardless of inner fraction or matrix depth.

### The Normalization Pipeline (`normalize_answer`)
Mathematical equivalence testing requires aggressive string normalization to prevent "Fake-Wrong" trace classifications. A Fake-Wrong trace occurs when the model generates mathematically correct output that fails a strict string equality check against the GT.

The normalization pipeline performs sequentially ordered transformations:
1. **Whitespace & Symbol Stripping**: `str(s).strip().lstrip("$").replace(",", "")`
2. **Backslash Order Precision**: `s = s.replace('\\$', '').replace('\\%', '').replace('%', '')`
   - *Architectural Note*: The sequence of replacement is strictly causal. If `%` is replaced before `\%`, a stranded backslash `\` remains, corrupting the float cast and causing pipeline failure.
3. **Fraction Evaluation**: `re.sub(r'\\(?:d)?frac\{([^{}]+)\}\{([^{}]+)\}', r'\1/\2', s)`

### Context Window Boundary Limitations
A critical architectural parameter discovered during auditing was the interaction between `MAX_NEW_TOKENS` and the termination generation phase. 
Initially set to `512`, the context window constraint induced truncation *during* the generation of the computation stage for complex MATH problems. 

This resulted in `AssertionError: extract_gt returned None`, as the model literally ran out of compute budget before rendering the `####` delimiter. Elevating the tensor allocation to `MAX_NEW_TOKENS = 2048` provided sufficient operational headroom for the 14B parameter model to unroll its latent reasoning graphs to completion.
