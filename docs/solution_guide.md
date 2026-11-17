# Solution Guide: Pulse of Prevention

How we solved the case, step by step. All numbers come from the cleaned data (**302 unique patients**) in `notebooks/01_heart_health_eda.ipynb`. "Disease" means the corrected `disease` column (see Step 2).

---

## Part 1: Approach in one page

1. **Look at the data.** 1,025 rows, 14 numeric columns, no blank cells.
2. **Check it before trusting it.** This found three problems: duplicates, invalid codes, and a `target` column that behaves backwards.
3. **Clean.** Remove duplicates, fix invalid codes, correct the target, flag outliers, normalise for comparisons.
4. **Answer the 25 questions.** Start with simple counts, move to group comparisons with tests, then correlations and a model.
5. **Turn it into profiles and actions.** A red-flag score, recommendations, and open research questions.

---

## Part 2: Data cleaning

### Step 1. Missing values
- No blank cells.
- Two columns had impossible codes: `ca = 4` and `thal = 0`. These are placeholders for "missing" in the original heart data.
- **Action:** set them to missing and fill with the most common value. After de-duplication that is only 6 values (4 in `ca`, 2 in `thal`), so the choice does not move results.

### Step 2. Duplicates and the target check
- **723 of 1,025 rows (71%) were exact copies**, leaving 302 unique patients. Keeping them would overweight a few patients and overstate how certain every result is. A random-forest test showed the effect: 98% accuracy on the raw file vs 84% on clean data, simply because the same patients landed in both train and test.
- **Target direction.** If `target = 1` meant disease we would expect those patients to be older, more often male, with more angina, more ST depression, more blocked vessels and a lower peak heart rate. All six went the other way (for example, angina in 14% of `target = 1` patients vs 55% of `target = 0`). So `target = 1` behaves like *no disease*.
- **Action:** new column `disease = 1 - target`. Switch `FLIP_TARGET` in the notebook to undo this. Please confirm with the data owner.
- After the fix: **138 of 302 patients (45.7%) have disease.**

### Step 3. Data types
All columns were already numeric. We split them into measurements (`age`, `trestbps`, `chol`, `thalach`, `oldpeak`) and coded categories (the rest). Coded columns stay numeric so correlations and the model work; label columns (`status`, `sex_label`) were added for charts.

### Step 4. Outliers (IQR rule)
| Column | Outliers | Example |
|---|---|---|
| trestbps | 9 | up to 200 mm Hg |
| chol | 5 | up to 564 mg/dl |
| thalach | 1 | 71 bpm |
| oldpeak | 5 | up to 6.2 |
| age | 0 | |

These are medically possible, not typos. **Action:** keep them for descriptive answers; use a capped copy (20 values pulled to the fence) for normalization and modeling.

### Step 5. Normalization
Z-scores (mean 0, standard deviation 1) put all measurements on one scale. Used for the side-by-side comparison in Medium Q10. The model scales inside its own pipeline, using training data only.

---

## Part 3: Basic questions

| # | Question | Answer | Why it matters |
|---|---|---|---|
| 1 | Average age | **54.4 years** (median 56; range 29-77). Patients with disease average 56.6 vs 52.6 without. | Prevention programs can target the mid-50s onward. |
| 2 | Gender split | **206 men (68.2%), 96 women (31.8%).** Disease rate: men 55.3%, women 25.0% (p < 0.001). | Men are both the majority and higher risk here. |
| 3 | Average resting BP | **131.6 mm Hg** (median 130). 97 patients (32%) are at or above 140. | A third of patients already sit at a commonly used "high" level. |
| 4 | Fasting sugar > 120 | **45 patients (14.9%).** | A smaller group, but worth diet and lifestyle advice. |
| 5 | Chest pain types | Codes **0, 1, 2, 3** with 143 / 50 / 86 / 23 patients. Disease rate: 72.7% / 18.0% / 20.9% / 30.4%. | Code 0 is the most common and has the highest disease rate. |
| 6 | Maximum heart rate | **202 bpm** (a 29-year-old without disease). Lowest 71, average 149.6. | Sets the range for safe exercise targets. |
| 7 | Exercise-induced angina | **32.8%** (99 patients). | One in three has symptoms during exertion. |
| 8 | Average cholesterol | **246.5 mg/dl** (median 240). 155 patients (51%) are at or above 240. | Half are at a commonly used "high" level. |
| 9 | Resting ECG = 2 | **4 patients.** | Too few to draw conclusions about this group. |
| 10 | Major vessels (`ca`) | 0: **179 (59.3%)**, 1: 65 (21.5%), 2: 38 (12.6%), 3: 20 (6.6%). | Most have none flagged; a minority carry the severe cases. |

*Method:* each is a one-line pandas calculation (`mean`, `value_counts`, `sum`, `max`), shown in the notebook. The "high" thresholds (140 mm Hg, 240 mg/dl) are common rules of thumb, not part of the dataset.

---

## Part 4: Medium questions

**M1. Age vs cholesterol.** Weak positive link: Pearson r = 0.21 (p < 0.001), Spearman 0.19. Older patients have slightly higher cholesterol, but age explains very little of it. *Action: check cholesterol at every age, not only in older patients.*

**M2. Chest pain type by age group.** The share of chest pain code 0 rises from 27% in the under-40s to 56% in the 60+. But the pattern is not statistically significant (chi-square p = 0.073) and the youngest group has only 15 patients. *Treat as a hint.* Age groups were used instead of single years because single years have too few patients per bar.

**M3. Max heart rate vs exercise angina.** Patients with angina reach **137.2 bpm** on average vs **155.6 bpm** without, an **18.4 bpm gap** (p < 0.001). *Action: patients with angina need supervised, gradually progressed exercise.*

**M4. Blood pressure by sex.** Women 133.1 vs men 130.9 mm Hg. **Not significant** (Welch t-test p = 0.344; Mann-Whitney p = 0.346). *No basis for sex-specific BP advice from this data.*

**M5. Fasting sugar vs disease.** Disease rate is 45.1% without high sugar and 48.9% with it, **no meaningful difference** (p = 0.761). In this data fasting sugar does not separate patients, although it still matters for health in general.

**M6. Major vessels vs disease.** A steep, clear pattern: disease rate **25.7% (0 vessels) → 67.7% → 81.6% → 85.0% (3 vessels)**, p < 0.001. *Action: prioritise patients with one or more vessels flagged.*

**M7. Average `oldpeak` by chest pain type.** Code 0: 1.38, code 1: 0.32, code 2: 0.81, code 3: 1.39 (differences significant, p < 0.001). Codes 0 and 3 show the most ST depression.

**M8. Thalassemia (`thal`).** Disease rate by code: **1: 66.7%** (n = 18), **2: 22.2%** (n = 167), **3: 76.1%** (n = 117). Code 3 accounts for **64.5% of all disease cases**. *Caution:* the brief's labels do not match how the codes behave (code 2 acts like "normal"), so we report codes only. Confirm labels with the data owner.

**M9. Common risk-factor combinations (among patients with disease).**

| Chest pain | Fasting sugar | Angina | thal | Patients | With disease | Rate | Share of all disease cases |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 1 | 3 | 43 | 40 | 93% | 29.0% |
| 0 | 0 | 0 | 3 | 22 | 19 | 86% | 13.8% |
| 0 | 0 | 1 | 2 | 20 | 14 | 70% | 10.1% |

One profile (chest pain code 0, no high sugar, exercise angina, thal 3) covers **29% of all disease cases** and 93% of the people with that profile have disease. We also show the rate inside each combination, because a count alone can just reflect a common combination.

**M10. Disease vs no disease.**

| Measurement | With disease | Without | Effect size (Cohen d) |
|---|---|---|---|
| `oldpeak` | 1.6 | 0.6 | **0.95** |
| `thalach` (max HR) | 139.1 | 158.4 | **-0.93** |
| `age` | 56.6 | 52.6 | 0.45 |
| `trestbps` | 134.4 | 129.2 | 0.30 |
| `chol` | 251.1 | 242.6 | 0.16 |

`oldpeak` and max heart rate separate the groups strongly. Age is moderate; blood pressure and cholesterol are small. (Cohen d: about 0.2 small, 0.5 medium, 0.8 large.) Figures: normalised bar chart and pair plot.

---

## Part 5: Advanced questions

**A1. Age, cholesterol and blood pressure together.** The pair plot shows the two groups overlap heavily on these three measures. Counting how many of three "elevated" flags a patient has (age ≥ 55, cholesterol ≥ 240, BP ≥ 140) gives disease rates of **26% (0 flags), 40% (1), 59% (2), 63% (3)**. So stacking factors does raise risk, but these three alone are a blunt tool. Age is the strongest single flag (59% vs 31%).

**A2. Strongest correlations with disease.**

| Variable | r | | Variable | r |
|---|---|---|---|---|
| `ca` | +0.46 | | `slope` | -0.34 |
| `exang` | +0.44 | | `sex` | +0.28 |
| `oldpeak` | +0.43 | | `age` | +0.22 |
| `cp` | -0.43 | | `trestbps` | +0.15 |
| `thalach` | -0.42 | | `chol` | +0.08 |
| `thal` | +0.36 | | `fbs` | +0.03 |

Strongest overall: `ca`. Strongest of the five clinical *measurements*: `oldpeak` (0.43), with max heart rate right behind (-0.42). Correlations for coded columns (`cp`, `slope`, `ca`, `thal`) are rough guides only.

**A3. Logistic regression.**
- Setup: capped data (cap limits worked out on all patients, a small shortcut); scaling and one-hot encoding inside a pipeline; 75/25 stratified split; plus 5x5 repeated cross-validation, because a 76-patient test set is small.
- **Test set:** accuracy 0.83, precision 0.84, recall 0.77, F1 0.81, ROC-AUC 0.90. Always guessing the majority class would give 0.54.
- **Cross-validation:** accuracy 0.84 (± 0.04), ROC-AUC 0.91 (± 0.02).
- **Factors that raise risk (odds ratios):** `ca` 3.7, male 3.0, `thal` code 3 3.0, exercise angina 2.7, ST slope code 1 1.9, `oldpeak` 1.7 (per standard deviation for numeric ones).
- **Factors that lower risk:** chest pain codes 3 and 2 (0.22, 0.32 vs code 0), higher max heart rate (0.73), `thal` code 2 (0.67).
- The 77% recall means about 1 in 4 patients with disease is missed in the test set. For screening, you would want to trade some precision for higher recall.
- *Warning:* with a small dataset, odds ratios are rough and some are correlated with each other.

**A4. ST slope by chest pain type.** The two are linked (chi-square p < 0.001) but modestly (Cramér's V = 0.21). Example: slope code 2 is 72% of chest pain code 1 patients but 34% of code 0.

**A5. "Survival rates over time", not possible as written.** The data is one snapshot with no follow-up or survival outcome, and a line plot of `thal` against `age` would just average category codes. Instead we show **disease prevalence by age band for each thal code**:

| thal | under 50 | 50-59 | 60+ |
|---|---|---|---|
| 1 (n = 3 / 10 / 5) | 67% | 70% | 60% |
| 2 (n = 61 / 59 / 47) | 10% | 22% | 38% |
| 3 (n = 23 / 56 / 38) | 78% | 71% | 82% |

In thal code 2, disease rises clearly with age; in code 3 it is high at every age. Code 1 is too small to read. Proper survival analysis needs follow-up data.

---

## Part 6: High-risk patient profile

We built an **8-point red-flag score**, one point each for: male, chest pain code 0, exercise angina, `oldpeak` ≥ 1, one or more vessels, `thal` not 2, max HR < 140, age ≥ 55.

| Band | Score | Patients | With disease | Disease rate |
|---|---|---|---|---|
| Low | 0-2 | 112 | 6 | **5%** |
| Medium | 3-4 | 81 | 33 | **41%** |
| High | 5+ | 109 | 99 | **91%** |

**Typical high-risk patient (5+ flags):** around 58 years old, 84% male, max heart rate about 133 bpm, ST depression (`oldpeak`) around 1.8, 68% have exercise angina, on average 1.3 vessels flagged.
**Typical low-risk patient (0-2 flags):** around 50 years old, 48% male, max heart rate about 164 bpm, `oldpeak` 0.4, 5% have angina, 0.2 vessels.

Cholesterol and BP barely differ between bands (cholesterol 244 vs 252, BP 128 vs 135).

*Caution:* the flags were picked using this same data, so the score looks better here (AUC 0.91) than it would on new patients. It is a simple tool to explain the findings, not a validated clinical score.

---

## Part 7: Recommendations

1. **Screen using symptoms *and* test results.** Chest pain code 0 had the highest disease rate (73%), so patients without classic symptoms should not be waved through. Use `oldpeak`, exercise angina, max heart rate and vessel findings in triage.
2. **Sort follow-up by red-flag count.** More flags, faster follow-up.
3. **Do not rely on cholesterol, blood pressure or fasting sugar alone.** They separate patients weakly here. Keep them as routine checks and as the basis for diet and lifestyle advice.
4. **Exercise programs:** supervised and gradual for patients with exercise angina (they reach about 18 bpm lower peak heart rate).
5. **Outreach for men and people in their mid-50s and older**, who make up more of the disease group.

Check all of these with clinicians before acting. This is practice analysis, not medical advice.

## Part 8: Further research

- Why are disease rates so different by sex (55% vs 25%)? Sampling, or a real effect?
- Why does the "typical" chest pain code 0 go with the highest disease rate? Confirm what the codes mean.
- Confirm the direction of `target` and the `thal` labels with the research institute.
- Get a larger, duplicate-free dataset with patient follow-up so survival analysis is possible.
- Add lifestyle data the file lacks (smoking, weight, activity, family history).

## Part 9: Limitations

- Only 302 unique patients from one source; results are indicative.
- A snapshot shows association, not cause.
- Several category meanings are undefined or doubtful.
- The red-flag score is tested on the data it was built from.
- Small groups (`restecg` = 2, `thal` = 1, under-40s) give unstable percentages.

## Part 10: Notes on the brief

- The brief's "Desired Outcome" mentions *loan analysis*, apparently left over from another case; it does not apply here.
- The resume snippet (20% better early diagnosis, 95% data accuracy, 10% fewer cases) is not backed by anything in this analysis. If you use a resume line, base it on results you can show: removed 723 duplicates, found and fixed a reversed target, logistic regression with about 83% test accuracy (ROC-AUC 0.90), a risk score that separates 5% from 91% disease rate.
