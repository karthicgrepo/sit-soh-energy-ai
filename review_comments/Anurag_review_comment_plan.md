# Review response plan: Dr Anurag Sharma, 21 September 2026

> ## STATUS AS OF 2026-09-29 (work done on the GPU machine, Linux RTX 2080 Ti)
>
> The plan below is the ORIGINAL pre-work triage. Most of it is now DONE. Read this
> box first, then `review_response.md` (point-by-point with final numbers).
>
> **New home of the paper:** `latex/energy_ai_paper/` (migrated to **elsarticle**, Energy and AI),
> pushed to `github.com/karthicgrepo/sit-soh-energy-ai` (branch `main`), compiled `main.pdf` = 66 pp.
> The old `latex/SIT_SOH_Paper/` (Wiley) is **FROZEN** as the reviewed version, do not edit it.
> **Code:** all fixes + experiment scripts + regenerated result CSVs are on branch
> `energy-ai-revision` of `battery-soh-prediction` (pushed).
>
> **DONE (22/25):** C1 (saturation-forward title/abstract/framing), C2 (cell-to-cell), C3 (softened),
> C4 (matched-exposure 7.6x cycle / 13.7x throughput), C5 (persistence 0.0128 beats hybrid, drift),
> C6 (fingerprint-conditioned transfer), C7 (intermittent-SOH sweep 0.015/0.055/0.114/0.202 at k=1/5/10/25),
> C8 (fixed budgets 30/50/100/200: +37/34/13/37%), C9 (dropped "irreducible"), **C10 (cell-disjoint
> pretraining fix + FULL re-run, both pretraining paths)**, C11 (PI not CI), C12+C13 (grouped/cross-conformal,
> selection cells excluded; raw MC 82%, grouped conservative to 100%), C14+C15 (fingerprint INERT =
> population prior, not cell-id; controls all ~0.0149), C16 (chemistry-alignment effect not principle),
> C18 (RUL censoring 12 EOL / 8 censored), C20 (full decoder spec), C21 (consolidated dataset table),
> C22 (Scratch-LSTM naming / NASA=LCO / PI / standardised terms), C23+C24 (already-done + pushed back).
>
> **KEY NUMBER CHANGES from the C10 leak fix (report before/after):** fingerprint-transfer 38.2% -> 23.3%,
> full hybrid 53.5% -> 51.1%, checkpoint-selection robustness 95.5% -> 100%, coverage now honest grouped
> conformal (old leaked 93.9% removed). Framing decision (user, 2026-09-28): capability-based win only,
> NOT one-step, aligned with C24. RUL numbers unaffected.
>
> **OPEN (3, laptop-doable, NO GPU / NO new data needed):**
> - **C17** literature update + cite Zhang 2026 (`zhang2026`, Energy and AI 25:100771) + add HUST/SNL bib
>   entries (verify real citations, do NOT fabricate).
> - **C19** risk-vs-coverage RUL plot (from `results/review/t53_benchmark_table.csv`).
> - **C25 / finalize:** abstract -> single paragraph <=250 words; cut manuscript 15-20%; decide TCN/Transformer
>   (matched search or drop).
>
> All experiment outputs: `battery-soh-prediction/results/{sit_paper,sensitivity,hybrid,review}/`.

**Manuscript:** "Deployment-Oriented Health Prediction for Large-Format LiFePO4 Cells: Manufacturing Variability, Transfer, and Calibrated Uncertainty"
**Source:** `latex/sit_soh_paper/` (Wiley NJD template, 26 pages, approx. 16,100 words)
**Review email:** `review_comments/RE Latest Battery dataset paper Scientific Data.msg` (subject line is a stale reply chain; the content is entirely about the SOH prediction paper, not the Scientific Data dataset paper)
**Attachment:** `review_comments/Reference_Paper.pdf` = Zhang, Diallo, Delpha, Benbouzid, "Cross-dataset battery life forecasting with time-series foundation models: From zero-shot to prefix-adapted", *Energy and AI* 25 (2026) 100771, DOI 10.1016/j.egyai.2026.100771

---

## 1. Headline verdict

I went in expecting to confirm your read that these are generic LLM-generated comments. **I cannot confirm that.** My assessment after checking each comment against the actual source files:

| Category | Count | Comments |
|---|---|---|
| Valid, requires new analysis or re-runs | 10 | 4, 5, 7, 8, 10, 12, 13, 18, 20, 21 |
| Valid, but wording, terminology or figure fixes only | 8 | 2, 3, 9, 11, 15, 16, 22, 25 |
| Partially valid; the body already does most of it, needs a pointer plus a small addition | 5 | 1, 6, 14, 17, 19 |
| Already addressed in the current draft; push back with quotes | 2 | 23, 24 |

So roughly **18 of 25 are solidly valid**, 5 are half-stale, and only 2 are genuinely wrong.

### Why I do not think these are boilerplate

Several comments cite details that cannot be hallucinated from a skim:

- **Comment 10** (pretraining validation split lets one MIT cell contribute overlapping windows to train and validation) targets a single buried sentence in [latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L168-L176). You disclosed it yourself; he read it and correctly identified it as a leakage attack surface.
- **Comment 22b** (Figure 12 labels NASA as NMC while the text says LCO) is real. `figures/fig8_coverage.png` is dated 27 July and still renders "NASA-NMC" and "MC-Dropout CI coverage", while [latex/sit_soh_paper/evaluation.tex](latex/sit_soh_paper/evaluation.tex#L206) says LCO. Verified by extracting text from `main_updated.pdf`.
- **Comment 9** ("irreducible within-cell variance") targets one sentence in [latex/sit_soh_paper/problem.tex](latex/sit_soh_paper/problem.tex#L182-L185).
- **Comment 12** (same cells used for checkpoint selection and conformal calibration) quotes the exact design in [latex/sit_soh_paper/evaluation.tex](latex/sit_soh_paper/evaluation.tex#L163-L172): "using the three calibration cells already reserved for checkpoint selection".

They may well have been drafted with LLM assistance, but they are **grounded in our text**. Treating them as boilerplate and replying with cosmetic edits is the fastest route to losing his support and, more importantly, to getting the same points back from a real reviewer at Journal of Energy Storage.

### The good news

Three of the most expensive-looking asks are **cheap or already done**:

1. **Comment 4 (matched-cycle variability): we already ran this and then lost it.** The 10 August 18:48 build (`main.pdf`) contains the sentence "it remains 7.7x when all cells are compared at a matched cycle count and 13.7x at matched energy throughput". That sentence is **not in the current sources** (`main_updated.pdf`, 20:19) but the abstract still quotes "7.7x even at the same cycle count". So right now we have an **unsupported number in the abstract**, and the fix is restoring a paragraph we already wrote, not new work. Recover it from git history or from `main.pdf`.
2. **Comments 23 and 24 are already implemented** and I can quote chapter and verse (Section 7 below).
3. **Comment 14's core claim is already in the paper.** We already ran the shuffled-fingerprint control and already concluded the population-prior reading. He is asking us to finish a job we started, not to reverse a conclusion.

---

## 2. Version confusion to resolve first

There are two compiled PDFs in `latex/sit_soh_paper/` and they differ:

| File | Built | Pages | Words | Matches current `.tex`? |
|---|---|---|---|---|
| `main.pdf` | 10 Aug 18:48 | 27 | 17,167 | No. Contains the matched-cycle-count analysis. |
| `main_updated.pdf` | 10 Aug 20:19 | 26 | 16,138 | Yes |

**Action:** confirm which PDF you emailed him on or before 21 September. If you sent `main_updated.pdf`, comment 4 is explained by the fact that we deleted the analysis. Either way, delete or rename the stale `main.pdf` so this cannot happen again.

---

## 3. Comment-by-comment triage

| # | Topic | Verdict | Cost | Priority |
|---|---|---|---|---|
| 1 | Reframe story around saturation; use his abstract | Partial: body already does this, abstract does not | Low | High |
| 2 | "manufacturing variability" to "cell-to-cell variability" | Valid | Low | High |
| 3 | Abstract dominance claim too strong | Valid | Low | High |
| 4 | 13.2x at final SOH is unfair; use matched cycle or throughput | Valid; **analysis already exists, restore it** | Very low | High |
| 5 | Add persistence and drift baselines | Valid | Low | High |
| 6 | "zero-shot" is misleading | Partial: already defined, but rename is right | Low | Medium |
| 7 | Is SOH actually observable every cycle in a BMS? | Valid and dangerous if unanswered | Medium | High |
| 8 | Fixed-cycle budgets (30/50/100/200 cycles) | Valid; strongest new experiment | Medium to high | High |
| 9 | "irreducible within-cell variance" is wrong | Valid, pure wording bug | Very low | High |
| 10 | Make pretraining split cell-disjoint | Valid; cheap fix, but forces re-runs | Medium | High |
| 11 | PI not CI; MC-Dropout is not calibrated by fiat | Partial: body is already correct, abstract and figures are not | Low | High |
| 12 | Separate checkpoint-selection cells from calibration cells | Valid; most serious methodological point | Medium | Critical |
| 13 | Residuals within a cell are correlated; use grouped conformal | Valid | Medium | Critical |
| 14 | More fingerprint controls (zero, mean, OOD) | Partial: shuffled control exists; add the rest | Low to medium | Medium |
| 15 | Drop "identifies the type of degrader" | Valid, one table cell | Very low | Medium |
| 16 | "chemistry alignment principle" too strong | Valid, terminology | Low | Medium |
| 17 | Update literature; cite attached paper | Valid | Low | High |
| 18 | RUL: right-censored cells not handled | Valid | Low to medium | High |
| 19 | Risk versus coverage plot for RUL | Partial: table and text already make the argument | Low | Low |
| 20 | Direct multi-horizon RUL model under-specified | Valid; reproducibility hole | Low | High |
| 21 | One consolidated dataset table | Valid; NASA is never formally described | Low | High |
| 22 | Figure and terminology inconsistencies | Valid, and worse than he noticed | Low | Critical |
| 23 | Do not claim 53.5% beats all methods | Already done; push back | None | n/a |
| 24 | Say the hybrid is not meant to win one-step | Already done; push back | None | n/a |
| 25 | Cut length by 15 to 20% | Partial: valid, but sequence it last | Medium | Medium |

---

## 4. Detailed assessment of the valid comments

### C2, C3: causal language around variability
**Valid.** [latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L18-L23) already hedges correctly ("most plausibly dominated by manufacturing tolerances, although cycler-channel differences, contact resistance, minor temperature variation, testing interruptions and production batch may also contribute"), but the rest of the paper then ignores that hedge: a section titled "Manufacturing Variability and the Two-Sided Failure Mode", a section titled "Early Detection of Manufacturing-Defective Cells", a contribution bullet "Manufacturing variability quantification", and an abstract that asserts "manufacturing variability, not temperature or model choice, dominates how these cells age".

That abstract sentence is indefensible as written: G2 is the only 40 C 1C group, there is no 2C ambient group, so temperature and C-rate are not independently isolated.

**Action:**
- Global rename to "cell-to-cell variability" in all results and claims. Keep "manufacturing variability" only where we explicitly discuss it as the *likely dominant* contributor, with the confound list attached.
- Rename Section 8 to "Early Detection of Fast-Degrading Cells".
- Replace the abstract dominance claim with the defensible version: cell identity explains 25 to 58% of prediction-error variance versus about 15% for method choice, within this dataset.

**Partial rebuttal worth including:** the two catastrophic degraders sit on **two different testers** (001-8 on Repower 001, 101-3 on Chroma 101). That weakens the "it is just cycler-channel differences" alternative and is worth one sentence.

### C4: variability at matched exposure
**Valid, and we already have the numbers.** Restore the matched-cycle-count (7.7x) and matched-throughput (13.7x) analysis into [latex/sit_soh_paper/problem.tex](latex/sit_soh_paper/problem.tex#L8-L18), next to the 13.2x figure. Add a small table or panel: final-cycle SOH, SOH at the minimum common cycle count across G1, and SOH at matched cumulative Ah throughput. Until this is restored the abstract cites a number the paper does not support.

### C5: persistence baseline
**Valid.** Table 3 in [latex/sit_soh_paper/problem.tex](latex/sit_soh_paper/problem.tex#L84-L110) has Linear AR (0.0068 / 0.0052 / 0.0052) and a cubic fit, but **no persistence**. Given that our whole saturation argument rests on "one-step from observed windows is a near-persistence task", omitting persistence is the single most obvious hole a reviewer will find. It costs no training time.

**Action:** add two rows, persistence (`y_hat_t = y_{t-1}`) and linear drift (`y_hat_t = y_{t-1} + mean recent delta`), at all three splits. If persistence lands near 0.005 to 0.007, it strengthens the paper considerably.

### C7: what SOH is actually observable in deployment
**Valid and the most likely source of a real reviewer rejection**, because the title says "Deployment-Oriented". We currently assume a true discharge capacity every cycle ([latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L150-L160): "at each held-out cycle the model receives the *observed* previous W SOH values"). A real BMS rarely gets a full-depth discharge every cycle.

**Action:** add a short subsection "Assumed onboard observability" stating explicitly what the model consumes, and defend it: periodic full-discharge reference tests, coulomb counting over qualifying deep discharges, or a partial-capacity SOH estimator. Then add a cheap robustness experiment: observe SOH every k-th cycle (k = 1, 5, 10, 25) and hold the prediction between observations. If accuracy degrades gracefully, this comment converts into a strength.

### C8: fixed-cycle budgets
**Valid; highest-value new experiment.** We already concede the point twice ([latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L146-L150) and limitation 8 in [latex/sit_soh_paper/discussion.tex](latex/sit_soh_paper/discussion.tex#L168-L176)) without running it. Note that 30/50/70% splits mean very different absolute budgets across cells (560 to 878 cycles).

**Action:** rerun the main comparison (persistence, linear AR, DE-LSTM, scratch, hybrid) at fixed budgets of the first 30, 50, 100 and 200 cycles. This is also the only honest setting in which the hybrid's early-life fingerprint should win, so it likely *helps* us.

### C9: "irreducible" variance
**Valid, pure wording bug.** [latex/sit_soh_paper/problem.tex](latex/sit_soh_paper/problem.tex#L182-L185) says mean R-squared 0.964 and then calls the residual 3.6% "irreducible within-cell variance". It is simply what one particular cubic fit fails to explain.

**Action:** "a cubic fit leaves 3.6% of within-cell trajectory variance unexplained". Delete "irreducible".

### C10: cell-disjoint pretraining split
**Valid.** [latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L168-L176) documents exactly one straddling cell contributing overlapping windows to both partitions. The leak is tiny but it is free ammunition for a reviewer.

**Action:** split the pretraining corpus by cell before windowing. This forces re-running Phase 1 pretraining, the five-seed checkpoint study, and everything downstream. Mechanical, but budget compute for it, and bundle it with the C12/C13 re-runs so the pipeline is only re-executed once.

### C11: PI versus CI
**Partially valid, and the remaining offenders are exactly two places.** The body is already correct and explicit ([latex/sit_soh_paper/method.tex](latex/sit_soh_paper/method.tex#L130-L135), [latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L186-L191)). The violations are:
- [latex/sit_soh_paper/abstract.tex](latex/sit_soh_paper/abstract.tex#L17): "calibrated confidence intervals".
- `figures/fig8_coverage.png`, axis label "MC-Dropout CI coverage" and "Mean CI coverage".

**Action:** fix the abstract; regenerate the figure.

### C12, C13: conformal calibration methodology
**Both valid, and jointly the most serious technical issue in the review.** Current design:
- The same three cells (001-1, 002-1, 002-5) are used to pick the pretrained checkpoint from five candidates **and** to compute the conformal quantile q95 ([latex/sit_soh_paper/evaluation.tex](latex/sit_soh_paper/evaluation.tex#L163-L172)).
- The conformal scores are pooled per cycle across those three cells, treating hundreds of strongly autocorrelated residuals as exchangeable samples.

Both inflate the reported 93.9% coverage. He is right on the statistics.

**Action (pick one, I recommend the second):**
1. Split into disjoint selection cells and calibration cells. Clean, but with 20 cells it costs us test cells.
2. **Leave-one-cell-out / cross-conformal calibration over the 17 test cells**, with the score aggregated at cell level (for example, one score per cell such as its residual quantile, or grouped conformal with cell as the group). Keeps all cells in the test set, and directly answers C13.

Also report the coverage *before and after* the fix; if it drops from 93.9% to, say, 90%, say so. That transparency is worth more than the number.

### C14, C15: what the fingerprint actually does
**Partially valid.** We already ran the shuffled-fingerprint control (0.0141 to 0.0146) and already drew the conservative conclusion, in three places: [latex/sit_soh_paper/method.tex](latex/sit_soh_paper/method.tex#L206-L212), [latex/sit_soh_paper/evaluation.tex](latex/sit_soh_paper/evaluation.tex#L50-L60), and the discussion. So we are **not** over-claiming in the way he implies.

What is genuinely inconsistent is Table 5 in [latex/sit_soh_paper/method.tex](latex/sit_soh_paper/method.tex#L180-L195), whose "Question answered" row still says "What type of degrader is this cell?".

**Action:**
- Fix that table cell (C15).
- Add the three missing control arms he asks for: zero initial state, population-mean fingerprint, out-of-distribution fingerprint (for example a NASA LCO cell's fingerprint). Cheap, and it converts a hand-wave into a proper mechanism study.
- Add the embedding visualisation he suggests (16-D fingerprint reduced to 2-D, coloured by final SOH and by cycle life). One figure, high persuasive value.

### C16: "chemistry alignment principle"
**Valid as terminology.** [latex/sit_soh_paper/evaluation.tex](latex/sit_soh_paper/evaluation.tex#L203-L210) already contains the confound caveat, but we then call it a "principle" in a section heading, a discussion heading, contribution bullet 5, and the conclusion. With one LCO dataset and one source-target direction for the calibration claim, "principle" is not earned.

**Action:** rename to "chemistry-alignment effect" throughout; keep "hypothesis" where it is used as motivation. Keep the evidence, soften the label.

**Ammunition to keep:** MIT to LISHEN (+80.2% zero-shot) holds chemistry constant while changing manufacturer, format, capacity and protocol. That is a real partial control in our favour and should be stated when we soften the claim.

### C17: literature update
**Valid.** The bibliography has 42 entries, 16 from 2024 onwards, but the transfer-learning subsection in [latex/sit_soh_paper/literature.tex](latex/sit_soh_paper/literature.tex#L71-L92) rests on `ye2021`, `ma2022`, `yu2026`, `finn2017` and `severson2019`. The 2024 to 2026 cross-dataset and foundation-model wave is missing.

The attached paper is directly adjacent and **helps us**:
- It is capacity-only cross-dataset RUL on NASA, CALCE, MIT and HUST, using the first 30% of life as history and recursively forecasting the remaining 70%. That is almost exactly our RUL setup.
- Its headline finding, "source-adapted transfer yields the best performance when source and target are from the same dataset, while cross-dataset transfer degrades accuracy", **independently corroborates our domain-alignment effect**.
- Its "zero-shot versus prefix-adapted" taxonomy is the cleanest existing vocabulary for the distinction we are trying to draw in C6. We should adopt it: "fingerprint-conditioned (prefix-adapted, no target fine-tuning)".

**Action:** add a "Cross-dataset and foundation-model transfer" paragraph; cite Zhang et al. 2026 plus 4 to 6 further 2024 to 2026 works; rewrite the novelty statement to be differential (large-format 50 Ah prismatic, per-cell thermal record, uncertainty calibration under chemistry mismatch) rather than "never been investigated".

### C18, C19, C20: RUL section
- **C18 valid.** From Table 1 in [latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L30-L70), only **11 of 20 cells** ever reach SOH 0.80; **9 are right-censored**. The paper says "every cell that reaches EOL" and "100% coverage" without ever reporting those counts, which makes "100%" read as 100% of 20. Report 11 EOL and 9 censored explicitly, and state how each method behaves on censored cells (does the decoder predict no crossing, or does it extrapolate one?).
- **C19 partially valid.** Table 8 already carries a coverage column and the text already makes his exact argument ("A method that withholds a forecast on the defective cells is of limited use for a manufacturing screen"). Adding a risk-versus-coverage curve is cosmetic but cheap and visually decisive. Low priority.
- **C20 valid.** [latex/sit_soh_paper/rul_forecasting.tex](latex/sit_soh_paper/rul_forecasting.tex) is 5.5 KB and specifies almost nothing: "a fixed set of future horizons up to 600 cycles" with no horizon grid, no loss function, no horizon weighting, no interpolation rule, no monotonicity constraint, and no statement of what happens when the predicted trajectory never crosses 0.80. Given our reproducibility claims elsewhere, this section is the weakest link in the paper. Expand it.

### C21: consolidated dataset table
**Valid, and it closes a real hole.** SIT and MIT are described in Section 4.1; LISHEN appears for the first time in Section 7.2; HUST and SNL appear in Section 9.2; **NASA is used in results but never formally described anywhere**.

**Action:** one table in Section 4 with columns: dataset, chemistry, format, nominal capacity, cells used, conditions, role in this study (pretraining / calibration / target / evaluation), source reference.

### C22: figure and terminology inconsistencies
**Valid, and the problem is bigger than he found.** Every figure PNG in `latex/sit_soh_paper/figures/` except four is dated 27 July, while the generator scripts in `battery-soh-prediction/figures_src/` have already been corrected. Confirmed by extracting text from `main_updated.pdf`:

| Figure | Renders | Text says | Status |
|---|---|---|---|
| `fig7_hybrid.png` (p17) | "Scratch DE-LSTM" | "Scratch LSTM" | Stale, C22a |
| `fig8_coverage.png` (p19) | "NASA-NMC", "NASA NMC" | NASA LCO | Stale, C22b |
| `fig8_coverage.png` | "MC-Dropout CI coverage" | PI | Stale, C22c |
| `fig8_coverage.png` | mean coverage 0.83 / 0.88 / 0.82 / **0.34** | Table 4: 80.3 / 79.0 / 87.1 / **43.1** | **He missed this. The figure contradicts the table.** |

`battery-soh-prediction/figures_src/generate_methodology_fig.py` even carries a comment noting a previous export had "stale claims: NMC, 34% coverage, 2.3x brittleness, 50 cells". So this is a known regression that was fixed in the scripts but never propagated to the manuscript.

**Action:** regenerate every figure from the current scripts and copy into `latex/sit_soh_paper/figures/`, then diff every number that appears in a figure against its table. Do this **first**; it is the cheapest credibility win in the whole list.

### C25: length
**Partially valid, but sequence it last.** 26 pages, approx. 16,100 words, is long for Journal of Energy Storage. The repetition he names is real: the saturation argument appears in the research gap, contribution 3, Section 5.1.1, discussion 10.2, and the conclusion.

But comments 5, 8, 12, 14, 18, 20 and 21 all **add** content. Cut only after the technical work lands, and target the duplicated argumentation (research gap versus contributions versus discussion) rather than the results.

---

## 5. Issues he did not raise that we should fix anyway

1. **The abstract cites 7.7x at matched cycle count, which appears nowhere in the current body.** Highest-embarrassment item in the paper right now.
2. **Figure 8 coverage numbers contradict Table 4** (see C22 above).
3. **TCN and Transformer are untuned** (RMSE 0.10 to 0.17, roughly 20x worse than linear AR). We flag the asymmetry in [latex/sit_soh_paper/setup.tex](latex/sit_soh_paper/setup.tex#L195-L206) and limitation 7, but a reviewer will still call it an unfair baseline. Either give them a comparable search budget or drop them and say so.
4. **Every number in the cross-dataset matrix (Table 7) is also "improvement over scratch"**, so C23's caveat applies there too and is currently unstated in that table's caption.
5. **Journal and template mismatch.** He suggests Journal of Energy Storage, Energy and AI, or Journal of Power Sources. **All three are Elsevier.** The manuscript is on `WileyNJDv5.cls`. Budget a template migration to `elsarticle.cls`, including table and figure environments and the bibliography style.
6. **Cross-reference to Paper 1.** The paper cites `ponnambalam2026sitdata` and a Figshare DOI. Confirm the Scientific Data submission status before this paper goes out, so the data citation resolves.

---

## 6. Where to push back, with evidence

Do not push back on the technical points. Do push back on these two, politely, by quoting the draft.

### C23: "do not present 53.5% as better than all competing methods"
Already done, in four places:
- Abstract: "fine-tuning raises this to 53.5% **over scratch** on all 17 test cells".
- Contribution 4: "achieves a 53.5% mean RMSE improvement **over scratch**".
- Table 6 column header: "Improvement **vs scratch**".
- [latex/sit_soh_paper/method.tex](latex/sit_soh_paper/method.tex#L228-L236) explicitly separates the scratch ablation control from the DE-optimised DE-LSTM baseline: "This is the ablation control for the hybrid and is distinct from the DE-optimised DE-LSTM baseline of Section 5.1; the two therefore differ in RMSE."
- The conclusion does not quote 53.5% at all.

**Suggested reply:** confirm it is already scoped to the scratch ablation in every occurrence, offer to add "ablation control" to the Table 6 caption for absolute clarity.

### C24: "state explicitly the hybrid is not meant to win the one-step benchmark"
Already done, prominently:
- [latex/sit_soh_paper/evaluation.tex](latex/sit_soh_paper/evaluation.tex#L95-L110): "The hybrid's decisive advantage is therefore not a marginal one-step number but a set of capabilities that no other method on the leaderboard provides."
- [latex/sit_soh_paper/discussion.tex](latex/sit_soh_paper/discussion.tex#L24-L40): "One-step RMSE is therefore a floor check, not a leaderboard."
- [latex/sit_soh_paper/conclusion.tex](latex/sit_soh_paper/conclusion.tex#L12-L15): "positioned not as the most accurate one-step predictor but as the one that supplies what deployment needs".

**Suggested reply:** point to these three passages and ask whether he wants it lifted into the abstract as well (which his own suggested abstract does, so the answer is probably yes).

### C1 (partial push back)
The body is already organised exactly as he asks: Section 5 is "Problem Characterisation: Limitations of Standard Methods", benchmark saturation is named contribution 3, and the discussion leads with it. What is genuinely misaligned is the **abstract**, which still leads with the variability-dominance claim he objects to in C3. Agree to rewrite the abstract and the introduction's opening; decline a full body restructure and explain why.

### C14 (partial push back)
State that the shuffled-fingerprint control is already in the paper and already led us to the weaker, population-prior interpretation, then agree to add the three further control arms he lists.

---

## 7. Proposed work plan

### Phase 0: hygiene, no new results (do this first, it is fast and removes most of the list)
- Regenerate all figures from `battery-soh-prediction/figures_src/` and copy into `latex/sit_soh_paper/figures/`. Resolves C22a, C22b, C22c.
- Audit every number that appears in a figure against its table. Resolves the Figure 8 versus Table 4 contradiction.
- Delete or archive the stale `main.pdf`.
- Global terminology pass: cell-to-cell variability (C2), PI not CI (C11, C22c), "chemistry-alignment effect" (C16), "fingerprint-conditioned transfer (no target-cell fine-tuning)" (C6), drop "irreducible" (C9), fix the "type of degrader" table cell (C15).
- Restore the matched-cycle and matched-throughput variability paragraph (C4).
- Rewrite the abstract along the lines he proposes, corrected for our actual numbers and with the dominance claim softened (C1, C3, C11, C23, C24).

**Outcome: C2, C3, C4, C6, C9, C11, C15, C16, C22, C23, C24 closed. Eleven of twenty-five, with no new experiments.**

### Phase 1: cheap additions
- Persistence and linear-drift baselines in Table 3 (C5).
- Consolidated dataset table, including a proper NASA description (C21).
- RUL censoring statistics: 11 EOL, 9 censored, per-method behaviour on censored cells (C18).
- Full specification of the direct multi-horizon decoder (C20).
- Literature update plus Zhang et al. 2026 and the differential novelty statement (C17).
- Risk-versus-coverage plot for RUL (C19).

### Phase 2: re-runs and new experiments (the real work; bundle into one pipeline execution)
- Cell-disjoint pretraining split, then re-run Phase 1 pretraining, the five-seed checkpoint study, and all downstream results (C10).
- Grouped or cross-conformal calibration with model selection separated from uncertainty calibration; report coverage before and after (C12, C13).
- Fixed-cycle-budget evaluation at 30, 50, 100, 200 cycles (C8).
- Intermittent-SOH-observation robustness study, k = 1, 5, 10, 25 (C7).
- Fingerprint control arms: zero state, population mean, out-of-distribution; plus the embedding visualisation (C14).

### Phase 3: consolidation
- Deployment-observability subsection (C7 write-up).
- Length reduction of 15 to 20%, targeting duplicated argumentation across research gap, contributions, discussion and conclusion (C25).
- Migrate to `elsarticle.cls` for the chosen Elsevier venue.
- Write the point-by-point response letter, using Section 6 above for the two push-backs.

---

## 7b. Compute impact: what actually needs re-running

Verified against the result CSVs in `battery-soh-prediction/results/`.

### No model execution at all (15 of 25 comments)

All wording, terminology and figure work is a re-plot from existing CSVs.

- **Key check:** `results/hybrid/hybrid_nasa_results.csv` already gives mean NASA coverage **0.4305**, which matches Table 4's 43.1%. The figure showing 0.34 is simply an older render that predates the deterministic re-run. **Regenerating the figures fixes C22 with zero training.**
- **C18 (RUL censoring) needs no re-run either.** `results/review/t52_foundation_rul.csv` already carries per-cell `actual_eol` and `reached`: **12 of 20 cells reach EOL, 8 are right-censored** (001-2, 001-5, 001-6, 001-7, 002-1, 002-2, 002-3, 002-5). Straight re-analysis.
- **C19 (risk versus coverage plot)** re-plots from `t53_benchmark_table.csv`.
- **C20** is documentation read off the existing script.
- Covers C1, C2, C3, C4, C9, C11, C15, C16, C17, C18, C19, C20, C21, C22, C25.

### Light runs, existing checkpoint reused, no pretraining (4 comments)

| Comment | What runs | Notes |
|---|---|---|
| C5 persistence and drift | None (pure NumPy over the SOH arrays) | Minutes |
| C14 fingerprint controls | Fine-tune plus MC-Dropout inference only; the encoder is frozen anyway | Mirrors the existing shuffled-fingerprint run, `t41_shuffle_control.csv` |
| C7 intermittent SOH observation | Inference-only sweep over k = 1, 5, 10, 25 | No retraining |
| C8 fixed-cycle budgets | Fine-tune plus scratch, 20 cells x 4 budgets x 5 seeds | The largest of this tier, still fine-tune scale |

### Post-hoc, but needs one inference pass to dump residuals (2 comments)

**C12 and C13.** Conformal recalibration is pure post-processing, but the saved CSVs only store summary columns (`hybrid_ci95`, `hybrid_coverage`), not per-cycle residuals and sigmas. So we need one MC-Dropout inference pass over the 20 SIT cells to dump per-cycle `(y, y_hat, sigma)`, then all the cross-conformal and grouped-conformal variants are re-analysis. No training.

### The one that cascades (1 comment)

**C10 (cell-disjoint pretraining split) is the only comment that invalidates the checkpoint.**

The fix itself is about ten lines in `pretrain_on_mit()` in [battery-soh-prediction/experiments/hybrid_solution.py](battery-soh-prediction/experiments/hybrid_solution.py#L184-L226): today it concatenates windows across all cells and then takes a tail slice, `vs = max(1, int(len(X_arr) * 0.1))`. It needs to partition `mit_cells` by cell id *before* windowing.

But changing it invalidates the released checkpoint, and therefore everything downstream:

- the five-seed pretraining variance study and the checkpoint-selection protocol (Section 6.3)
- the 1140-subset calibration-cell robustness claim (95.5%)
- the ablation, Table 6 (38.2 / 47.7 / 53.5%)
- main results, Table 5, and the per-cell figures
- coverage, Table 4 and Figure 8, and the 93.9% conformal result
- all ten cross-dataset scenarios, Table 7
- the RUL foundation stage, since its corpus nests the MIT cells

That is effectively the whole of Sections 6, 7 and 9. Every number in the paper has to be re-derived and re-checked against the text, and the Zenodo checkpoint and the byte-for-byte determinism claim both have to be reissued.

### Practical consequences

1. **Do Phase 0 first.** It closes 11 comments, needs no compute, and can be sent to him as a visible response while the rest is running.
2. **Bundle C10 with C12, C13, C7, C8 and C14 into a single pipeline execution.** Re-running the pipeline twice is the main avoidable cost here.
3. **This needs the GPU machine**, not the laptop. See `GPU_TASKS.md` and `INVENTORY_FROM_GPU.md`.
4. **Expect the headline numbers to move.** Removing the leak should move them slightly; honest, correct conformal calibration will likely pull the 93.9% coverage down. Plan to report the before and after rather than quietly restating.
5. **If C10 is deferred**, it must move into Limitations with the leak stated explicitly, in which case we keep the current numbers. I do not recommend this: it is a ten-line fix on a documented leak, and a Journal of Energy Storage reviewer who spots it after we were told about it is a much worse outcome.

---

## 7c. Where to run it: laptop versus GPU box

**Short answer: the compute is light, but Phase 2 must run on the Linux GPU box. The blocker is data and environment, not horsepower.**

### The workload is genuinely small

The hybrid is **21,089 parameters**: an LSTM with 64 hidden units over a 20-step window, plus two Conv1D layers. At that size a GPU buys almost nothing, because each step is dominated by kernel-launch and Python overhead rather than arithmetic. The RTX 2080 Ti was never the bottleneck.

Rough wall-clock estimate for the whole Phase 2 bundle on a single machine:

| Job | Scale | Estimate |
|---|---|---|
| Phase 1 pretraining, 5 seeds | approx. 38,000 windows, 100 epochs, patience 12 | 15 to 25 min |
| Full SIT evaluation | 20 cells x 3 splits x 5 seeds x (hybrid + scratch) | 30 to 45 min |
| Cross-dataset matrix | 10 scenarios, each pretrain plus eval | 2 to 3 h |
| C8 fixed-cycle budgets | 20 cells x 4 budgets x 5 seeds x 2 models | 40 to 60 min |
| C7, C12, C13, C14 | inference sweeps and fine-tune-only controls | 30 to 45 min |
| RUL foundation stage | 155-cell decoder pretrain plus 20-cell LOOCV | 1 to 2 h |

**Total: roughly 4 to 8 hours of wall clock.** This is an overnight job, not a multi-day one.

### Why the laptop cannot do it anyway

Laptop: Intel i7-11800H (8 cores, 16 threads), 32 GB RAM, RTX 3050 Ti Laptop (4 GB).

Three hard blockers, in order of severity:

1. **The datasets are not here.** `other_data/` does not exist on this machine at all, so MIT, LISHEN, HUST, SNL and CALCE are all missing. `battery-soh-prediction/refined_dataset/` is empty, so NASA raw is missing too (it is gitignored as `refined_dataset/full/`, being large). `hybrid_solution.py` resolves `MIT_DATA = BASE.parent / 'other_data' / 'MIT_Toyota_LFP' / 'processed'`, which simply does not exist locally. **C10 cannot run here by definition**, since it is a change to how the MIT corpus is partitioned. Only the SIT data (`publishable/Data`, 14,820 files) is present.
2. **There is no TensorFlow.** `import tensorflow` fails in `.venv`. And installing it does not give GPU: TensorFlow dropped native Windows GPU support after 2.10, so on this laptop it would be CPU-only unless you set up WSL2 with CUDA. The pipeline needs `tf.config.experimental.enable_op_determinism()`, which is 2.8 or newer.
3. **Determinism and consistency.** The paper claims every number "reproduces bit-for-bit on the same hardware and software stack", and ships a checkpoint on Zenodo. Numbers produced on a Windows CPU stack will not match numbers produced on the Linux CUDA stack. Since some results are not being re-run (DE-LSTM baselines, LOOCV, screening), a laptop re-run would leave the paper as a mix of two stacks and would invalidate the reproducibility claim outright.

### What can run on the laptop, right now, with no setup

The scientific Python stack is already present (matplotlib 3.10.8, pandas 3.0.1, numpy 2.4.2, scipy 1.17.1), and the figure scripts in `battery-soh-prediction/figures_src/` import only matplotlib, numpy, pandas and scipy. No TensorFlow.

Also already present locally and tracked in git: `results/hybrid/selected_checkpoint.weights.h5` (0.29 MB), the five pretrain candidates, and `results/review/lfp_corpus.pkl` (0.91 MB), plus all result CSVs (21 MB).

So **do all of this locally**:

- Regenerate every figure (C22, and the Figure 8 versus Table 4 contradiction).
- C5 persistence and drift baselines: pure NumPy over the SIT SOH arrays, no TensorFlow needed.
- C18 censoring statistics and C19 risk-coverage plot: pandas re-analysis of `t52_foundation_rul.csv` and `t53_benchmark_table.csv`.
- All wording, terminology, abstract, literature, dataset table, RUL documentation: C1, C2, C3, C4, C9, C11, C15, C16, C17, C20, C21, C25.

That is Phase 0 plus most of Phase 1, roughly 15 of 25 comments, with zero GPU time.

### Recommended split

1. **Local, now:** Phase 0 and the analysis-only parts of Phase 1. Send that to Dr Anurag as a visible first response.
2. **Commit and push:** `battery-soh-prediction` (remote `https://github.com/karthicgrepo/battery-soh-prediction.git`) currently has four uncommitted modified files (`experiments/make_methodology_fig.py`, `experiments/t53_rul_benchmark_figure.py`, and two `t53` figure outputs). `experiments/`, `figures_src/` and `results/` are all tracked, so the push is small. `latex/sit_soh_paper` is a separate repo and needs its own push.
3. **GPU box, one batch:** C10 plus C12, C13, C7, C8, C14 in a single pipeline execution at `/home/c1035830/g/Batery Degradation data NYP/battery-soh-prediction/`, which already holds `other_data/` and `refined_dataset/full/`. Write the context file before triggering, as last time.
4. **Pull results back**, then re-plot figures locally and reconcile every number against the manuscript.

One caution for step 3: the GPU box was last inventoried on 2026-06-11 and at that point held the *older* results (ablation 47.4%, NASA coverage 34%, brittleness 2.3x). The current paper's numbers come from the later deterministic runs dated 9 July onward. Confirm the box is synced to the current state before launching, or you will silently re-run from a stale baseline.

---

## 8. Decisions I need from you

1. **Venue.** Journal of Energy Storage, Energy and AI, or Journal of Power Sources. This changes the template and the framing. Energy and AI is the best fit for the transfer plus uncertainty plus deployment angle and is where the attached reference was published, which also makes citing it natural.
2. **Conformal fix.** Disjoint selection and calibration cell sets, or leave-one-cell-out cross-conformal over the 17 test cells. I recommend the second.
3. **TCN and Transformer.** Give them a matched hyperparameter search, or remove them and state why.
4. **Scope control.** Comments 7, 8, 12, 13 and 14 are five new experiments. Confirm you want all five before I start, or tell me which to defer to a limitations paragraph.
5. **Whether to send an interim reply now** acknowledging the review and flagging C23 and C24 as already implemented, so he does not repeat them in the next round.
