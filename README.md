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

| Class | Risk Level | Description |
|-------|-----------|-------------|
| `Normal` | 🟢 LOW | Legitimate IoT traffic (MQTT, IoT device telemetry) |
| `NMAP` | 🟡 MEDIUM | Network reconnaissance scan (FIN, OS detection, TCP, UDP, XMAS) |
| `ARP_Poisoning` | 🔴 HIGH | ARP cache poisoning attack |
| `DOS_SYN_Hping` | 🔴 CRITICAL | SYN flood denial-of-service attack |
---

## Dataset

**RT-IoT2022** — UCI Machine Learning Repository  
Sharmila, B. S., & Nagapadma, R. (2023). [https://doi.org/10.24432/C5P338](https://doi.org/10.24432/C5P338)

| Stage | Rows | Features |
|-------|------|----------|
| Original | 123,117 | 85 |
| After preprocessing | 37,264 | 17 |

**Preprocessing decisions:**
- Dropped unnamed index column and random source port feature (`id.orig_p`)
- Removed 85,853 duplicate rows
- Replaced missing service values (`-`) with `Unidentified`
- Downsampled `DOS_SYN_Hping` to 10,000 rows to reduce class imbalance
- Grouped 3 normal IoT traffic types into `Normal`; grouped 5 NMAP scan variants into `NMAP`
- Removed two minority classes (`DDOS_Slowloris`, `Metasploit_Brute_Force_SSH`)
---

## Pipeline

```
Raw Dataset (85 features, 123,117 rows)
        │
        ▼
 ── Section 1: Data Quality ──────────────────
 ├── Drop index & constant columns
 ├── Remove duplicates (85,853 dropped)
 ├── Fix categorical anomalies
 └── Class grouping & downsampling
        │
        ▼
 ── Section 2: EDA ───────────────────────────
 ├── service vs. target analysis
 └── proto vs. target analysis
        │
        ▼
 ── Section 3: Feature Engineering ──────────
 ├── Variance Threshold (threshold=0.01)   → removes 3 near-constant features
 ├── Pearson Correlation (threshold=0.90)  → removes highly correlated features
 └── Mutual Information (top 15 scored)    → 17 final features selected
        │
        ▼
 ── Section 4: Model Training ────────────────
 ├── Stratified 80/20 split (random_state=42)
 └── 5 models trained in sklearn Pipelines
     ├── Logistic Regression
     ├── Decision Tree
     ├── Random Forest
     ├── K-Nearest Neighbors (KNN)
     └── Support Vector Machine (SVM)
        │
        ▼
 ── Section 5: Evaluation ────────────────────
 └── Accuracy · Hamming Loss · ROC-AUC · Precision · Recall · Confusion Matrix
        │
        ▼
 ── Section 6: Hyperparameter Tuning ─────────
 └── GridSearchCV on Random Forest (5-fold CV, 32 combinations)
        │
        ▼
 ── Section 7: Export & SHAP ─────────────────
 └── Serialize top 3 models (.pkl) + generate SHAP background sample
```

---
 
## Selected Features (17)
 
After the three-stage feature selection process, the following features were retained:
 
| # | Feature | Type |
|---|---------|------|
| 1 | `fwd_subflow_bytes` | Numeric |
| 2 | `fwd_pkts_payload.tot` | Numeric |
| 3 | `fwd_pkts_payload.avg` | Numeric |
| 4 | `flow_pkts_payload.avg` | Numeric |
| 5 | `id.resp_p` | Numeric |
| 6 | `flow_pkts_payload.std` | Numeric |
| 7 | `payload_bytes_per_second` | Numeric |
| 8 | `active.avg` | Numeric |
| 9 | `active.tot` | Numeric |
| 10 | `active.min` | Numeric |
| 11 | `flow_pkts_per_sec` | Numeric |
| 12 | `flow_iat.avg` | Numeric |
| 13 | `fwd_last_window_size` | Numeric |
| 14 | `fwd_pkts_payload.min` | Numeric |
| 15 | `fwd_header_size_tot` | Numeric |
| 16 | `service` | Categorical |
| 17 | `proto` | Categorical |
 
---

## Results

| Model | Accuracy | ROC-AUC | Hamming Loss |
|-------|----------|---------|--------------|
| **Random Forest** | **99.37%** | **0.9999** | **0.0063** |
| Decision Tree | 99.19% | 0.9945 | 0.0081 |
| KNN | 98.91% | 0.9984 | 0.0109 |
| SVM | 97.99% | 0.9955 | 0.0201 |
| Logistic Regression | 97.11% | 0.9920 | 0.0288 |

**Top 3 models selected for deployment:** Random Forest, Decision Tree, KNN.
 
Key findings:
- `DOS_SYN_Hping` achieves perfect recall (1.00) across all three top models — no attack was ever missed
- `ARP_Poisoning` is the hardest class, with partial overlap with `Normal` traffic patterns
- Hyperparameter tuning via GridSearchCV improved Random Forest accuracy by +0.03% (99.37% → 99.40%), confirming the default configuration was already near-optimal

---

## Explainability

SHAP (SHapley Additive exPlanations) values are computed for all three selected models to provide transparent, feature-level explanations for each prediction. A background sample of 100 transformed rows is exported to `data/background_sample.npy` for use by the backend API at inference time.
 
This allows the system to surface which specific network features drove a classification decision — critical for security deployments where analysts need to understand and trust model outputs.

---

## Notebook Structure
 
The analysis is contained in a single Jupyter notebook (`Project_Group4_1.ipynb`) organized into 7 sections:
 
| Section | Description |
|---------|-------------|
| `0. Configuration` | Constants, file paths, and library imports |
| `1. Data Quality Checks` | Load, clean, deduplicate, and preprocess the dataset |
| `2. EDA` | Explore categorical features (`service`, `proto`) vs. target |
| `3. Feature Engineering` | Three-stage feature selection (Variance → Correlation → MI) |
| `4. Model Training` | Train 5 classifiers using sklearn Pipelines |
| `5. Evaluation` | Accuracy, Hamming Loss, ROC-AUC, Confusion Matrix |
| `6. Hyperparameter Tuning` | GridSearchCV on Random Forest |
| `7. Web Dev Integration` | Serialize models and generate SHAP background data |
 
---

## Getting Started
 
### Prerequisites
 
- Python 3.10+
- Jupyter Notebook or JupyterLab
- RT-IoT2022 dataset downloaded as `RT_IOT2022.csv`

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

Open `Project_Group4_1.ipynb` and run all cells sequentially to reproduce the full pipeline — from raw data ingestion through model evaluation, hyperparameter tuning, and SHAP export.


---

## Project Structure

```
netsievex-ml/
├── Project_Group4_1.ipynb     # Full ML pipeline notebook
├── data/
│   └── background_sample.npy  # SHAP background (generated by notebook)
├── models/
│   ├── random_forest.pkl       # Serialized Random Forest pipeline
│   ├── decision_tree.pkl       # Serialized Decision Tree pipeline
│   └── knn.pkl                 # Serialized KNN pipeline
├── requirements.txt            # Python dependencies
└── README.md
```
 
> **Note:** `RT_IOT2022.csv` and `clean_dataset.csv` are not tracked by git. Download the dataset from the [UCI repository](https://doi.org/10.24432/C5P338) and place it in the root directory before running the notebook.

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
