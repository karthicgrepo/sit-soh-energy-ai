# Draft reply to Dr Anurag Sharma

Paste the body below into the reply. Two notes before you send:

- His thread subject is the stale one, "RE: Latest Battery dataset paper Scientific
  Data", but the content is entirely about the SOH prediction paper. Suggested
  subject correction is in the draft.
- Attach the rebuilt `main.pdf`, and `review_response.md` if you want him to have the
  long-form point-by-point alongside the summary.

---

**Subject:** SOH prediction paper: revision and point-by-point response (was: RE: Latest Battery dataset paper Scientific Data)

Dear Anurag,

Thank you for the review. It was detailed and specific, and it found a real error, so
the paper is materially better for it. I have worked through all twenty-five comments
and attach the revised manuscript. Below is a point-by-point summary; a fuller
version with section and table references is in the attached response document.

Three things are worth flagging before the list.

**The pretraining split was leaking, and fixing it moved the headline numbers.** Your
comment 10 was correct. The pretraining data were partitioned by a flat tail slice
after windowing, which let cycles from one cell fall on both sides of the
train/validation boundary. Both the main and the cross-dataset pretraining now
partition whole cells before windowing, and I re-ran the pipeline end to end. The
numbers moved, and I am reporting them before and after rather than quietly
restating: fingerprint-conditioned transfer 38.2% to 23.3%, full hybrid 53.5% to
51.1%, checkpoint-selection robustness 95.5% to 100%. The released checkpoint and the
Zenodo artefact are reissued.

**Your comments 14 and 15 led to a negative result that I have now reported as one.**
Testing the fingerprint against controls, as you asked, showed the encoder is not
doing what we claimed. That is written up plainly rather than smoothed over.

**The paper is now prepared for Energy and AI** rather than the previous venue, using
Elsevier's CAS template. The transfer, uncertainty and deployment framing fits that
journal, and it is where the paper you attached was published, which made engaging
with it natural.

---

**C1. Lead with the saturation finding.** Done. Title, abstract, research gap and
discussion now open with benchmark saturation and cell-to-cell variation, and frame
the model as a deployment capability set. New title: "Beyond One-Step Accuracy:
Cell-to-Cell Variability, Transfer, and Calibrated Uncertainty in
Deployment-Oriented Health Prediction for Large-Format LiFePO4 Cells".

**C2. "Manufacturing variability" overclaims causation.** Agreed. The phenomenon is
now labelled cell-to-cell variability throughout, with manufacturing named only as one
likely contributor alongside cycler channel, contact resistance, temperature and batch
effects.

**C3. Abstract dominance claim too strong.** Agreed. We now say cell identity explains
substantially more prediction-error variance than model choice (58/31/25% against 15
to 17%), rather than that manufacturing dominates.

**C4. The 13.2x spread is unfair at unequal cycle counts.** A fair point, and I have
added the matched comparison. The G1 spread is 7.6x at a common cycle count (684
cycles) and 13.7x at common cumulative throughput (26.8 kAh), against 13.2x at final
SOH. The dispersion survives every fair comparison, so it is genuine rather than an
artefact of unequal durations.

**C5. Add persistence and drift baselines.** Done, and it strengthened your argument
rather than ours. Persistence reaches RMSE 0.0128 at the 50% split, which beats our
hybrid (0.0156) and nearly matches the best deep model (DE-LSTM, 0.0118). Drift
overshoots at 0.021 to 0.023. Both are now in the baseline table and the saturation
discussion.

**C6. "Zero-shot" is misleading.** Agreed. Renamed to fingerprint-conditioned transfer
with no target-cell fine tuning, and we now state exactly what target information is
used: the first 30 cycles, from which the fingerprint is computed, with no target SOH
label fitted.

**C7. What SOH is actually observable in a BMS?** Done, as a new study. Supplying true
SOH only every k cycles and letting the model self-feed otherwise, mean one-step RMSE
over the 17 test cells rises from 0.015 at k=1 to 0.055, 0.114 and 0.202 at k=5, 10 and
25. The paper now states that one-step accuracy holds only under frequent SOH
estimation, and that the trajectory-level RUL forecaster is what serves the sparse
observation regime.

**C8. Use fixed cycle budgets, not retrospective percentages.** Done. Trained on the
first 30/50/100/200 cycles, the hybrid improves on the scratch baseline by 37/34/13/37%
mean per-cell RMSE, including 37% from only the first 30 cycles.

**C9. "Irreducible within-cell variance" is incorrect.** Agreed and reworded. The 3.6%
is the residual not captured by that particular cubic fit, and is no longer called
irreducible.

**C10. Make the pretraining split cell-disjoint.** Done and re-run, as above.

**C11. Prediction interval, not confidence interval.** Corrected throughout, in text,
tables and figures.

**C12. Do not reuse selection cells for calibration.** Done. The three
checkpoint-selection cells are excluded from conformal calibration.

**C13. Cycles from one cell are not independent calibration samples.** Agreed, and this
one changed our reported coverage. We moved to grouped leave-one-cell-out conformal,
taking one nonconformity score per cell. Honestly: raw MC-Dropout under-covers at 82%
against a nominal 95%, and grouped conformal reaches at least nominal coverage but
conservatively, because only about 16 cells calibrate the quantile. The previous 93.9%
figure is removed, since it reused the selection data.

**C14. Reassess what the fingerprint contributes.** You were right, and the result is
decisive against our earlier claim. Across correct, zero, population-mean, shuffled and
random fingerprints the fine-tuned model gives indistinguishable RMSE (0.0149 to
0.0156) and coverage (80 to 82%), with the correct fingerprint marginally worse than
the controls. The trained encoder maps every cell to a near-constant embedding, mean
pairwise cosine similarity close to 1.0. The paper now states that the mechanism is an
in-distribution population prior, not cell-specific identification, and all claims to
the contrary are removed.

**C15. Do not say the fingerprint identifies the type of degrader.** Removed, per C14.

**C16. "Chemistry alignment principle" is too strong.** Agreed. Renamed to
chemistry-alignment effect, and softened to note that the datasets also differ in
format, manufacturer, laboratory and protocol.

**C17. Update the literature, including the attached paper.** Done. There is a new
subsection on cross-dataset and foundation-model transfer that engages with Zhang et
al. rather than merely citing it. Their finding that cross-dataset transfer degrades
under distribution shift corroborates our domain-alignment effect from the opposite
direction, since they vary the dataset at fixed chemistry while we vary the chemistry.
I have adopted their zero-shot versus prefix-adapted vocabulary, which is a cleaner
way to draw the distinction you raised in C6. The novelty claim is now differential,
large-format prismatic cells with a per-cell thermal record and a test of whether
calibrated uncertainty survives transfer, instead of claiming the question has not been
investigated.

**C18. Report censored cells in the RUL analysis.** Done. Twelve of twenty cells reach
the 0.80 threshold and eight are right-censored, still at SOH 0.85 to 0.92. Error
statistics are computed on the twelve; the foundation model still forecasts every
censored cell, placing the crossing 190 to 324 cycles beyond the anchor and never
inside the observed span.

**C19. Show RUL error together with coverage.** Done, as a new risk-versus-coverage
figure. It makes the trade-off explicit, and not in our favour on accuracy alone: at
the 50% anchor DE-LSTM reaches 12 cycles and the autoregressive hybrid 11, against our
49. They reach that by declining the cells a manufacturer most needs forecast. The text
says so directly, and our claim is full coverage rather than lowest error.

**C20. The RUL decoder was under-specified.** Agreed, that section was the weakest part
of the paper. It now specifies the 14 horizons (10 to 600 cycles), equal-weight MSE,
Adam at 1e-3, target padding past the record, linear interpolation for the 0.80
crossing, explicit no-crossing handling with no slope extrapolation, the absence of a
monotonicity constraint, and MC-Dropout 80% intervals.

**C21. Consolidated dataset table.** Done, and it closed a real hole: NASA was used in
results but never formally described. The new table lists SIT, MIT, NASA, LISHEN, HUST
and SNL with chemistry, format, capacity, cell count, conditions, role and source
reference.

**C22. Figure and terminology inconsistencies.** Done. The ablation baseline is named
Scratch LSTM consistently, NASA is labelled LCO everywhere, prediction interval is used
throughout, and the figures are regenerated so they no longer disagree with the tables.

**C23. Do not present 53.5% as better than all methods.** Agreed. The improvement, now
51%, is scoped as over the scratch baseline in the abstract, the ablation table and the
conclusion, and any leaderboard reading is removed.

**C24. State that the hybrid is not meant to win the one-step benchmark.** Agreed, and
now stated directly in the abstract, results and conclusion. Its value is transfer to
new cells, calibrated uncertainty, early screening and RUL.

**C25. Reduce length by 15 to 20%.** Not yet done, and I do not want to claim
otherwise. Adding the new analyses you asked for in C7, C8, C14, C18 and C19 has made
the manuscript longer, not shorter. The editing pass is the last thing I will do, on
the final structure, and I will send you the shortened version before submission. If
you would rather I cut first and you review the shorter draft, say so and I will
reverse the order.

---

I would value your view on two things in particular. First, whether the honest
conformal coverage in C13 is acceptable as it stands or whether the calibration set is
simply too small to make the claim. Second, whether the TCN and Transformer baselines
should be given a matched hyperparameter search or removed with a stated reason, since
at present they are fixed-configuration and I do not want them read as a tuned upper
bound.

Thank you again for the care you put into this.

Best regards,
Karthickumar
