# Draft reply to Dr Anurag Sharma

Paste the body below into the reply. Numbering follows his 1 to 25. Attach the
rebuilt `main.pdf`; attach `review_response.md` too if you want him to have the
long-form version with section and table references.

His thread subject is the stale one, "RE: Latest Battery dataset paper Scientific
Data", though the content is all about the SOH paper. Suggested correction below.

---

**Subject:** SOH prediction paper: revision and point-by-point response (was: RE: Latest Battery dataset paper Scientific Data)

Dear Anurag,

Thank you for the review. It found a real error, and the paper is better for it.
All twenty-five points are addressed below; twenty-four are implemented, and I have
said plainly which one is not.

Two things first. Your point 10 was correct: the pretraining split was leaking, so I
fixed it and re-ran end to end. The headline numbers moved and I report them both
ways rather than quietly restating: fingerprint transfer 38.2% to 23.3%, and the
full hybrid 53.5% to 51.1%. Second, points 14 and 15 turned up a negative result, and it is
now written up as one. The paper is also prepared for Energy and AI, where your
attached paper appeared.

1. Done. Title, abstract, gap and discussion now lead with saturation and
   cell-to-cell variation.
2. Done. "Cell-to-cell variability" throughout; manufacturing named only as one
   likely contributor.
3. Done. Cell identity explains more error variance than model choice (58/31/25%
   against 15 to 17%), rather than "dominates".
4. Done. Spread is 7.6x at matched cycle count and 13.7x at matched throughput,
   against 13.2x at final SOH. It survives the fair comparison.
5. Done, and it favours your argument. Persistence reaches 0.0128 at the 50% split,
   beating our hybrid at 0.0156. Drift overshoots.
6. Done. Renamed "fingerprint-conditioned transfer, no target-cell fine tuning", with
   the target information used stated explicitly.
7. Done, as a new study. Supplying true SOH every k cycles, RMSE rises from 0.015
   (k=1) to 0.055, 0.114 and 0.202 (k=5, 10, 25). One-step accuracy holds only under
   frequent SOH estimation.
8. Done. On the first 30/50/100/200 cycles the hybrid beats scratch by 37/34/13/37%.
9. Done. Reworded; 3.6% is the residual of that cubic fit, not "irreducible".
10. Done and re-run, as above. Checkpoint and Zenodo artefact reissued.
11. Done. Prediction interval throughout, including figures.
12. Done. The three selection cells are excluded from calibration.
13. Done. Grouped leave-one-cell-out conformal, one score per cell. Honestly: raw
    MC-Dropout under-covers at 82% against nominal 95%, and grouped conformal is
    conservative because only ~16 cells set the quantile. The old 93.9% is removed,
    as it reused selection data.
14. You were right. Correct, zero, mean, shuffled and random fingerprints all give
    the same RMSE (0.0149 to 0.0156); the encoder maps every cell to a near-constant
    embedding. It is an in-distribution population prior, not cell identification,
    and the paper now says so.
15. Removed, per 14.
16. Done. "Chemistry-alignment effect", softened to note the datasets also differ in
    format, manufacturer and protocol.
17. Done. New subsection engaging with Zhang et al., whose finding that cross-dataset
    transfer degrades under distribution shift corroborates ours from the opposite
    direction. I have adopted their zero-shot versus prefix-adapted vocabulary.
18. Done. 12 of 20 cells reach EOL, 8 are right-censored at SOH 0.85 to 0.92. Errors
    are computed on the 12; censored cells are still forecast, 190 to 324 cycles
    beyond the anchor.
19. Done, as a new figure. It does not flatter us on accuracy alone: at the 50%
    anchor DE-LSTM reaches 12 cycles and the autoregressive hybrid 11, against our
    49. They get there by declining the cells that matter most. Our claim is full
    coverage, not lowest error.
20. Done. Full specification: 14 horizons (10 to 600 cycles), equal-weight MSE, Adam
    1e-3, interpolation rule, no-crossing handling, no monotonicity constraint.
21. Done, and it closed a real hole, since NASA was used but never described. The
    table now covers all six datasets with sources.
22. Done. Scratch LSTM named consistently, NASA labelled LCO, figures regenerated so
    they no longer disagree with the tables.
23. Agreed. The improvement, now 51%, is scoped as over the scratch baseline
    everywhere it appears.
24. Agreed, and stated directly in the abstract, results and conclusion.
25. Not done. Adding the analyses you asked for in 7, 8, 14, 18 and 19 made the paper
    longer, not shorter. I will do the cut last, on the final structure, and send you
    the shortened version before submission. Say the word if you would rather review
    a shortened draft first and I will reverse the order.

Two things I would value your view on: whether the honest conformal coverage in 13
stands or the calibration set is simply too small, and whether TCN and Transformer
should get a matched hyperparameter search or be removed with a stated reason.

Thank you again.

Best regards,
Karthickumar
