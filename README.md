# AI-Based Detection and Mitigation of Cyberattacks on OT/ICS
### Bachelor's Thesis — Cybersecurity Programme, GUtech
**Author:** AL-Yaqeen AL-Mahrouqi
**ID**: ** 21-0029
**Supervisor:** Dr. Abdelaziz Amara Korba  
**Co-Supervisor:** Dr. Ali Alhumairi  
**Submission Deadline:** June 1, 2026

---

## 1. Project Overview

This project implements a multi-detector anomaly detection and mitigation pipeline for Operational Technology / Industrial Control Systems (OT/ICS). Two real-world ICS datasets are used, covering a water treatment plant and a gas pipeline.

**Three detectors are implemented across all datasets:**
- Z-score with Median Absolute Deviation (MAD) + K-of-N voting (K=4, τ=5.0)
- Isolation Forest (n_estimators=200, FPR ≤ 5% constraint)
- LSTM Autoencoder (window W=30, reconstruction-error thresholding)

**Detection is followed by event construction** (cooldown T_cool=60s, minimum duration T_min=10s) and a four-level risk-limited mitigation framework.

---

## 2. Datasets

| # | Dataset | Domain | Protocol | Source |
|---|---------|--------|----------|--------|
| 1 | **SWaT** (Secure Water Treatment) | Water treatment | EtherNet/IP + CIP | iTrust, SUTD Singapore |
| 2 | **Morris Dataset 4** (IanArffDataset.arff) | Gas pipeline | Modbus TCP | Turnipseed 2014 via Mississippi State |

> **Important — Morris Dataset clarification:**  
> `IanArffDataset.arff` is the correct **Dataset 4** used in this thesis.

> **Important — SWaT data quality finding:**  
> "Event 1" in the Kaggle version is a 13.7-hour merged artifact that inflates published benchmarks. The evaluation window in this thesis starts at `2016-01-01 14:28:35`, explicitly excluding Event 1. This is a documented contribution of the thesis.

---

## 3. Repository Structure

```
thesis_submission/
│
├── README.md                          ← This file
├── requirements.txt                   ← Python dependencies
├── SUBMISSION_CHECKLIST.md            ← What is included / verified
│
├── notebooks/
│   ├── SWaT_Detection_Pipeline.ipynb  ← Full SWaT pipeline (all 3 detectors)
│   └── Morris_Detection_Pipeline.ipynb ← Morris Gas Pipeline v4
│
├── results/                           ← Pre-computed outputs (load without re-running)
│   ├── swat/
│   │   ├── zscore_baseline.pkl
│   │   ├── iforest_baseline.pkl
│   │   ├── lstmae_results_v2.pkl
│   │   ├── lstmae_model.keras
│   │   ├── feature_metadata_v2.pkl
│   │   ├── phase10_audit.pkl
│   │   └── split_info.pkl
│   └── morris/
│       └── (pkl files from v4 pipeline run)
│
└── data/                              ← Empty — see Section 2 for access
    ├── swat/   (place SWaT CSV files here)
    ├── morris/ (place IanArffDataset.arff here)
    └── hai/    (place HAI CSV files here)
```

---

## 4. Environment Setup

### Option A — Google Colab (Recommended for GPU)

```python
# Cell 1: Mount Drive (skip if running locally)
from google.colab import drive
drive.mount('/content/drive')

# Cell 2: Install dependencies
!pip install -r requirements.txt
```

### Option B — Local Machine

```bash
# Python 3.10+ required
pip install -r requirements.txt
```

> **Critical Colab note:** Do NOT pin numpy when TensorFlow 2.20 is present.  
> If you hit binary incompatibility errors, use **Factory Reset** (Runtime → Factory reset runtime), NOT a simple restart.

---

## 5. Path Configuration

Both notebooks use a path configuration block at the top. Edit this single block before running — you do not need to change anything else in the notebook.

```python
# ── PATH CONFIG (edit this block only) ──────────────────────────────────────
import os

# Set to True if running on Google Colab with Drive mounted
USE_DRIVE = False  # <── change to True on Colab

if USE_DRIVE:
    BASE = "/content/drive/MyDrive/Gaspipelinedataset"   # Morris
    SWAT_BASE = "/content/drive/MyDrive/SWaT"            # SWaT
    RESULTS_BASE = "/content/drive/MyDrive/results"
else:
    BASE = "./data/morris"
    SWAT_BASE = "./data/swat"
    RESULTS_BASE = "./results"

os.makedirs(RESULTS_BASE, exist_ok=True)
# ─────────────────────────────────────────────────────────────────────────────
```

---

## 6. How to Run

### 6.1 Load Pre-computed Results (Fastest — No GPU Needed)

Both notebooks include a **"Load Saved Results"** section. Run only those cells to reproduce all evaluation metrics and plots without re-training.

For SWaT, load in this order:
```python
import pickle, os
results_dir = "./results/swat"

with open(os.path.join(results_dir, "zscore_baseline.pkl"), "rb") as f:
    zscore = pickle.load(f)
with open(os.path.join(results_dir, "iforest_baseline.pkl"), "rb") as f:
    iforest = pickle.load(f)
with open(os.path.join(results_dir, "lstmae_results_v2.pkl"), "rb") as f:
    lstmae = pickle.load(f)
```

### 6.2 Full Re-run (Requires Dataset Files + GPU for LSTM-AE)

Run all cells top-to-bottom. Training the LSTM Autoencoder on SWaT takes ~20–40 minutes on Colab T4 GPU.

---

## 7. Key Results

### SWaT Dataset (36 attacks, 131,278 prediction rows)

| Detector | Row-level F1 | Event Recall | Notes |
|----------|-------------|--------------|-------|
| Z-score (MAD + K-of-N) | 0.79 | Event F1 = 0.55 | 6 drift sensors excluded from voting: AIT201/202/203, AIT402, AIT502, FIT601 |
| Isolation Forest | 0.09 | 2/8 events (precision = 1.000) | FPR constrained ≤ 5%; AUC = 0.61 |
| LSTM Autoencoder | — | 7/8 events (87.5% recall) | AUC = 0.87 |

### Morris Gas Pipeline Dataset 4 (IanArffDataset.arff)

Key implementation decisions in v4 pipeline:
- NMRI/CMRI attacks: floating-point overflow pressure values (~10³⁵) → handled with signed log transform
- MSCI/MPCI attacks: subtle (~3σ) → require careful threshold tuning
- Recon attacks: non-standard Modbus addresses
- ~75% of rows are command packets with NaN process variables by protocol design → NaN handled before feature extraction

> **Evaluation policy:** Point adjustment is explicitly excluded. All metrics are raw. FAR/hr is reported as a differentiating metric. This is a deliberate methodological contribution.

---

## 8. Contact

For questions about running the code or dataset access, contact the author through GUtech email (21-0029@student.gutech.edu.om).
