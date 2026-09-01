# Sequential Record Linkage Framework for Pregnancy Episode Reconstruction

Code accompanying **Chapter 4** of [Thesis title / manuscript title], reconstructing pregnancy episodes from fragmented routine health registers lacking reliable patient identifiers, developed and validated within the **MiMBa Pregnancy Exposure Registry** (42 healthcare facilities, Homa Bay County, western Kenya).

> **Citation:** [Author names]. [Manuscript title]. [Journal / Thesis, year]. DOI: [xxx]

---

## Overview

Routine antenatal, maternity, outpatient, and inpatient registers in low-resource settings are rarely linked by a shared unique identifier. This repository implements a **three-stage, precision-first record linkage framework** that reconstructs longitudinal pregnancy episodes from these fragmented records without relying on a complete patient identifier.

| Stage | Task | Method |
|---|---|---|
| 1 | Resolve pregnancies lost to follow-up (LTFU) / missing birth outcomes | XGBoost classifier (candidate scoring) + clinical decision rules |
| 2 | Cluster orphan antenatal care (ANC) records into pregnancy-level groups | Deterministic rule-based matching + Fellegi-Sunter probabilistic linkage ([Splink](https://github.com/moj-analytical-services/splink)) |
| 3 | Link ANC clusters, inpatient, and outpatient records to pregnancy episodes | XGBoost classifier + Hungarian algorithm (global one-to-one assignment) |

Each stage is deliberately conservative — tolerating missed matches over false matches — to minimise false attribution of medication exposures to the wrong pregnancy, which is essential for downstream pharmacovigilance analyses.

---

## Repository structure

```
.
├── data/                     # NOT included — see "Data availability" below
├── src/
│   ├── stage1_ltfu_resolution/
│   │   ├── train_xgboost_ltfu.py
│   │   ├── clinical_decision_rules.py
│   │   └── config.yaml
│   ├── stage2_anc_clustering/
│   │   ├── deterministic_clustering.py
│   │   ├── probabilistic_linkage_splink.py
│   │   └── config.yaml
│   ├── stage3_episode_linkage/
│   │   ├── train_xgboost_episode_linkage.py
│   │   ├── hungarian_assignment.py
│   │   └── config.yaml
│   └── utils/
│       ├── feature_engineering.py
│       ├── validation_metrics.py       # precision, recall, F1, cluster purity
│       └── plots.py
├── notebooks/
│   └── validation_and_figures.ipynb    # reproduces Tables 4.2–4.5 and Figures
├── requirements.txt
├── environment.yml                     # optional conda environment
└── README.md
```

*(Adjust the above to match your actual file names before publishing — this is a template based on the pipeline described in the manuscript, not a listing of your real files.)*

---

## Requirements

- Python `[fill in version, e.g. 3.11]`
- Key packages:
  - `xgboost`
  - `splink` (Fellegi-Sunter probabilistic linkage)
  - `scikit-learn`
  - `pandas`, `numpy`
  - `scipy` (Hungarian algorithm — `scipy.optimize.linear_sum_assignment`, if that's the implementation used)
- `[Add R version and packages here if any part of the pipeline — e.g. summary tables or plotting — was done in R, since the manuscript doesn't currently specify this.]`

Install dependencies:

```bash
pip install -r requirements.txt
```

or, if using conda:

```bash
conda env create -f environment.yml
conda activate [env-name]
```

---

## Usage

```bash
# Stage 1: resolve LTFU / missing birth outcome episodes
python src/stage1_ltfu_resolution/train_xgboost_ltfu.py --config src/stage1_ltfu_resolution/config.yaml

# Stage 2: cluster orphan ANC records
python src/stage2_anc_clustering/deterministic_clustering.py
python src/stage2_anc_clustering/probabilistic_linkage_splink.py

# Stage 3: link ANC clusters, inpatient, and outpatient records to episodes
python src/stage3_episode_linkage/train_xgboost_episode_linkage.py
python src/stage3_episode_linkage/hungarian_assignment.py
```

*(Replace with your actual CLI/script invocation once finalised.)*

---

## Data availability

The underlying MiMBa Pregnancy Exposure Registry data are **not included in this repository** due to participant privacy and the data governance terms of the registry. Access requests can be directed to [contact / data access process, e.g. the MiMBa study PIs or KEMRI-CGHR data access committee].

This repository contains code only, intended to allow the linkage methodology to be reviewed, reused, and adapted to other routine health datasets with a similar structure (LMP, parity, facility code, gestational age indicators, etc. — see Chapter 4, §4.4 "Framework's core architecture").

---

## Validation summary

Key validation metrics reported in the manuscript (see Table 4.3 and §4.10.3–4.10.4 for full detail):

- Stage 1 (LTFU resolution): mean AUC-PR 0.9995, F1 0.988 (5-fold cross-validation)
- Stage 2 (ANC clustering): pair-level precision 0.992, recall 0.705, F1 0.824; cluster purity 0.997
- Stage 3 (episode linkage): AUC-PR 0.9991 (cross-validation), AUC-PR 1.0000 (external validation)

## License

`[Add license, e.g. MIT, once decided]`

## Contact

`[Your name / email / ORCID]`
