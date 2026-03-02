# EEG_Pipeline_TASK
Build an ML pipeline to predict BIS (Bispectral Index) scores from raw EEG, and implement the open-source BIS algorithm.

## Instructions to run code:
1. Clone or download the GitHub repository that contains the Jupyter notebook.
2. Download the dataset and place it in the project directory.
3. Open the notebook:
```bash
jupyter notebook
```
4. Open the provided notebook file and run all cells from top to bottom.
No additional configuration is required, as the notebook loads all files directly from the `data/` folder.

## Approach:



## Discussion: 

### What the results show

The pipeline achieved **MAE = 6.4 BIS units** and **R² = 0.664** on regression, 
and **73.2% accuracy / ROC-AUC = 0.887** on classification — both evaluated on 
held-out patients via GroupKFold, making these realistic out-of-sample estimates.

The openibis algorithm, with **no training at all**, reached 47.7% accuracy and 
ROC-AUC = 0.645 — a meaningful baseline given it uses only 4 fixed spectral 
coefficients. The RF gains **+25.5% accuracy and +0.241 AUC** by learning from 
labelled data.

Notably, from the feature correlation plot, **SampEn and SEF95 had the highest 
Pearson r with BIS (~0.6)**, while from the RF importance plot **rel_gamma and 
log_gamma_lo dominated** — consistent with Connor (2022)'s finding that 
low-γ (30–47 Hz) is the primary BIS driver, not true bispectral features.

The regression MAE by zone bar chart (Section 5b) shows the regressor struggles 
most in the **Deep zone (MAE = 10.5)** — this is where burst suppression creates 
near-isoelectric EEG that is hard to differentiate from other deep states without 
BSR.

### What didn't work / Limitations

- openibis coefficients are heuristic — the proprietary Aspect Medical weights 
  are unpublished, explaining the MAE ~20 BIS units vs ground truth
- 24 cases is a small cohort; fold-level accuracy ranged from 61–80%, showing 
  high variance
- Single frontal EEG channel; commercial BIS uses 4 electrodes with spatial 
  averaging
- No temporal smoothing — the real BIS monitor applies a 15–30 s rolling window; 
  our epoch predictions are noisier

### What I'd explore next

1. **LSTM / GRU** — capture induction → maintenance → emergence state transitions 
   that RF treats as independent epochs
2. **Per-patient z-score normalisation** — EEG amplitude varies 5–10× between 
   patients; within-patient normalisation would reduce inter-subject variance
3. **Calibrate openibis coefficients** — fit the 4 spectral weights by minimising 
   MAE on this dataset; a 4-parameter regression on top of the signal features
4. **Cross-frequency coupling** — θ-γ phase-amplitude coupling (PAC) decreases 
   under anesthesia and is not captured by marginal power features
5. **Leave-one-subject-out (LOSO)** — stricter than GroupKFold; gives the most 
   conservative and realistic estimate for clinical deployment

### References

- Connor CW (2022). *Open Reimplementation of the BIS Algorithms for Depth of 
  Anesthesia.* Anesthesia & Analgesia, 135(4):855-864. PMID: 35767469
- Li Ma (2017). EEG and BIS raw data. figshare. 
  https://doi.org/10.6084/m9.figshare.5589841
