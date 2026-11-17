# Pulse of Prevention: Heart Health EDA

An exploratory data analysis (EDA) project for a fictional company, **HealthPulse Analytics**. The job: find which factors go with heart disease, describe what a high-risk patient looks like, and suggest preventive actions.

Practice project. Nothing here is medical advice.

## What's inside

```
heart-health-eda/
├── README.md                  <- you are here
├── requirements.txt           <- Python packages
├── data/
│   └── heart.csv              <- raw data (1,025 rows, 14 columns)
├── notebooks/
│   └── 01_heart_health_eda.ipynb   <- the whole analysis, already run
├── outputs/
│   └── figures/               <- 21 charts saved by the notebook
└── docs/
    ├── case_study.md / .docx  <- the problem, data, and the 25 questions
    └── solution_guide.md / .docx  <- answers, steps and reasoning
```

## How to run

```bash
pip install -r requirements.txt
cd notebooks
jupyter notebook 01_heart_health_eda.ipynb     # then Run All
```

The notebook reads `../data/heart.csv` and writes charts to `../outputs/figures/`, so run it from inside `notebooks/`.

## Notebook outline

| Section | What happens |
|---|---|
| 1. Setup | Imports, colours, two switches (`DROP_DUPLICATES`, `FLIP_TARGET`) |
| 2. Load and first look | Shape, types, summary stats |
| 3. Cleaning | Missing values, data types, duplicates, target check, outliers, normalization |
| 4. Basic questions | 10 quick descriptive questions |
| 5. Medium questions | 10 comparisons, with simple statistical tests |
| 6. Advanced questions | Combined risk factors, correlations, logistic regression, and more |
| 7. High-risk profile | A simple red-flag score and what high-risk patients look like |
| 8. Wrap-up | Key numbers, recommendations, further research, limitations |

## Headline findings

- **302 unique patients**, not 1,025: 723 rows in the file are exact duplicates.
- About **46%** of patients have heart disease.
- Strongest signals: **number of major vessels (`ca`)**, **exercise-induced angina**, **ST depression (`oldpeak`)**, **chest pain code**, and **max heart rate**.
- **Men** have a much higher disease rate than women here (55% vs 25%).
- **Cholesterol, fasting blood sugar and resting blood pressure are weak separators** in this data.
- A logistic regression reaches about **83% test accuracy** (ROC-AUC about 0.90; 5x5 cross-validated accuracy 84%).
- A simple 8-point red-flag score sorts patients well: **5% / 41% / 91%** disease rate in the low / medium / high bands.

## Things to know before using the results

These came up while checking the data and the brief. The notebook explains each one.

1. **Duplicates.** Removed by default. Set `DROP_DUPLICATES = False` to see the raw-file version.
2. **The `target` column looks reversed.** Per the data dictionary `target = 1` means disease, but those patients have *less* angina, *fewer* blocked vessels and *higher* max heart rate, so the label behaves the opposite way (0 of 6 clinical checks agree). The notebook builds a `disease` column that flips it. Set `FLIP_TARGET = False` to use `target` as given. Worth confirming with the data owner.
3. **Invalid codes.** `ca = 4` and `thal = 0` are not in the dictionary. They are treated as missing and filled with the most common value (6 rows after de-duplication).
4. **`thal` labels look shifted.** Code 2 is the most common and has the lowest disease rate, so it behaves like "normal", not 1 as the dictionary says. The notebook uses the codes only.
5. **"Survival rate" question (Advanced Q5) can't be answered.** The data is a one-time snapshot with no follow-up or survival column. The notebook shows disease prevalence by age band instead.
6. **Resume snippet in the brief** (20% better diagnosis, 95% accuracy, 10% fewer cases) is not supported by anything in this analysis. Use only numbers you can show, for example the model accuracy and risk-band results above.

## Data

Source named in the brief: [Kaggle heart disease dataset](https://www.kaggle.com/datasets/johnsmith88/heart-disease-dataset). Column meanings are in `docs/case_study.md`.

## Tools

Python, pandas, NumPy, Matplotlib, Seaborn, SciPy, scikit-learn.
