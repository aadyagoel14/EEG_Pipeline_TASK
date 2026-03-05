# EEG_Pipeline_TASK

Build an ML pipeline to predict BIS (Bispectral Index) scores from raw EEG, and implement the open-source BIS algorithm.

## Instructions to run code:

1. Clone or download the GitHub repository that contains the Jupyter notebook.
2. Download the dataset and place it in the project directory named "data".
3. Open the notebook:

```bash
jupyter notebook
```

4. Open the provided notebook file and run all cells from top to bottom.

No additional configuration is required, as the notebook loads all files directly from the `data/` folder.

---

## Approach

### 1. Data loading and exploration

All 24 `.mat` files (`case1.mat`–`case24.mat`) were loaded programmatically. For each case, raw EEG, BIS values, and sampling rates were extracted, handling minor variations in field names across files. Initial exploration included:

* Distribution of BIS values across all cases
* Example BIS time series
* Raw vs filtered EEG visualization

This confirmed the expected BIS range (0–100), nonstationary EEG structure, and the need for temporal windowing.

---

### 2. Preprocessing and epoching

EEG signals were preprocessed using standard clinical EEG steps:

* **Band-pass filter:** 0.5–47 Hz
* **Notch filter:** 50/60 Hz (power-line interference)
* **Epoching:** 5-second windows with 1-second stride

Each EEG epoch was aligned with BIS by interpolating the BIS signal to EEG time and assigning the **median BIS value** within each epoch. Epochs containing NaNs or extreme artifacts were discarded.

---

### 3. Feature extraction

From each EEG epoch, interpretable time- and frequency-domain features were computed, including:

* Relative band powers (δ, θ, α, β, low-γ)
* Spectral Edge Frequency (SEF95)
* Approximate Sample Entropy (SampEn surrogate)
* Log-transformed γ-band powers

These features were chosen to align with known BIS drivers while remaining computationally lightweight.

---

### 4. Machine learning model

A **Random Forest** was used for both regression and classification:

* **Regression:** Predict continuous BIS values
* **Classification:** Predict BIS zones

  * Deep (<40), General (40–60), Light (>60)

To prevent data leakage, evaluation used **GroupKFold**, grouping by patient (case ID), ensuring no subject appears in both training and test folds.

Metrics reported:

* Regression: MAE, R²
* Classification: Accuracy, ROC-AUC, confusion matrix

---

### 5. Open-source BIS (openibis) implementation

A simplified implementation inspired by Connor (2022) was built using four fixed spectral features (β ratio, γ ratio, entropy surrogate). No learning or calibration was applied.

For **1–2 representative cases**, openibis-derived BIS scores were compared directly against ground-truth BIS using:

* Time-series overlay
* Pearson correlation
* Mean Absolute Error (MAE)

This provided a mechanistic baseline against which the learned RF model was evaluated.

---

## Discussion

### What the results show

The pipeline achieved **MAE = 6.4 BIS units** and **R² = 0.664** on regression, and **73.2% accuracy / ROC-AUC = 0.887** on classification — both evaluated on held-out patients via GroupKFold, making these realistic out-of-sample estimates.

The openibis algorithm, with **no training at all**, reached 47.7% accuracy and ROC-AUC = 0.645 — a meaningful baseline given it uses only 4 fixed spectral coefficients. The RF gains **+25.5% accuracy and +0.241 AUC** by learning from labelled data.

Notably, from the feature correlation plot, **SampEn and SEF95 had the highest Pearson r with BIS (~0.6)**, while from the RF importance plot **rel_gamma and log_gamma_lo dominated** — consistent with Connor (2022)'s finding that low-γ (30–47 Hz) is the primary BIS driver, not true bispectral features.

The regression MAE by zone bar chart shows the regressor struggles most in the **Deep zone (MAE = 10.5)** — where burst suppression creates a near-isoelectric EEG that is difficult to differentiate without explicit BSR features.

---

### What didn't work / Limitations

* openibis coefficients are heuristic; proprietary BIS weights are unpublished
* Small cohort (24 cases) → fold-level variance
* Single frontal EEG channel vs multi-electrode commercial BIS
* No temporal smoothing; real BIS applies 15–30 s averaging

---

### What I'd explore next

1. **LSTM / GRU** to capture anesthesia state transitions
2. **Per-patient normalization** to reduce inter-subject EEG variance
3. **Calibrated openibis weights** via 4-parameter regression
4. **Cross-frequency coupling (PAC)** features
5. **Leave-One-Subject-Out (LOSO)** validation for clinical realism

---

### References

* Connor CW (2022). *Open Reimplementation of the BIS Algorithms for Depth of
  Anesthesia.* Anesthesia & Analgesia, 135(4):855–864. PMID: 35767469
* Li Ma (2017). EEG and BIS raw data. figshare.
  [https://doi.org/10.6084/m9.figshare.5589841](https://doi.org/10.6084/m9.figshare.5589841)
