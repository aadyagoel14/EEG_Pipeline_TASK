# EEG_Pipeline_TASK
Build an ML pipeline to predict BIS (Bispectral Index) scores from raw EEG, and implement the open-source BIS algorithm.

DISCUSSION
--

WHAT WORKED: 

1. γ-band features (log_gamma_lo, rel_gamma) drove RF performance,
   consistent with Connor 2022's finding that low-γ (30-47 Hz) is
   the primary BIS driver — not true bispectral features as the
   proprietary algorithm's name implies.

2. GroupKFold by patient is the single most important design
   decision. Epoch-level random splitting would leak correlated
   signals from the same patient into both train and test sets,
   artificially inflating R² and accuracy.

3. BSR (Burst Suppression Ratio) was critical for BIS < 40.
   Without it the model would fail entirely in the deep anesthesia
   zone where EEG becomes isoelectric.

4. Regression + Classification together tell a richer story:
   regression shows continuous tracking ability (MAE in BIS units),
   classification shows clinically actionable zone-level accuracy.

5. openibis provided a strong interpretable baseline. Its ROC-AUC
   without any training data shows how much structure is already
   recoverable from pure signal processing alone.
   
---

WHAT DIDN'T WORK / LIMITATIONS: 

1. openibis heuristic coefficients (0.60, 0.20, 0.15, 0.05) are
   approximations — the proprietary Aspect Medical weights are not
   published. This explains the MAE ~20 BIS units gap vs ground truth.

2. 24 cases is a small cohort. GroupKFold with 5 splits means each
   fold tests on ~5 patients — high variance in fold-level metrics.

3. Sample Entropy is O(N²) and was approximated for speed. A proper
   multiscale entropy implementation would likely improve feature
   quality, especially for detecting transitions.

4. No temporal smoothing — the real BIS monitor applies a 15-30 s
   rolling average. Our epoch-level predictions are noisier than
   what clinicians see on the monitor display.

5. Single frontal channel only. Multi-channel spatial features
   (e.g. asymmetry index) are used in some commercial systems.
   
---

WHAT I'D EXPLORE NEXT: 

1. LSTM / GRU temporal model — RF treats each epoch independently.
   A recurrent model would capture the induction → maintenance →
   emergence trajectory, which is highly predictable.

2. Per-patient z-score normalization — EEG amplitude baseline
   varies 5-10x between patients. Normalizing within-patient
   before cross-patient evaluation could significantly close the
   openibis vs RF gap.

3. Calibrate openibis coefficients on this dataset — fit the
   4 weights (p_high, p_vh, p_mid, p_low) by minimizing MAE
   against ground-truth BIS. This would be a 4-parameter linear
   regression on top of the signal features.

4. Cross-frequency coupling — θ-γ phase-amplitude coupling (PAC)
   decreases under anesthesia and is not captured by marginal
   power features alone.

5. Leave-one-subject-out (LOSO) evaluation — stricter than
   GroupKFold, gives the most realistic estimate of performance
   on a completely new patient.
