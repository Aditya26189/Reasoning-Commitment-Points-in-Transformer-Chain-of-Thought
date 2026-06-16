# Forensic Documentation Audit Report

**Date:** 2026-06-16  
**Auditor scope:** Every factual claim in `README.md` and `docs/01`–`docs/08`, cross-checked against executed outputs in `final/notebooke66c37d069.ipynb`, `final/results (1).zip`, and pilot notebook `notebooks/v2_math_development/notebook_final_pilot.ipynb`.

**Source-of-truth hierarchy:**
1. `final/notebooke66c37d069.ipynb` executed stdout
2. `final/results (1).zip` JSON exports
3. `docs/07_final_methods_and_architecture_deep_dive.md`
4. All other docs (lower priority; several stale)

---

## 1. Executive Summary

### What is solid (notebook-proven)

The **core quantitative results are real and internally consistent** with the final notebook:

| Finding | Value | Status |
|---------|-------|--------|
| Block A same-problem flip rate | **54.5% (6/11)** | VERIFIED |
| Block B cross-problem flip rate | **23.1% (3/13)** | VERIFIED |
| Specificity gap | **+31.4 pp** (2.4× ratio) | VERIFIED |
| Per-layer table (18 measured layers) | Matches notebook + JSON | VERIFIED |
| Organic pair purity | 19/19 | VERIFIED |
| Token distribution averages | 36.2 / 43.8 / 16.0 / 4.0 % | VERIFIED |

This is a **legitimate mechanistic interpretability result**: more than half of committed-wrong traces can be causally steered by computation-token activations, with a clear layer-wise signature (plateau → collapse → late-layer zero).

### What is broken (must fix before external sharing)

| Issue | Severity | Impact |
|-------|----------|--------|
| **B1:** "Full 48-layer sweep" — only **18/48 layers** measured | CRITICAL | Overstates mechanistic coverage |
| **B2:** "Layers 44–47 cliff" — only **L44 and L47** tested | CRITICAL | Implies untested layers are 0% |
| **B3:** Block A (n=11) vs Block B (n=13) — **different pair sets** | CRITICAL | Specificity gap not paired comparison |
| **B4:** Causal locking "validated" — only 1-pair sanity check | CRITICAL | Overstates late-layer evidence |
| **B5:** Doc 01 describes wrong mechanism (no activation patching) | HIGH | Misleads reviewers |
| **Pilot stage mislabel in doc 03** | HIGH | Cites 35.3% for wrong stage name |

### Publication readiness verdict

| Category | Score |
|----------|-------|
| Core numbers (A/B flip rates, per-layer table) | **100%** |
| Statistical auxiliary math | **~98%** |
| Mechanistic sweep scope language | **~40%** |
| Cross-block comparability | **~60%** |
| Internal doc consistency | **~70%** |
| Reproducibility completeness | **~75%** |

**Overall:** Strong pilot with compelling causal evidence. **Not submission-ready** until P0 fixes (B1–B5) are applied. The work should not be undersold — but docs currently **oversell scope** in ways a sharp reviewer will catch.

---

## 2. Claim-by-Claim Verification Table

Legend: **PASS** = matches notebook/JSON exactly | **PARTIAL** = directionally true but imprecise | **FAIL** = contradicted by evidence

### README.md

| Line / Claim | Doc Value | Notebook / JSON | Verdict |
|--------------|-----------|-----------------|---------|
| N=19 organic pairs | 19 | `"Total pairs available: 19"`, `"Organic pairs: 19 / 19"` | **PASS** |
| Block A flip rate | 54.5% (6/11) | `total=11 flips=6 already_flipped=7 skip=1` | **PASS** |
| Block B flip rate | 23.1% (3/13) | `total=13 flips=3 already_flipped=5 skip=1` | **PASS** |
| Specificity gap | +31.4 pp | 54.5 − 23.1 = 31.4 | **PASS** |
| Effect ratio | 2.4× | 54.5/23.1 = 2.359 | **PASS** |
| 25 pp design threshold exceeded | Yes | Notebook checklist: `"should be >25pp"` | **PARTIAL** — design note, not formal pre-registration (B7) |
| Plateau layers 0–28, 36–45% | Range | L0–L28 measured (sparse): 36.4–45.5% | **PASS** (for measured layers) |
| Transition L30→L35, 27%→9% | 27.3%→9.1% | Block D stdout | **PASS** |
| Cliff L44–47, 0% (0/11) | 0% | L44=0/11, L47=0/11 only | **PARTIAL** — L45–46 never tested (B2) |
| "Full 48-layer mechanistic sweep" | Implied full | 18/48 layers measured | **FAIL** (B1) |
| "no prior CoT intervention study has achieved" | Superlative | Unverifiable from repo | **FAIL** (B8) |
| `final/` file listing | 4 files + zip | Directory listing matches | **PASS** |
| `final_nb_img_39_0.png` | Not listed | File exists (35 KB) | **PARTIAL** (B12) |
| Blocks A–D runtime ~1.5–2 h | Estimate | Not timed end-to-end in notebook | **PARTIAL** — plausible, not measured |

### docs/03_results_and_future_directions.md

| Claim | Verdict | Notes |
|-------|---------|-------|
| Block A 54.5% (6/11) | **PASS** | |
| Block B 23.1% (3/13) | **PASS** | |
| 31.4 pp gap, 2.4× | **PASS** | |
| $0.7^{11} \approx 2.0\%$ | **PASS** | 0.01977… |
| 4× reduction L28→L35 | **PASS** | 36.4/9.1 = 4.0 |
| CI ±30% at n=11 | **PASS** | SE ≈ 29.4% |
| Pilot setup 0.0%, n=15 | **PASS** | `notebook_final_pilot.ipynb` stdout |
| Pilot computation_only 35.3%, n=17 | **FAIL** | Pilot shows `computation_only` **57.1% n=14**; 35.3% is **`+computation` n=17** |
| "Pre-registered 25 pp" | **PARTIAL** | B7 |
| Blocks C+D "all 48 layers" | **FAIL** | B1 |

### docs/04_deep_logical_audit_and_final_architecture.md

| Claim | Verdict | Notes |
|-------|---------|-------|
| Block A/B results | **PASS** | |
| Segmentation 4/4 passed | **PASS** | Check 2 stdout |
| Causal locking validated via baselines | **FAIL** | B4 — one-pair sanity check only |
| Block C "functional rift across all 48 layers" | **FAIL** | B1 |
| Canary 11.3 min, Block D 18.4 min | **PASS** | |

### docs/07_final_methods_and_architecture_deep_dive.md

| Claim | Verdict | Notes |
|-------|---------|-------|
| Full per-layer table | **PASS** | Matches JSON exactly |
| Block A/B tables | **PASS** | |
| Token distribution % | **PASS** | Notebook stdout lines 2141–2144 |
| collect_pairs_v3 4-phase pipeline | **PASS** | Matches notebook code |
| Block D layers [30,31,33,34,35] | **PASS** | |
| Table annotation "tight 4-layer window" | **PARTIAL** | B16 — 6 indices in 30–35 |

### docs/01, docs/02, docs/08 (stale / orphaned)

| Doc | Key Issue | Verdict |
|-----|-----------|---------|
| 01 §1.5 | Describes prefix replacement, not hook patching | **FAIL** (B5) |
| 01 §1.7 | Retention control described as implemented | **FAIL** (B15) |
| 02 §2.1 | Two-phase collect_pairs (missing pre-screen) | **FAIL** (B6) |
| 08 | Full abstract/results draft; repeats B1/B2 | **PARTIAL** — good content, wrong sweep language; not in README index (B9) |

---

## 3. Code-to-Claim Traceability

### How Block A measures 54.5%

```
run_experiment(pairs, cross_problem=False)
  → for each pair: baseline generate from truncated wrong trace
  → if baseline already matches GT: already_flipped += 1 (excluded)
  → else: patch ALL 48 layers at computation token positions from same-problem correct trace
  → measure if output matches GT → flip
```

**Notebook output:** `computation_only: total=11 flips=6 skipped=1 already_flipped=7`

### How Block B measures 23.1%

Same as Block A but `cross_problem=True`:
- Activation source: `pairs[(pi + len//2) % len]` — different problem's correct trace
- **Critical:** `cumul` masks are **NOT clipped** to `c_trace_len` when `cross_problem=True` (notebook ~L1771)
- Cross-problem hook uses modulo alignment: `c_abs_pos = c_p_len + (w_rel % max(1, src.shape[0] - c_p_len))`

**Notebook output:** `computation_only: total=13 flips=3 skipped=1 already_flipped=5`

### How per-layer sweep works

```
run_per_layer(pairs, stage="computation_only", subset_layers=[...])
  → baseline check (same as Block A, always clips to c_trace_len)
  → for each layer_i in subset_layers: patch ONLY that layer, generate, check flip
```

**Layers actually executed:**

| Block | Layers | Count |
|-------|--------|-------|
| Block C (sparse) | 0, 4, 8, 12, 16, 20, 24, 28, 40, 44, 47 | 11 |
| Canary | 32, 36 | 2 |
| Block D (pinning) | 30, 31, 33, 34, 35 | 5 |
| **Total unique** | — | **18 of 48** |

**Never measured:** 1–3, 5–7, 9–11, 13–15, 17–19, 21–23, 25–27, 29, 37–39, 41–43, 45–46

### JSON export confirmation (`final/results (1).zip`)

All JSON values match notebook stdout exactly:

**results_main.json:** `flips=6, total=11, already_flipped=7, skip=1`  
**results_cross.json:** `flips=3, total=13, already_flipped=5, skip=1`  
**results_per_layer_full.json:** 18 layers, all rates match doc 07 table

---

## 4. Statistical Analysis (Gap Fill)

### Wilson 95% confidence intervals

| Block | k/n | Rate | Wilson 95% CI |
|-------|-----|------|---------------|
| A (same-problem) | 6/11 | 54.5% | **[28.0%, 78.7%]** |
| B (cross-problem) | 3/13 | 23.1% | **[8.2%, 50.3%]** |

CIs overlap substantially — the difference is **directionally clear but not precisely estimated**.

### Fisher's exact test (Block A vs B, independent proportions)

Contingency table (success / failure):

| | Success | Failure | Total |
|---|---------|---------|-------|
| Block A | 6 | 5 | 11 |
| Block B | 3 | 10 | 13 |

| Test | Statistic | p-value |
|------|-----------|---------|
| Two-sided Fisher's exact | OR = 4.0 | **p = 0.206** |
| One-sided (A > B) | OR = 4.0 | **p = 0.122** |

**Interpretation:** At α=0.05, the difference is **not statistically significant** with these sample sizes. The 31.4 pp gap is a **large effect size** on **non-identical pair sets** — report as directional evidence, not confirmed significance.

### Paired analysis on intersection of eligible pairs

Eligible pair indices from `flip_list` in JSON:

- Block A: {0, 1, 4, 5, 6, 7, 10, 12, 14, 17, 18} → 11 pairs
- Block B: {0, 1, 3, 4, 5, 6, 10, 12, 13, 14, 15, 17, 18} → 13 pairs
- **Intersection:** {0, 1, 4, 5, 6, 10, 12, 14, 17, 18} → **10 pairs**

On these 10 shared pairs:

| Block | Flips | Rate |
|-------|-------|------|
| A | 6 (pairs 4,5,10,14,17,18) | **60.0%** |
| B | 2 (pairs 0,1) | **20.0%** |
| Gap | — | **+40.0 pp** |

Paired sign test: 4 pairs A flips & B doesn't, 0 pairs B flips & A doesn't, 6 ties → **p = 0.125** (binomial, one-sided). Still underpowered but gap widens on paired subset.

### Late-layer zero significance

If true rate = 30%, P(0 flips in 11 trials) = $0.7^{11}$ = 0.01977 ≈ **2.0%** (one-sided). **PASS** for L44 and L47 individually.

### Per-layer CI width

At n=11, p≈0.45: 95% CI half-width ≈ **±29.4%**. Layer-to-layer differences within plateau or transition zones are **not individually significant**.

---

## 5. Pilot Study Verification (N≈20)

**Source:** `notebooks/v2_math_development/notebook_final_pilot.ipynb` executed stdout.

| Stage | Pilot Rate | Pilot n | Already flipped | Doc 03 §3.6 Claim | Match? |
|-------|------------|---------|-----------------|-------------------|--------|
| `setup` | 0.0% | 15 | 4 | 0.0%, n=15 | **PASS** |
| `computation_only` | **57.1%** | **14** | 4 | *(not cited)* | — |
| `+computation` | **35.3%** | **17** | 2 | 35.3%, n=17 labeled as `computation_only` | **FAIL** — wrong stage name |
| `+transition` | 35.3% | 17 | 2 | *(not cited)* | — |

**Finding:** Doc 03 attributes the pilot 35.3% rate to `computation_only`, but the pilot notebook shows:
- `computation_only` = **57.1% (8/14)** — closer to final 54.5%
- `+computation` (cumulative stage) = **35.3% (6/17)** — what doc 03 actually cites

The narrative "pilot 35.3% → final 54.5% strengthened" is **misleading**. The comparable stage (`computation_only`) went from **57.1% → 54.5%** (stable). The final run added cross-problem controls and per-layer mapping — that's the real upgrade.

---

## 6. Error Register (B1–B17)

### CRITICAL

**B1 — "Full 48-layer mechanistic sweep" is false**  
- **Where:** README L58; docs 03, 04, 05, 06, 08  
- **Evidence:** Only 18 layers measured (see §3 table)  
- **Fix:** "Sparse 18-layer mechanistic map across the 48-layer stack"

**B2 — "Layers 44–47 cliff" overstates coverage**  
- **Where:** README L44–45; docs 03, 06, 08  
- **Evidence:** Only L44 and L47 tested (both 0/11); L45–46 never run  
- **Fix:** "Hard 0% at layers 44 and 47 (the only late layers tested)"

**B3 — Cross-problem denominator asymmetry**  
- **Where:** All docs comparing Block A vs B  
- **Evidence:** n=11 vs n=13; different `already_flipped` (7 vs 5); `cross_problem=True` skips `c_trace_len` clipping in `run_experiment` (~L1771)  
- **Fix:** Document as limitation; report paired intersection analysis (60% vs 20% on n=10); ideally re-run Block B with clipping enabled

**B4 — Causal locking validation overstated**  
- **Where:** docs/04 §4.2.3; docs/06 §4.3; docs/08 §4.3  
- **Evidence:** Sanity check on Pair 0 only; no late-layer failure text audit  
- **Fix:** "Consistent with locking" or add output audit cell for L44/L47 failures

### HIGH

**B5 — Doc 01 §1.5 wrong mechanism** — prefix replacement vs hook-based activation patching  
**B6 — Doc 02 outdated collect_pairs phases** — missing pre-screen phase  
**B7 — "Pre-registered" 25 pp** — internal design note only  
**B8 — README unverifiable superlative** — "no prior CoT study has achieved"  
**Pilot stage mislabel** — doc 03 §3.6 cites `+computation` as `computation_only`

### MEDIUM

**B9 — Doc 08 orphaned** (not in README index)  
**B10 — Doc 05 "mapped all 48 layers"**  
**B11 — Dutta et al. no full citation** — paper: *"How to think step-by-step: A mechanistic understanding of chain-of-thought reasoning"* (Llama-2 7B, fictional ontologies, ~layer 16 rift); your work differs in model, task, layer range  
**B12 — `final_nb_img_39_0.png` undocumented**  
**B13 — `results (1).zip` not indexed in docs** (indexed in this report §7)  
**B14 — Notebook metadata stale** (header N=40, comments N=40, `N_PAIRS=19`)  
**B15 — Doc 01 §1.7 retention control** not executed (Blocks E/F)

### LOW

**B16 — "4-layer" vs "6-layer" transition window** inconsistency in doc 07 table vs doc 08 text  
**B17 — pairs.json external provenance** not documented (`/kaggle/input/.../pairs.json`, 20 loaded, 1 fake-wrong removed)

---

## 7. `final/results (1).zip` Contents Index

| File | Size | Description |
|------|------|-------------|
| `results_main.json` | 552 B | Block A: 6/11 flips, 7 already_flipped, 1 skip |
| `results_cross.json` | 630 B | Block B: 3/13 flips, 5 already_flipped, 1 skip |
| `results_per_layer.json` | 2.2 KB | Block C + Canary only (layers 0,4,8,12,16,20,24,28,32,36,40,44,47) |
| `results_per_layer_full.json` | 3.1 KB | Merged C + Canary + D (all 18 measured layers) |
| `per_layer_computation_only.png` | 58 KB | Sparse sweep plot |
| `per_layer_pinned.png` | 71 KB | Transition pinning plot |
| `__results___files/__results___39_3.png` | 35 KB | (= `final_nb_img_39_0.png` in repo) |
| `__results___files/__results___40_3.png` | 45 KB | (= `final_nb_img_40_1.png` in repo) |
| `__huggingface_repos__.json` | 810 B | Model download metadata (Qwen2.5-14B-Instruct) |

---

## 8. Missing Content from Prior Chats

| Item | Status |
|------|--------|
| N=19 as canonical | Done |
| Cross-problem 23.1% (not ~0%) | Done |
| Doc 07 as canonical reference | Done |
| Remove `jounels/` from README | Done |
| `final/` supersedes notebooks | Done |
| 2.0% rounding for $0.7^{11}$ | Correct |
| **This AUDIT_REPORT.md** | **Done (this file)** |
| Link doc 08 in README | Not done |
| Document zip contents | Done (§7) |
| Document `final_nb_img_39_0.png` | Not done |
| Fix doc 01 mechanism | Not done |
| Sync doc 02 collect_pairs | Not done |
| Cross-problem asymmetry limitation | Not done |
| "Sparse 18-layer" language | Not done |
| Dutta et al. full bibliography | Not done |
| Notebook header N=40 → N=19 | Not done |
| Fisher's exact / Wilson CIs in docs | Done in this report; not in docs 03/07 |

---

## 9. Correction Priority Matrix

### P0 — Before public sharing or any submission

| ID | Action |
|----|--------|
| B1, B2 | Replace "full 48-layer sweep" with "sparse 18-layer map"; narrow cliff claim to L44/L47 |
| B3 | Add limitation paragraph on cross-problem denominator asymmetry; cite paired n=10 analysis |
| B4 | Soften causal-lock validation language |
| B5 | Rewrite doc 01 §1.5 to describe hook-based activation patching |
| Pilot | Fix doc 03 §3.6 stage naming (`computation_only` 57.1% n=14, not 35.3%) |

### P1 — Before paper submission

| ID | Action |
|----|--------|
| B6–B11 | Sync doc 02, add Dutta citation, link doc 08, fix doc 05 |
| Stats | Add Wilson CIs and Fisher's exact to doc 03 or 07 |
| B13–B14 | Index zip in README; fix notebook N=40 metadata |
| Limitations | Add consolidated limitations section to doc 07 |

### P2 — Polish

| ID | Action |
|----|--------|
| B12, B15–B17 | Document all PNGs, note pairs.json provenance, harmonize transition window wording |

---

## 10. Suggested Limitations Paragraph (paste-ready)

> This study evaluates causal steering on **N=19 organically sampled correct/wrong trace pairs** from MATH Level 4–5, with **n=11 committed-wrong traces** entering the per-layer analysis after excluding traces that self-corrected without patching. Per-layer flip rates are estimated at **18 of 48 transformer layers** (sparse sweep with transition pinning), each with a 95% confidence interval of approximately ±30 percentage points. The cross-problem control (23.1%, n=13) and same-problem main experiment (54.5%, n=11) used **slightly different eligible pair sets** due to asymmetric mask clipping in the cross-problem code path; a paired analysis on the 10-pair intersection yields 60.0% vs 20.0% (directional gap +40 pp, but underpowered for formal significance: Fisher's exact p=0.21). Results are reported for a **single model** (Qwen2.5-14B-Instruct, NF4) with regex-based semantic segmentation. Blocks E (shuffled position) and F (correct-to-correct retention) were designed but not executed in the canonical final run.

---

## 11. Completeness Verdict

**Is the documentation 100% correct?** **No.**

The **numbers that matter are correct** and reproducible from the notebook and JSON exports. The **narrative oversells** mechanistic coverage (48-layer sweep), causal-lock proof, statistical significance of the A/B gap, and pilot-to-final improvement story.

**The underlying science is real.** Fixing P0 items makes the documentation defensible without weakening the core claims: computation tokens are causal steering vectors (54.5%), cross-problem patching is substantially less effective (directional +31 pp gap), and a sparse layer map reveals plateau → collapse → late-layer zero at the only tested late layers.

---

*End of audit report. No changes were made to README or docs 01–08 per user instruction. Request P0 fixes when ready to proceed.*
