# 05. The Evolution of the CoT Commitment Experiment (20 Iterations)

This document provides a chronological analysis of how the codebase and experimental methodology evolved across ~20 notebook iterations, from the early `notebookf786ef6636` series to the final `final/notebooke66c37d069.ipynb`.

This narrative is explicitly designed to support the "challenges overcome" and methodology sections of the final academic paper.

---

## Phase 1: Prototyping & Infrastructure (`notebookf786ef6636` series)

### 1. Shift from GSM8K to MATH Level 4-5
- **What Changed:** The dataset loader and prompts were upgraded from GSM8K to the Hendrycks MATH dataset (Level 4-5).
- **Why it Changed:** GSM8K problems are often solved by the model in 1-2 trivial steps, making it impossible to isolate "setup" from "computation." MATH Level 4-5 requires deep, multi-step reasoning (e.g., polynomial factoring, modular arithmetic). This provides longer traces and distinct functional stages necessary for high-resolution activation patching.

### 2. Fixing the Extraction Pipeline
- **What Changed:** The `extract_gt` function evolved from basic string splitting to a robust regex: `re.findall(r'####\s*\$?(-?[\d,]+(?:\.\d+)?)', text)`, and later incorporated `\boxed{}` extraction logic.
- **Why it Changed:** MATH dataset answers are wrapped in LaTeX formatting (e.g., `\boxed{x=5}`). Early notebooks failed to accurately extract the final answers, throwing away valid data. *Note: During this phase, an escaping error was identified where `r'\(\boxed\{'` mistakenly matched `(\boxed{`. The correct string `r'\\\(\\boxed\{'` was established, though eventually made redundant by priority-based line classification.*

### 3. Memory Leaks and OOM Crashes
- **What Changed:** Introduced aggressive memory management inside the patching loops: explicit hook removal (`h.remove()`), `_acts.clear()`, `del`, `gc.collect()`, and `torch.cuda.empty_cache()`.
- **Why it Changed:** Activation patching requires storing hundreds of megabytes of hidden states per layer. In early versions, running the notebook on a Kaggle T4 GPU would crash halfway through due to Out of Memory (OOM) errors.

---

## Phase 2: Methodological Shifts (`notebook84b96eb612` series)

### 4. The "Truncate & Generate" Paradigm
- **What Changed:** Instead of doing a standard forward-pass patch over the entire fixed sequence, the code was rewritten to truncate the prompt at specific functional boundaries, patch the activations up to that point, and then let the model generate the rest of the text.
- **Why it Changed:** Wrong traces and correct traces diverge quickly. Patching tokens after the divergence point breaks token semantic alignment. By truncating at the boundary and forcing generation, we effectively ask the model: *"Given this injected mindset, what do you decide to do next?"*

### 5. Semantic Stage Masks (`STAGES` vs `CUMUL_STAGES`)
- **What Changed:** Moved away from patching raw absolute token indices and implemented a semantic line-by-line classifier (`classify_line`) that categorizes text into `setup`, `computation`, `transition`, and `conclusion`.
- **Why it Changed:** Absolute token positions vary wildly between math problems. Semantic segmentation ensures that when we claim we are patching "computation," we are *only* patching mathematical reasoning tokens, regardless of their absolute index.

---

## Phase 3: Audits and Bug Squashing (Mid `notebook84b96eb612` series)

### 6. The BitsAndBytes Downgrade Bug
- **What Changed:** In the model loader, `bnb_4bit_compute_torch_dtype=torch.float32` was corrected to the actual valid key: `bnb_4bit_compute_dtype=torch.float16`.
- **Why it Changed:** The `transformers` library silently ignored the invalid key and fell back to `float32`. This caused the Qwen2.5-14B model to run with misaligned precision, causing massive numerical drift and slowing down generation by 30%. Fixing this stabilized the baseline.

### 7. Cross-Problem Alignment and Clipping
- **What Changed:** Added `c_trace_len` clipping to `get_stage_masks` and implemented modulo arithmetic for cross-problem patching.
- **Why it Changed:** When patching activations from a *different* problem (cross-problem ablation), the source trace might be shorter than the target trace. The early notebooks threw "Out of Bounds" index errors or fed unpatched wrong-trace tokens at the end of the context. Clipping ensured logical symmetry.

---

## Phase 4: Publication-Quality Rigor (Final `notebook84b96eb612` series)

### 8. The Silent Data-Drop Bug
- **What Changed:** Replaced `max_new_tokens=150` with `max_new_tokens=MAX_NEW_TOKENS` (2048) in all generation calls.
- **Why it Changed:** This was a critical logical flaw. When truncating early (at `setup`), the model needed 500+ tokens to finish the math problem. A hard cap of 150 tokens caused the model to cut off mid-thought, returning `None`. The script logged this as a baseline failure ("already_flipped") and threw the data away. Fixing this rescued the integrity of the early-stage metrics.

### 9. The Inverted Control Metric
- **What Changed:** Rewrote the "Flip Rate" metric for `target_key="correct"` to check `not answers_match(...)` instead of `answers_match(...)`.
- **Why it Changed:** In the NF4 artifact control experiment, we patch *wrong* acts into a *correct* trace. Outputting the Ground Truth (GT) means the model was robust, not that it "flipped". The old metric counted "stayed correct" as a "flip", inverting the axis.

### 10. The Ultimate Control Suite
- **What Changed:** The final notebook architecture expanded into a multi-block pipeline (Main, Cross-Problem, Per-Layer Sweep, Transition Pinning). Blocks A–D were executed in the canonical final run; shuffled-position and retention controls remain as designed extensions.
- **Why it Changed:** Top-tier conferences require bulletproof evidence. The executed blocks — especially the cross-problem control (+31.4 pp specificity gap) and sparse 18-layer mechanistic map — preempt the primary reviewer counter-arguments with hard causal data.

---

## Phase 5: The Final Resolution (`final/notebooke66c37d069.ipynb`)

### 11. The Definitive Run (Blocks A–D)
- **What Changed:** The experimental sequence culminated in the `final/` folder execution on **N=19 organic pairs** (100% temperature-sampled, zero injection fallbacks). Blocks A–D were executed: behavioral main experiment (54.5% flip rate), cross-problem control (23.1%, +31.4 pp specificity gap), sparse per-layer sweep, and transition pinning.
- **Why it Changed:** The pilot study established the directional signal for `computation_only` patching (exact rate pending verification of `final/notebook_final_pilot_cummulative.ipynb`), but lacked cross-problem controls and layer-level resolution. The final run tightened segmentation, expanded the ablation suite, and mapped the functional rift across a sparse 18-layer subset of the transformer stack — producing the functional rift finding (plateau → collapse at layers 30–35 → 0% lock at tested late layers).

### 12. Pinning the Transition (Block D)
- **What Changed:** Block D (Transition Pinning) mapped layers 30, 31, 33, 34, and 35 at high resolution, complementing the Block C sparse sweep and canary runs at layers 32 and 36.
- **Why it Changed:** The broad per-layer sweep identified a wide gap between layer 28 (36.4% flip rate) and layer 36 (9.1% flip rate). Block D proved the collapse is localized in the layers 30–35 window — the model irrevocably locks into its terminal decision across a tight mechanistic boundary.
