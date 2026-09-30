# Draft reply to Dr Anurag Sharma

Every point now restates his comment before answering it, so the reply reads on its
own without his email open beside it. His numbering, 1 to 25.

Attach the rebuilt `main.pdf`. Attach `review_response.md` too if you want him to
have the long-form version with section and table references.

---

**Subject:** SOH paper: revised manuscript and point-by-point response

Dear Anurag,

Thank you for the review. These were not boilerplate comments, and point 10 in
particular found a genuine error, so the paper is materially better for them.

On your closing instruction, that this needed analysis rather than only textual
changes: agreed, and the model was re-run rather than the text rewritten. The four
areas you singled out were all redone from the data. Under all of them sits point 10,
which invalidated the trained checkpoint and forced a full re-run, so several headline
numbers have moved. I give the before and after in each case rather than quietly
restating them.

Twenty-four of the twenty-five are implemented. Point 25, the length cut, is not, and
I say why below.

**1. Restructure the story around saturation, and a suggested abstract.** Done. I have
used your suggested abstract close to verbatim, with two numbers updated after the
point-10 re-run (38% becomes 23%, 53.5% becomes 51%). The title, research gap,
discussion and conclusion now open on saturation and cell-to-cell variation rather
than on one-step accuracy.

**2. "Manufacturing variability" cannot be proven; use "cell-to-cell variability".**
Done throughout the results and headings. Manufacturing is now named as one likely
contributor alongside cycler channel, contact resistance, temperature variation and
batch differences, as you listed.

**3. The abstract's dominance claim is too strong.** Rewritten. It now says cell
identity explains substantially more prediction-error variance than model choice
within this dataset (58/31/25% against 15 to 17%), and no longer claims manufacturing
dominates temperature or model choice.

**4. The 13.2x spread at final SOH is unfair; reanalyse at matched cycle number or
throughput.** Done, and the argument survives it. At a common cycle count (684) the
G1 spread is 7.6x, and at common cumulative throughput (26.8 kAh) it is 13.7x,
against 13.2x at final SOH. So the dispersion is real rather than an artefact of
cells reaching final SOH at different cycle numbers.

**5. Add persistence, and a drift model.** Done, and it strengthens your point rather
than ours. Persistence reaches RMSE 0.0128 at the 50% split, which beats our hybrid
at 0.0156 and is close to the best deep model at 0.0118. Drift overshoots at 0.021 to
0.023. Both are now in the baseline table and the saturation discussion.

**6. "Zero-shot" is confusing; define what target-cell information is available.**
Renamed to fingerprint-conditioned transfer with no target-cell fine tuning, your
second suggestion. The paper now states exactly what is used: the first 30 cycles of
the target cell, from which the fingerprint is computed, with no target SOH label
fitted, and rolling observations thereafter.

**7. In a real BMS the true SOH is not available every cycle.** This needed a new
experiment, and it is the one that most changed how the paper reads. Supplying true
SOH only every k cycles and letting the model self-feed otherwise, mean one-step RMSE
over the 17 test cells rises from 0.015 at k=1 to 0.055, 0.114 and 0.202 at k=5, 10
and 25. The paper now states plainly that one-step accuracy holds only under frequent
SOH estimation, and that the RUL forecaster, which predicts from a single early
anchor, is what serves the sparse-observation case.

**8. The 30/50/70% splits are retrospective; use fixed cycle budgets.** Done as new
training runs: 20 cells by four budgets (first 30, 50, 100, 200 cycles) by five seeds,
for both the hybrid and the scratch baseline. The hybrid improves by 37/34/13/37%,
including 37% from only the first 30 cycles.

**9. "Irreducible within-cell variance" of 3.6% is not correct.** Reworded. It is
described as the fraction of trajectory variation not captured by that particular
cubic fit, and the word irreducible is gone.

**10. The pretraining split lets one MIT cell contribute windows to both training and
validation; make it cell-disjoint.** You were right. The split was taking a flat tail
slice after windowing, so cycles from a single cell fell on both sides. It now
partitions whole cells before windowing, in both the main and the cross-dataset
pretraining. That invalidated the trained model, so I re-ran the entire pipeline and
regenerated the released checkpoint and code archive. The numbers moved, as expected
for a leak of this kind: fingerprint-conditioned transfer 38.2% to 23.3%, full hybrid
53.5% to 51.1%, checkpoint-selection robustness 95.5% to 100%. The RUL results were
unaffected.

**11. Use "prediction interval", and MC-Dropout is not calibrated by fiat.** Corrected
throughout, in text, tables and figures. The paper no longer treats a nominal 95%
MC-Dropout range as a calibrated interval, which point 13 then quantifies.

**12. Do not use the same cells for checkpoint selection and for calibration.** Done.
The three checkpoint-selection cells are now excluded from the conformal calibration
set entirely.

**13. Hundreds of cycles from one cell are not independent; use grouped calibration.**
Done, and this one changed a headline number for the worse, so I want to be direct
about it. Calibration is now grouped leave-one-cell-out, taking one nonconformity
score per cell. Raw MC-Dropout under-covers, at 82% against a nominal 95%. Grouped
conformal does reach nominal coverage but conservatively, because only about 16 cells
set the quantile. The previously reported 93.9% is removed, because it was split
conformal computed on the selection cells, which is exactly the reuse you flagged in
point 12.

**14. The shuffled control performs close to the correct fingerprint; add zero, mean,
correct, shuffled and random controls.** Done, all five, and the result is decisive
against our earlier interpretation. The fine-tuned model gives indistinguishable RMSE
across every control (0.0149 to 0.0156) and indistinguishable coverage (80 to 82%),
with the correct fingerprint marginally worse than the controls. Your reading was the
right one: the encoder supplies an in-distribution population embedding, not
cell-specific identification, and the paper now says so.

**15. Avoid "identifies the type of degrader"; visualise the embeddings and check
whether they correlate with final SOH, degradation rate, knee or cycle life.** The
claim is removed. On the correlation question, the answer falls straight out of the
control study in point 14: the trained encoder maps every cell to a near-constant
embedding, mean pairwise cosine similarity essentially 1.0, so the embeddings cannot
correlate with final SOH or degradation rate, because they barely differ between
cells. I have reported that number rather than adding an embedding scatter plot; say
the word if you would still like the figure and I will add it.

**16. "Chemistry alignment principle" is too strong; the datasets differ in more than
chemistry.** Agreed and renamed to chemistry-alignment effect, with an explicit note
that MIT, SIT and NASA also differ in format, manufacturer, laboratory, protocol and
capacity, so chemistry is not isolated.

**17. Update the literature, and see the attached paper.** Done. There is a new
subsection on cross-dataset and foundation-model transfer built around Zhang et al.,
the paper you attached. Their finding that cross-dataset transfer degrades under
distribution shift corroborates our alignment effect from the opposite direction,
since they vary the dataset at fixed chemistry while we vary the chemistry. I have
also adopted their zero-shot versus prefix-adapted vocabulary, which is a cleaner way
to draw the distinction you raised in point 6. The novelty claim is now differential,
large-format prismatic cells with a per-cell thermal record and a test of whether
calibrated uncertainty survives transfer, rather than any claim that this has not been
investigated.

**18. Cells that never reach 80% are right-censored, not simply unavailable.** Done.
12 of the 20 cells reach the threshold and 8 are right-censored, still at SOH 0.85 to
0.92. Error statistics are computed on the 12, the counts are now stated explicitly,
and the censored cells are still forecast, with the predicted crossing placed 190 to
324 cycles beyond the anchor and never inside the observed record.

**19. Show RUL error together with coverage; a risk-versus-coverage plot.** Done, as a
new figure, and it does not flatter us on accuracy alone. At the 50% anchor DE-LSTM
reaches 12 cycles and the autoregressive hybrid 11, against our 49. They reach that by
declining to forecast the cells a manufacturer most needs. Our claim in the paper is
now full coverage, not lowest error, which is the point you were making.

**20. Specify the direct multi-horizon RUL model properly.** Done, and I agree that
section was the weakest part of the paper. It now gives the 14 horizons (10 to 600
cycles), the equal-weight MSE loss and the absence of horizon weighting, the linear
interpolation rule for the 0.80 crossing, the explicit treatment when the trajectory
never crosses (it reports no crossing, with no slope extrapolation), and the fact that
monotonic degradation is not enforced.

**21. One consolidated dataset table.** Done, and it closed a real hole, because NASA
was used in the results but never formally described anywhere. The new table covers
SIT, MIT, NASA, LISHEN, HUST and SNL with chemistry, format, nominal capacity, cell
count, conditions, role in the study and source reference.

**22. Terminology and figure inconsistencies.** All four done. The ablation baseline is
named Scratch LSTM consistently, NASA is labelled LCO everywhere after checking the
assigned chemistry, PI is used throughout for predictive uncertainty, and the
fine-tuning and pretraining terms are standardised. I also regenerated the figures,
which had drifted out of step with the tables.

**23. Do not present 53.5% as better than all competing methods.** Agreed. The
improvement, now 51%, is stated as being over the scratch baseline in the abstract,
the ablation table and the conclusion, and any leaderboard reading has been removed.

**24. State explicitly that the hybrid is not meant to win the one-step benchmark.**
Agreed, and now stated directly in the abstract, results and conclusion. Its value is
transfer to a new cell, calibrated uncertainty, early screening and RUL forecasting.

**25. Reduce the paper by 15 to 20%.** Not done, and I will not pretend otherwise. The
new analyses in points 7, 8, 14, 18 and 19 have made the manuscript longer rather than
shorter. I would rather make the cut last, on the final structure, and send you the
shortened version before submission. If you would prefer to review a shortened draft
first, tell me and I will reverse the order.

All eight items on your must-revise list are covered: variability and causality in 2
and 4, the deployment benchmarks in 5 and 8, the zero-shot definition in 6 and 7, the
conformal methodology in 12 and 13, the fingerprint controls in 14, the alignment
claim in 16, censoring and coverage in 18 and 19, and the naming and PI
inconsistencies in 22.

On the journal, I have taken Energy and AI. You noted it would be a good option if we
sharpened the transfer learning, uncertainty and deployment contributions, and that is
what this revision has done, so it now looks like the better fit. The attached
manuscript is rebuilt in Elsevier's template for that journal.

Two things I would value your view on. First, whether the honest conformal coverage in
point 13 is acceptable, or whether 16 calibration cells are simply too few to support
an interval claim at all. Second, whether TCN and Transformer should be given a matched
hyperparameter search or removed with a stated reason, since at present they run in a
fixed configuration and I do not want them read as a tuned upper bound.

Thank you again for the care you put into this.

Best regards,
Karthic
