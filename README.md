# 🔬 Cadmium Sensor ML — Electrochemical Concentration Prediction

[![Python](https://img.shields.io/badge/Python-3.8+-blue?logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-orange?logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![XGBoost](https://img.shields.io/badge/XGBoost-1.x-red)](https://xgboost.readthedocs.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> A supervised machine learning pipeline that maps electrochemical sensor responses from **Cyclic Voltammetry (CV)** measurements to cadmium ion (Cd²⁺) concentrations ranging from **0 to 1000 ppm**, using Random Forest and XGBoost regression models.

---

## 📌 Table of Contents

- [Background](#-background)
- [Dataset](#-dataset)
- [Methodology](#-methodology)
- [Project Structure](#-project-structure)
- [Installation](#-installation)
- [Usage](#-usage)
- [Results](#-results)
- [Future Work](#-future-work)
- [Author](#-author)

---

## 🧪 Background

Cadmium (Cd²⁺) is a toxic heavy metal found in industrial wastewater, agricultural runoff, and contaminated drinking water. Accurate and rapid detection is critical for environmental monitoring and public health.

This project applies machine learning to **Cyclic Voltammetry (CV)** — an electrochemical technique that measures current response as a function of applied voltage — to build a predictive model capable of estimating Cd²⁺ concentration from raw sensor signals. This approach enables faster and more automated analysis compared to traditional calibration curve methods.

---

## 📂 Dataset

The dataset was collected using an **EmStat potentiostat** and consists of CV measurements across 16 concentration levels.

### Folder Structure

```
data/
├── 2/          → Cad_2ppm_1.xlsx ... Cad_2ppm_50.xlsx
├── 4/          → Cad_4ppm_1.xlsx ... Cad_4ppm_50.xlsx
├── 6/
├── ...
├── 1000/       → Cad_1000ppm_1.xlsx ... Cad_1000ppm_50.xlsx
└── KCL/        → KCL_EMSTAT_01.xlsx ... KCL_EMSTAT_50.xlsx
```

### File Format

Each `.xlsx` file contains **10 CV scans**, where each scan has two columns:

| Column | Description |
|--------|-------------|
| `V` | Applied voltage (Volt), range ≈ −0.5 V to +0.5 V |
| `µA` | Measured current (microampere) — the sensor response |

```
| CV i vs E Scan 1 |    | CV i vs E Scan 2 |    | ... | CV i vs E Scan 10 |    |
|       V          | µA |        V         | µA | ... |        V          | µA |
|    -0.4999       |... |     -0.4999      |... | ... |      -0.4999      |... |
```

### Dataset Summary

| Property | Value |
|----------|-------|
| Concentration levels | 16 (0, 2, 4, 6, 8, 10, 20, 40, 60, 80, 100, 200, 400, 600, 800, 1000 ppm) |
| Files per concentration | 50 |
| Total samples | ~800 |
| Features (X) | 181 µA values from Scan 10 |
| Target (Y) | Concentration in ppm (continuous) |

> **Why Scan 10?** The final scan represents the most electrochemically stable state of the electrode, minimizing noise from the equilibration process in earlier scans.

> **Why KCL = 0 ppm?** KCl solution serves as the blank electrolyte (no Cd²⁺ present), representing the baseline sensor response.

---

## ⚙️ Methodology

```
Raw Excel Files
      │
      ▼
Extract µA from Scan 10 (181 data points per file)
      │
      ▼
Assign label Y from folder name (ppm)
      │
      ▼
Merge all files → DataFrame (800 × 182)
      │
      ▼
Train/Test Split (80:20)
      │
      ▼
Train Models: Random Forest | XGBoost
      │
      ▼
Evaluate: MAE | RMSE | R²
```

### Feature Engineering

- **X**: 181 µA values extracted from Scan 10 of each file (one value per voltage point)
- **Y**: Cadmium concentration in ppm
- Voltage (V) is used as an implicit index — it is identical across all samples, so it carries no discriminative information and is excluded as a feature

### Models

| Model | Rationale |
|-------|-----------|
| **Random Forest Regressor** | Robust to high-dimensional data, no scaling required, interpretable via feature importance |
| **XGBoost Regressor** | High predictive performance, built-in regularization, handles correlated features well |

### Evaluation Metrics

| Metric | Description |
|--------|-------------|
| **MAE** | Mean Absolute Error — average prediction error in ppm |
| **RMSE** | Root Mean Squared Error — penalizes large errors more heavily |
| **R²** | Coefficient of determination — proportion of variance explained (0–1) |

---

## 📁 Project Structure

```
cadmium-sensor-ml/
│
├── data/                   # Raw Excel files (organized by concentration)
│   ├── 2/
│   ├── 4/
│   ├── ...
│   └── KCL/
│
├── notebooks/
│   ├── 01_data_loading.ipynb
│   ├── 02_eda.ipynb
│   └── 03_modeling.ipynb
│
├── src/
│   ├── load_data.py        # Data loading and merging pipeline
│   ├── train.py            # Model training and evaluation
│   └── predict.py          # Inference on new samples
│
├── outputs/
│   └── dataset_cadmium.csv # Merged dataset
│
├── requirements.txt
└── README.md
```

---

## 🛠️ Installation

```bash
# Clone repository
git clone https://github.com/yourusername/cadmium-sensor-ml.git
cd cadmium-sensor-ml

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

**requirements.txt**
```
pandas
numpy
scikit-learn
xgboost
openpyxl
matplotlib
seaborn
jupyter
```

---

## 🚀 Usage

### 1. Load and Merge All Data

```python
from src.load_data import load_all_data

X, y, meta = load_all_data(root_folder="data/")

print(f"X shape: {X.shape}")   # (800, 181)
print(f"y shape: {y.shape}")   # (800,)
```

### 2. Train Models

```python
from src.train import train_and_evaluate

results = train_and_evaluate(X, y)
```

### 3. Export Dataset to CSV

```python
import pandas as pd

df = pd.DataFrame(X, columns=[f"uA_V{i+1}" for i in range(X.shape[1])])
df['konsentrasi_ppm'] = y
df.to_csv("outputs/dataset_cadmium.csv", index=False)
```

---

## 📊 Results

> Results will be updated after full dataset training.

| Model | MAE (ppm) | RMSE (ppm) | R² |
|-------|-----------|------------|-----|
| Random Forest | — | — | — |
| XGBoost | — | — | — |

---

## 🔭 Future Work

- [ ] Hyperparameter tuning with GridSearchCV / Optuna
- [ ] Compare additional models: SVR, PLS Regression, 1D CNN
- [ ] Explainability analysis with SHAP values — identify which voltage regions are most predictive
- [ ] Deploy as a simple web app using Streamlit for real-time concentration prediction
- [ ] Extend to multi-analyte detection (e.g., simultaneous Cd²⁺ and Pb²⁺)

---

## 👤 Author

**[Your Name]**
- 📧 Email: your@email.com
- 💼 LinkedIn: [linkedin.com/in/yourprofile](https://linkedin.com)
- 🐙 GitHub: [github.com/yourusername](https://github.com)

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

> *This project was developed as part of a portfolio in machine learning applied to electrochemical sensor data.*