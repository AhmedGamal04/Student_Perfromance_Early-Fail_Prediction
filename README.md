# 🎓 StudiX — Student Performance Early-Warning System

A decision-support system that identifies, at the post-midterm point in a semester,
which students are at risk of failing — built end-to-end from raw data cleaning through
machine learning to an interactive Power BI dashboard.

> **Note:** this is a decision-support tool intended for use by academic advisors, not an
> automated determination of a student's outcome.

---

## 📌 Project overview

| | |
|---|---|
| **Dataset** | 20,000 students × 41 variables |
| **Primary task** | Post-midterm Pass/Fail early-warning classification |
| **Secondary task** | Final score regression |
| **Final model** | Logistic Regression (class-weighted) |
| **Test-set recall (Fail class)** | 88.1% |
| **Deliverables** | Cleaned dataset, ML notebook, PDF report, Power BI dashboard |

**The headline finding:** most demographic and psychological variables (stress, sleep,
motivation, previous GPA) carried negligible predictive signal in this dataset. Real
signal was concentrated in just four features — midterm score, quiz average, lecture
attendance rate, and assignment submission rate. A simple, interpretable **Logistic
Regression model outperformed a tuned LightGBM model on every evaluation metric**,
reinforcing that model complexity should be justified by the data, not assumed.

---

## 📂 Repository structure

```
.
├── student_data_cleaned.xlsx          # Cleaned source dataset (20K students × 41 vars)
├── student_data.xlsx          # Source dataset (20K students × 41 vars)
├── cleaning_documentation.pdf         # Data cleaning & imputation decisions, column by column
├── student_performance_project_report.pdf   # Full project report (LaTeX-generated)
├── Studix_Student_Performance_Presentation.pptx   # Full project presentation
├── student-performance-early-warning-prediction.ipynb   # Full ML pipeline, cell by cell
├── dashboard.jpeg                     # StudiX Power BI dashboard (screenshot)
├── studix_dashboard.pbix              # Power BI source file (add your .pbix here)
└── README.md
```

---

## 🧭 Pipeline

```
Raw data (20K students, 41 vars)
        │
        ▼
Data cleaning & governance   ── median imputation (numeric), "Unknown" (categorical),
        │                        targets never imputed
        ▼
EDA & data-leakage audit     ── verified final_grade/pass_fail are exact deterministic
        │                        thresholds of final_score; excluded from features
        ▼
Feature engineering          ── engagement composite, academic risk score, etc.
        │                        (tested, not assumed, for signal)
        ▼
ML modeling                  ── Logistic Regression, Random Forest, LightGBM,
        │                        5-fold stratified CV + hyperparameter tuning
        ▼
Evaluation & explainability  ── PR-AUC/recall focus, SHAP, subgroup fairness checks
        │
        ▼
Power BI dashboard           ── advisor-facing, filterable, KPI-driven view
```

---

## 🧹 Data cleaning

Full documentation is in [`docs/cleaning_documentation.pdf`](docs/cleaning_documentation.pdf).
Key decisions:

- **Numerical/ordinal columns** (age, parent income, previous GPA, etc.) → imputed with
  the **column median** (robust to outliers).
- **Categorical columns** (gender, part-time job, course type) → imputed as **"Unknown"**,
  never an assumed mode.
- **Outcome variables** (`final_score`, `final_grade`, `pass_fail`) → **never imputed**.
  Inventing a student's result would corrupt the entire downstream analysis.
- Every numeric column was validated against a realistic range (e.g. attendance 0–100%,
  stress level 0–10) before any imputation.

---

## 🔍 Key findings

### Data leakage audit
`final_grade` and `pass_fail` turned out to be **exact deterministic threshold functions**
of `final_score`:

| Grade | `final_score` range |
|---|---|
| A | 90.0 – 100.0 |
| B | 75.0 – 89.9 |
| C | 60.0 – 74.9 |
| D | 50.0 – 59.9 |
| F | 25.7 – 49.9 |

`pass_fail` splits cleanly at `final_score = 50`. Both were excluded from the feature set,
enforced with a hard code assertion in the notebook.

### What actually predicts performance
| Feature | Correlation with `final_score` |
|---|---|
| `midterm_score` | 0.634 |
| `quiz_avg_score` | 0.378 |
| `lecture_attendance_rate` | 0.311 |
| `assignment_submission_rate` | 0.309 |
| All other numeric features | \|r\| < 0.05 |

### Model comparison (test set, Pass/Fail classification)
| Model | ROC-AUC | PR-AUC | Recall | F1 | MCC |
|---|---|---|---|---|---|
| **Logistic Regression** | **0.944** | **0.613** | **0.881** | **0.448** | **0.461** |
| Random Forest | 0.931 | 0.539 | 0.093 | 0.167 | 0.266 |
| LightGBM (tuned) | 0.938 | 0.598 | 0.859 | 0.442 | 0.451 |

Logistic Regression — the simplest, most interpretable model — beat the tuned gradient
boosting model on every metric. Random Forest was rejected outright due to its very low
recall on the Fail class.

---

## 📊 Dashboard

![StudiX Dashboard](dashboard/dashboard.jpeg)

The Power BI dashboard gives advisors a filterable, real-time view of cohort health:
KPI cards (pass rate, average score, average GPA, average attendance), grade and
attendance breakdowns, and filters by final grade, family support, gender, course type,
and part-time job status.

---

## 🚀 Getting started

### Requirements
```
python >= 3.10
pandas, numpy, scikit-learn
matplotlib, seaborn, scipy
lightgbm, xgboost (optional — falls back gracefully if unavailable)
shap (optional, for explainability cells)
```

### Setup
```bash
git clone https://github.com/<your-username>/studix-early-warning.git
cd studix-early-warning
pip install -r requirements.txt
```

### Run the notebook
```bash
jupyter notebook notebooks/student-performance-early-warning-prediction.ipynb
```
Or open it directly in **Kaggle** / **Google Colab** — the data-loading cell auto-detects
`/kaggle/input/` and falls back to a local path otherwise.

### View the report
The full write-up (methodology, results, ethics, limitations) is in
[`docs/student_performance_project_report.pdf`](docs/student_performance_project_report.pdf).

---

## ⚖️ Ethical considerations

This system is a **decision-support tool**, not an automated verdict on a student's
future. All flagged predictions are meant for human review by an academic advisor.
Model explanations (via SHAP) describe what the model relied on to make a prediction —
they are **not** evidence of a causal driver of student failure.

---

## 🔮 Future work

- Collect week-by-week behavioral data instead of semester aggregates for earlier
  detection.
- Add student/course identifiers to enable longitudinal tracking.
- Integrate live model scoring directly into the Power BI dashboard.
- Validate findings on real (non-synthetic) institutional data.

---

## 👥 Contributors

- [Ahmed Gamal]
- [Mohamed Osama]
- [Moustafa Fouad]
- [Ibrahim Amin]

**Instructor:** [Rawan Mohamed]

---

## 📄 License

[Choose a license — e.g. MIT] — see [`LICENSE`](LICENSE) for details.
