# Response to Reviewer (Dr Anurag Sharma), 21 September 2026

We thank the reviewer for the detailed and constructive comments. They materially
improved the paper. Below we respond point by point. Section and table references
are to the revised manuscript (now prepared for *Energy and AI*, elsarticle format).
The most consequential change is a correction to the pretraining split (C10), which
we re-ran end to end; we report the before/after numbers honestly wherever they moved.

Twenty-four of the twenty-five comments are now implemented in the manuscript. The
remaining one, C25 (length), is noted honestly at the end.

---

**C1: Lead with the saturation finding; restructure abstract/contributions/discussion/conclusion.**
Done. The abstract, title, research-gap and discussion now lead with benchmark
saturation and cell-to-cell variation, and frame the hybrid as a deployment
capability set rather than a one-step winner. New title: "Beyond One-Step Accuracy:
Deployment-Oriented Health Prediction for Large-Format LiFePO4 Cells." The abstract
follows the structure you suggested.

**C2: "manufacturing variability" overclaims causation.**
Done. We use "cell-to-cell variability" as the phenomenon label throughout the main
results and headings, and mention manufacturing only as a likely contributor
alongside cycler-channel, contact-resistance, temperature and batch effects
(Section 4.1, 5.3).

**C3: Abstract dominance claim too strong.**
Done. We now state that cell identity explains substantially more prediction-error
variance than model choice (58/31/25% vs 15-17%), not that manufacturing "dominates."

**C4: 13.2x final-SOH spread is unfair (different cycle counts).**
Done. New matched-exposure analysis (Section 5.3): the G1 spread is 7.6x at a common
cycle count (684 cycles) and 13.7x at common cumulative throughput (26.8 kAh), versus
13.2x at final SOH. The dispersion is an order of magnitude under every fair
comparison, so it is genuine, not an artefact of unequal durations.

**C5: Add persistence (and drift) baselines.**
Done, and it strengthens the argument. Persistence (yhat=y_{t-1}) reaches RMSE
0.0128 at the 50% split, which beats the proposed hybrid (0.0156) and nearly matches
the best deep model (DE-LSTM 0.0118) on the one-step metric. Drift (0.021-0.023)
overshoots. Both are added to Table 3 and the saturation discussion (Section 5.2).

**C6: "zero-shot" is misleading.**
Done. We renamed it "fingerprint-conditioned transfer (no target-cell fine-tuning)"
throughout, and state exactly what target information is used (the first 30 cycles,
from which the fingerprint is computed; no target SOH label is fitted).

**C7: What SOH is observable in a BMS?**
Done. New intermittent-observation study (Section 7.6): the true SOH is supplied only
every k cycles (k = 1, 5, 10, 25) and the model self-feeds otherwise. Mean one-step
RMSE over the 17 test cells rises from 0.015 (k=1) to 0.055, 0.114 and 0.202 (k=5, 10,
25). We state honestly that the one-step accuracy holds only under frequent SOH
estimation, and that the trajectory-level RUL forecaster (Section 9), which predicts
from a single early anchor, is what serves the sparse-observation regime.

**C8: Add fixed-cycle budgets (30/50/100/200), not just retrospective %.**
Done (Section 7.5, new table). Hybrid vs scratch trained on the first 30/50/100/200
cycles: the hybrid improves at every budget by 37/34/13/37% mean per-cell RMSE on the
17 test cells, including 37% from only the first 30 cycles.

**C9: "irreducible within-cell variance" is incorrect.**
Done. Reworded: 3.6% is the residual not captured by that particular cubic fit; we
no longer call it irreducible (Section 5.3).

**C10: Make the pretraining split cell-disjoint.**
Done, and re-run end to end. Both the main pretraining and the cross-dataset
pretraining now partition whole cells into train/validation folds before windowing
(previously a flat tail slice let one cell straddle the boundary). The checkpoint,
all downstream tables/figures, and the Zenodo artefact are reissued. Numbers moved as
expected and we report before/after: fingerprint-transfer improvement 38.2% -> 23.3%,
full hybrid 53.5% -> 51.1%, checkpoint-selection robustness 95.5% -> 100%. The drop in
the no-fine-tuning number is informative (see C14).

**C11: Use "prediction interval," not "confidence interval."**
Done throughout (text, tables, figures).

**C12: Do not use the same cells for checkpoint selection and calibration.**
Done. The three selection cells are now excluded from conformal calibration
(Section 7.3).

**C13: Hundreds of cycles from one cell are not independent calibration samples.**
Done. We switched to grouped, leave-one-cell-out conformal, taking one nonconformity
score per cell so correlated within-cell residuals are not counted as independent.
Honest coverage: raw MC-Dropout under-covers (82% at nominal 95%); grouped conformal
attains at least nominal coverage but conservatively (91% at nominal 80%, 100% at
95%), because only ~16 cells calibrate the quantile. The previous 93.9% (split
conformal on the selection cells) is removed as it reused the selection data.

**C14: Reassess what the fingerprint contributes (shuffled ~ correct).**
Done, and the result is decisive (Section 7.1, new control table). Across the correct,
zero, population-mean, shuffled and random fingerprints the fine-tuned model gives
indistinguishable RMSE (0.0149 to 0.0156) and coverage (80 to 82%); the correct
fingerprint is if anything marginally worse than the controls. The trained encoder
maps every cell to a near-constant embedding (mean pairwise cosine similarity ~1.0
across the MIT cells). We now state plainly that the mechanism is an in-distribution
population prior, not cell-specific identification, and we removed all claims to the
contrary (this also resolves C15).

**C15: Do not say the fingerprint "identifies the type of degrader."**
Done. Removed; the fingerprint's role is reframed per C14. The methodology table now
asks "what is this cell's early-life degradation signature?" rather than claiming
degrader-type identification.

**C16: "chemistry alignment principle" is too strong.**
Done. Renamed "chemistry-alignment effect" throughout and softened to note the
datasets also differ in format, manufacturer, laboratory and protocol.

**C17: Update the literature; cite recent cross-dataset transfer work incl. attached paper.**
Done. A new subsection, "Cross-Dataset and Foundation-Model Transfer", engages with
the paper you attached (Zhang et al., Energy and AI 25 (2026) 100771) rather than
merely listing it: we set out their zero-shot versus prefix-adapted protocol, note
that their finding that cross-dataset transfer degrades under distribution shift
corroborates our domain-alignment effect from a different direction (they vary the
dataset at fixed chemistry, we vary the chemistry), and adopt their vocabulary, so
our encoder is now described as prefix-adapted rather than zero-shot. The novelty
claim is rewritten to be differential, large-format 50 Ah prismatic cells with a
per-cell thermal record and an explicit test of whether calibrated uncertainty
survives transfer, rather than asserting the question has not been investigated. We
also added the missing dataset source references (HUST: Ma et al. 2022; SNL: Preger
et al. 2020).

**C18: RUL: report censored cells and how methods behave on them.**
Done. 12 of 20 cells reach the 0.80 EOL threshold; 8 are right-censored (001-2,5,6,7;
002-1,2,3; 002-5), still at SOH 0.85-0.92. Error statistics are computed on the 12
EOL cells; the foundation model still forecasts every censored cell, placing the
crossing 190-324 cycles beyond the anchor and never inside the observed span
(Section 9).

**C19: Show RUL error together with coverage (risk-vs-coverage).**
Done. New figure plotting median RUL error against the fraction of cells forecast, at
both anchors. It makes the trade-off explicit: accuracy alone would rank the recursive
lineage first at the 50% anchor (DE-LSTM 12 cycles, autoregressive hybrid 11, against
our 49), coverage alone would rank them last, and they reach that low error by
declining exactly the cells a manufacturer needs a forecast for. The accompanying text
states plainly that on the cells they do forecast the recursive methods are the more
accurate, and that our claim is full coverage, not lowest error.

**C20: Under-specified RUL decoder.**
Done. Full specification added (Section 9.1): 14 horizons (10-600 cycles), equal-weight
MSE, Adam 1e-3, targets padded past record, linear interpolation for the 0.80 crossing,
no-crossing handling (reports no crossing, no slope extrapolation), no monotonicity
constraint, MC-Dropout 80% intervals.

**C21: Consolidated dataset table.**
Done. New Table 1 lists SIT, MIT, NASA, LISHEN, HUST and SNL with chemistry, format,
capacity, cell count, conditions and train/calibration/test role.

**C22: Figure and terminology inconsistencies.**
Done. (a) The ablation baseline is named "Scratch LSTM" consistently. (b) NASA is
labelled LCO everywhere (its assigned chemistry), correcting the earlier NMC label.
(c) PI is used for predictive uncertainty throughout. (d) "fine-tuning,"
"pretraining," "fingerprint-conditioned transfer" standardised.

**C23: Do not present 53.5% as better than all methods (push-back with agreement).**
Agreed, and this was already scoped in the manuscript: the improvement (now 51%) is
stated as "over the scratch baseline" in the abstract, ablation table, and conclusion.
We have made it explicit in each place and removed any leaderboard reading.

**C24: State explicitly the hybrid is not meant to win the one-step benchmark (push-back with agreement).**
Agreed, and now stated directly in the abstract, results and conclusion: the hybrid is
not intended to win the saturated one-step benchmark; its value is transfer to new
cells, uncertainty, early screening and RUL.

**C25: Reduce length by 15-20%. [in progress]**
In progress as the final editing pass once the C7/C8/C14 results are written in, so the
cut is made on the final structure.
