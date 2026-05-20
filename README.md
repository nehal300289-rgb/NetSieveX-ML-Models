# NetSieveX — ML Models

> Machine learning pipeline for IoT network intrusion detection using the RT-IoT2022 dataset.
---

## Overview

NetSieveX is a full end-to-end ML pipeline that trains, evaluates, and explains classification models for detecting network intrusions in IoT environments. The system classifies network traffic into four risk categories using a curated 17-feature subset derived from the RT-IoT2022 dataset.

This repository contains the model training and evaluation component. See the related repositories below for the full system.

| Component | Repository |
|-----------|-----------|
| Frontend | [WebSait](https://github.com/ingenriquecardosoalfonso/WebSait) |
| Backend API | [NetSieveXAPI](https://github.com/ingenriquecardosoalfonso/NetSieveXAPI) |

---

## Traffic Classification

| Class | Risk Level |
|-------|-----------|
| `Normal` | 🟢 LOW | 
| `NMAP` | 🟡 MEDIUM |
| `ARP_Poisoning` | 🔴 HIGH |
| `DOS_SYN_Hping` | 🔴 CRITICAL | 
---

## Dataset

**RT-IoT2022** — UCI Machine Learning Repository  
Sharmila, B. S., & Nagapadma, R. (2023). [https://doi.org/10.24432/C5P338](https://doi.org/10.24432/C5P338)

| Stage | Rows | Features |
|-------|------|----------|
| Original | 123,117 | 85 |
| After preprocessing | 37,264 | 17 |

---

## Pipeline

```
Raw Dataset (85 features)
        │
        ▼
 Feature Selection
 ├── Variance Threshold  (removes near-constant features)
 ├── Pearson Correlation (removes highly correlated features)
 └── Mutual Information  (ranks features by predictive value)
        │
        ▼
 17 Final Features
        │
        ▼
 Train / Test Split
 └── Stratified 80/20 (random_state=42)
        │
        ▼
 Model Training
 ├── Logistic Regression
 ├── Decision Tree
 ├── Random Forest
 ├── K-Nearest Neighbors (KNN)
 └── Support Vector Machine (SVM)
        │
        ▼
 Evaluation & Explainability
 └── Accuracy · ROC-AUC · Hamming Loss · SHAP Values
```

---

## Results

| Model | Accuracy | ROC-AUC | Hamming Loss |
|-------|----------|---------|--------------|
| **Random Forest** | **99.37%** | **0.9999** | **0.0063** |
| Decision Tree | 99.19% | 0.9945 | 0.0081 |
| KNN | 98.91% | 0.9984 | 0.0109 |
| SVM | 97.99% | 0.9955 | 0.0201 |
| Logistic Regression | 97.11% | 0.9920 | 0.0288 |

Random Forest achieves near-perfect classification across all four traffic classes, with a ROC-AUC of 0.9999 and a Hamming Loss of just 0.0063.

---

## Explainability

SHAP (SHapley Additive exPlanations) values are computed for all trained models to provide transparent, feature-level explanations for each prediction. This allows the system to highlight which network features most strongly influenced a given classification decision — critical for real-world security deployments where trust and interpretability matter.

---

## Getting Started

### Prerequisites

- Python 3.10+
- Jupyter Notebook or JupyterLab

### Installation

```bash
# Clone the repository
git clone https://github.com/your-username/netsievex-ml.git
cd netsievex-ml

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook
```

Open the notebook and run all cells to reproduce the full pipeline, from raw data ingestion through model evaluation and SHAP analysis.

---

## Project Structure

```
netsievex-ml/
├── data/                  # Dataset files (not tracked by git)
├── notebooks/             # Jupyter notebooks for the full pipeline
├── models/                # Serialized trained models
├── requirements.txt       # Python dependencies
└── README.md
```

---

## References

- Lundberg, S. M., & Lee, S. I. (2017). A unified approach to interpreting model predictions. *Advances in Neural Information Processing Systems*, 30.
- Sharmila, B. S., & Nagapadma, R. (2023). RT-IoT2022 [Dataset]. UCI Machine Learning Repository. [https://doi.org/10.24432/C5P338](https://doi.org/10.24432/C5P338)

---

## Team

Developed as part of the **Integrated Artificial Intelligence Post-Diploma Certificate** program at SAIT (2026).

- Nehal Gadhavi
- Barbara Alfaro
- Ludwig Cardoso 

---

*Part of the NetSieveX ecosystem — real-time IoT intrusion detection powered by machine learning.*
