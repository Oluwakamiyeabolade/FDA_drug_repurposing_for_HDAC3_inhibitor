# In Silico Identification of Potential HDAC3 Inhibitors Through Machine Learning, Molecular Docking, and Molecular Dynamics Simulations for Drug Repurposing

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue)](https://www.python.org/)
[![RDKit](https://img.shields.io/badge/RDKit-2023-green)](https://www.rdkit.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-orange)](https://scikit-learn.org/)

This repository contains the machine learning and cheminformatics pipeline from the study:

> **"In silico identification of potential HDAC3 inhibitors through machine learning, molecular docking, and molecular dynamics simulations for drug repurposing"**  
> *Informatics in Medicine Unlocked*, 2025  
> DOI: [10.1016/j.imu.2025.101659](https://doi.org/10.1016/j.imu.2025.101659) (pii: S2949688825000309)  
> Published: 2025

---

## Overview

**Histone Deacetylase 3 (HDAC3)** is an epigenetic enzyme involved in cell cycle regulation, apoptosis, and gene expression. Its overexpression is associated with breast cancer progression, making it a high-value therapeutic target. This project applies a **drug repurposing** strategy to identify known approved drugs that could be repositioned as HDAC3 inhibitors, using a fully in silico pipeline:

1. **ML Classification** — Train ensemble classifiers on 576 known HDAC3 inhibitors from ChEMBL to distinguish Strong vs. Weak Inhibitors
2. **Virtual Screening** — Score ~4,288 ZINC15 non-FDA compounds using the trained model
3. **Drug-likeness Filtering** — Apply Lipinski Ro5, QED (≥ 0.6), and SA Score (< 5.0) filters
4. **Molecular Docking** — Top hits docked against HDAC3 crystal structure (PDB: 4A69); reference ligand BG45 used as benchmark (−4.18 kcal/mol)
5. **MD Simulation** — 100 ns molecular dynamics to confirm stability of top complexes (described in the paper)

The top hit, **ZINC000095618609**, showed a docking score of **−8.81 kcal/mol** and stable MD behavior, outperforming the reference ligand.

---

## Repository Structure

```
hdac3-drug-repurposing/
├── notebooks/
│   ├── ML_-_classification.ipynb                      # ML model training + evaluation on ChEMBL HDAC3 data
│   └── FDA_drug_repurposing_for_HDAC3_inhibitor.ipynb # Virtual screening + Ro5/QED/SA filtering pipeline
├── data/
│   ├── CHEMBL_HDAC3_inhibitors.csv   # 576 HDAC3 inhibitors from ChEMBL (PDB: 4A69); training set
│   └── world-not-fda.csv             # 4,288 non-FDA ZINC15 compounds for virtual screening
├── results/
│   └── figures/                      # Output plots (EDA, confusion matrix, ROC, feature importance)
├── requirements.txt
└── README.md
```

---

## Methodology

### 1. Data & Labeling (`ML_-_classification.ipynb`)
- **576 HDAC3 inhibitors** from ChEMBL (target: PDB 4A69 / CHEMBL2111363)
- Binary classification threshold: **pIC50 ≥ 6.5 → Strong Inhibitor**, else Weak Inhibitor
- Full RDKit descriptor set computed (~200 descriptors per molecule)

### 2. Feature Selection (`ML_-_classification.ipynb`)
A multi-step pipeline to reduce dimensionality before model training:

| Step | Method | Threshold |
|---|---|---|
| Remove correlated features | Pearson correlation | r ≥ 0.9 |
| Remove low-variance features | VarianceThreshold | 0.1 |
| Scale features | MinMaxScaler | — |
| Select top features | SelectKBest (mutual_info_classif) | k = 10 |

**Top 10 selected descriptors:** BCUT2D_MWHI, BCUT2D_MRHI, SMR_VSA10, SMR_VSA3, SMR_VSA7, SlogP_VSA2, TPSA, VSA_EState2, VSA_EState4, VSA_EState5

### 3. Model Training & Evaluation (`ML_-_classification.ipynb`)
Four ensemble classifiers trained on an 80/20 stratified train-test split:

| Model | Notes |
|---|---|
| Random Forest | Best performer; saved as `best_model.joblib` |
| AdaBoost | — |
| Gradient Boosting | — |
| Extra Trees | — |

Evaluation metrics: Accuracy, Classification Report, Confusion Matrix, ROC-AUC

### 4. Virtual Screening (`FDA_drug_repurposing_for_HDAC3_inhibitor.ipynb`)
- Descriptor calculation for all 4,288 ZINC15 compounds using the same 10 features
- Prediction using the saved `best_model.joblib`
- Compounds predicted as **Strong Inhibitors** are shortlisted

### 5. Multi-Stage Drug-Likeness Filtering (`FDA_drug_repurposing_for_HDAC3_inhibitor.ipynb`)

```
4,288 ZINC15 compounds
        ↓ ML prediction (Strong Inhibitor)
        ↓ Lipinski Ro5 filter (zero violations)
        ↓ QED ≥ 0.6
        ↓ SA Score < 5.0
        → Final candidates → molecular docking
```

### 6. Molecular Docking & MD Simulation (see paper)
- Top candidates docked against HDAC3 crystal structure (PDB: 4A69)
- Reference ligand: BG45 (docking score: −4.18 kcal/mol)
- Top hit ZINC000095618609: docking score **−8.81 kcal/mol**
- MD simulation: 100 ns to assess complex stability

---

## Results Summary

| Output File | Description |
|---|---|
| `best_model.joblib` | Trained Random Forest classifier |
| `Features.csv` | Top 10 selected features with mutual information scores |
| `top_fda_compounds_new.csv` | ML-predicted Strong Inhibitors from ZINC15 |
| `RO5filtered_top_external_compounds.csv` | After Lipinski Ro5 filtering |
| `QEDfiltered_RO5_top_external_compounds.csv` | After QED ≥ 0.6 filtering |
| `sa_filtered_compounds.csv` | Final candidates after SA Score < 5.0 filtering |
| `results/figures/` | EDA plots, confusion matrix, ROC curve, feature importance bar chart |

---

## Installation

```bash
git clone https://github.com/YOUR_USERNAME/hdac3-drug-repurposing.git
cd hdac3-drug-repurposing
pip install -r requirements.txt
```

> RDKit is best installed via conda:
> ```bash
> conda install -c conda-forge rdkit
> ```

---

## Usage

Run notebooks in order:

```bash
# Step 1: Train the ML model
jupyter notebook notebooks/ML_-_classification.ipynb

# Step 2: Run virtual screening and drug-likeness filtering
jupyter notebook notebooks/FDA_drug_repurposing_for_HDAC3_inhibitor.ipynb
```

Both notebooks expect data files in the `data/` directory and will write outputs to the working directory and `results/figures/`.

---

## Dependencies

See [`requirements.txt`](requirements.txt).

---

## Citation

If you use this code, please cite:

```bibtex
@article{hdac3repurposing2025,
  title={In silico identification of potential HDAC3 inhibitors through machine learning, molecular docking, and molecular dynamics simulations for drug repurposing},
  journal={Informatics in Medicine Unlocked},
  year={2025},
  doi={10.1016/j.imu.2025.101659}
}
```

---

