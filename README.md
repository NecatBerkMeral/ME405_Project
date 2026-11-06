# Note: THIS README WAS PRODUCED VIA AI.
# Physics-Informed Non-Invasive Tool Wear Classification in CNC Milling

[[https://img.shields.io/badge/python-3.8+-blue.svg]](https://www.python.org/downloads/)
[[https://img.shields.io/badge/License-MIT-yellow.svg]](https://opensource.org/licenses/MIT)
[[https://img.shields.io/badge/Dataset-NASA%20Ames-red.svg]](https://www.nasa.gov/intelligent-systems-division/discovery-and-systems-health/pcoe/pcoe-data-set-repository/)

> **ME405/505 Machine Learning for Mechanical Engineering · Term Project**  
> **Author:** Necat Berk Meral  
> **Institution:** Department of Mechanical Engineering, Bilkent University  

---

## 📌 Project Overview

In high-precision discrete manufacturing, cutting tool wear accounts for up to **20% of unscheduled machining downtime** and conservative tool-change intervals prematurely discard inserts with **15–20% residual useful life**. Conventional Tool Condition Monitoring (TCM) systems rely on external piezoelectric dynamometers or acoustic emission transducers, which are prohibitively expensive ($10,000+), structurally invasive, and fragile in production environments.

This repository implements a **strictly non-invasive, physics-informed machine learning framework** to classify face milling insert degradation at the international critical tool-replacement boundary (**$VB \ge 0.30$ mm**, ISO 8688-1) using internal **spindle motor current signals** ($I_{AC,RMS}$ and $\bar{I}_{DC}$) from the benchmark NASA Ames Milling Dataset.

---

## 🔬 Core Methodology & Engineering Highlights

1. **Non-Invasive Feature Engineering (9 Features):**
   - **Kinematic Descriptors:** Depth of cut ($DOC \in [0.75, 1.50]$ mm), Feed rate ($feed \in [0.25, 0.50]$ mm/rev).
   - **Material Properties:** Brinell Hardness ($HB$), Elastic Modulus ($E$) for Cast Iron and Steel 4340.
   - **Electrical Signatures:** Spindle AC current RMS ($I_{AC,RMS}$), DC current mean ($\bar{I}_{DC}$), AC crest factor.
   - **Physics-Informed Proxies:** 
     - Material Removal Rate: $MRR = DOC \times feed \times v_c$
     - Specific Cutting Energy Proxy: $u_s = \frac{I_{AC,RMS}}{MRR + \epsilon}$ (isolates tool rubbing friction from volumetric cutting load).

2. **Leak-Free Preprocessing & Stratification:**
   - Removal of corrupted sensor acquisition artifacts ($I_{AC} > 10^{20}$ A).
   - Clean isolation of 21 unmeasured runs ($VB = \text{NaN}$) to prevent synthetic label leakage.
   - Stratified 60% Train ($n=87$), 20% Validation ($n=29$), and 20% untouched Holdout Test split ($n=29$).
   - `StandardScaler` fitted strictly on the training partition.

3. **Transparent & Sparse Modeling:**
   - **Balanced Logistic Regression ($L_2$ & $L_1$ Lasso):** Full physical explainability via standardized log-odds weights ($w_j$).
   - **Automated Feature Sparsity:** $L_1$ Lasso at optimal $C = 1.0$ zeroes out redundant parameters (`feed` and `crest factor`), retaining a compact 7-feature model with zero performance loss.

---

## 📊 Key Results

| Model Pipeline | Accuracy | Macro F1 | Healthy Recall | Worn Recall ($VB \ge 0.30$ mm) | Active Features |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Dummy Baseline (Most Freq.)** | 51.72% | 0.3409 | 100.0% | 0.0% | 0 / 9 |
| **Logistic Regression ($L_2$, Full)** | **89.66%** | **0.8966** | 86.67% | **92.86%** | 9 / 9 |
| **Lasso Logistic ($L_1$, $C=1.0$)** | **89.66%** | **0.8966** | 86.67% | **92.86%** | **7 / 9 (Sparse)** |

- **Absolute Gain:** **+37.93% accuracy improvement** over naive baseline.
- **Catastrophic Failure Prevention:** Successfully detected **13 out of 14 degraded tools (92.86% recall)** on the unseen validation partition.
- **Feature Dominance:** Standardized coefficients confirm that motor electrical signals heavily govern decisions (average $\vert{}w\vert{} = 1.18$) compared to raw process kinematics (average $\vert{}w\vert{} = 0.45$).

---
