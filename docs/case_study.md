# Case Study: Pulse of Prevention
### Analyzing Heart Health for Better Outcomes

## 1. Background

You are a data analyst at **HealthPulse Analytics**, a healthcare company that uses data to improve patient outcomes. A cardiology research institute has shared a dataset on patients with and without heart disease. Management wants to know what drives heart disease and who is most at risk, so that care providers can act earlier.

## 2. Problem statement

Find the key factors linked to heart disease and build a profile of high-risk patients, using patient demographics, medical history and clinical measurements.

**Business goal:** reduce the incidence of heart disease by spotting high-risk patients early and recommending preventive steps.

## 3. Stakeholders

| Internal | External |
|---|---|
| Management team | Patients |
| Healthcare providers | Cardiology research institute |
| Data analysts | Healthcare policymakers |

## 4. What the analysis needs to deliver

- **Metrics:** descriptive statistics, correlations, and key risk factors found through EDA.
- **Insights and actions:** profiles of high-risk patients, preventive and lifestyle recommendations, and areas for further medical research.
- **Communication:** a clear summary backed by visuals.

## 5. Data

- 1,025 rows and 14 columns in the file (302 unique patients after removing duplicates).
- Source: Kaggle heart disease dataset.

### Data dictionary

| Column | Meaning | Type | Notes from checking the data |
|---|---|---|---|
| `age` | Age in years | Measurement | Range 29-77 |
| `sex` | 1 = male, 0 = female | Category | |
| `cp` | Chest pain type (0-3) | Category | Brief does not name the types |
| `trestbps` | Resting blood pressure (mm Hg) | Measurement | |
| `chol` | Serum cholesterol (mg/dl) | Measurement | |
| `fbs` | Fasting blood sugar > 120 mg/dl (1 = yes) | Category | |
| `restecg` | Resting ECG result (0-2) | Category | Only 4 patients have code 2 |
| `thalach` | Maximum heart rate achieved | Measurement | |
| `exang` | Exercise-induced angina (1 = yes) | Category | |
| `oldpeak` | ST depression from exercise vs rest | Measurement | |
| `slope` | Slope of peak exercise ST segment (0-2) | Category | |
| `ca` | Major vessels colored by fluoroscopy (0-3) | Category | File also has 4, an invalid value |
| `thal` | Thalassemia (brief: 1 normal, 2 fixed defect, 3 reversible defect) | Category | File also has 0; the labels look shifted (see below) |
| `target` | Heart disease (brief: 1 = yes) | Category | Appears reversed (see below) |

### Data issues found

| Issue | What we saw | What we did |
|---|---|---|
| Duplicates | 723 of 1,025 rows repeated exactly | Removed (302 unique patients) |
| Invalid codes | `ca = 4` (4 rows), `thal = 0` (2 rows) after de-duplication | Treated as missing, filled with most common value |
| Target direction | `target = 1` patients had less angina, fewer blocked vessels, higher max heart rate | Created `disease` column = reverse of `target` |
| `thal` labels | Code 2 is most common and lowest disease rate | Used codes, not names |
| Outliers | A few extreme BP, cholesterol, oldpeak values | Kept for descriptive questions, capped for modeling |

## 6. Cleaning steps required

1. Handle missing values (check blanks and invalid codes).
2. Make sure data types are right.
3. Detect and handle outliers.
4. Normalize clinical measurements.

## 7. Questions

### Basic
1. What is the average age of patients?
2. What is the gender distribution?
3. What is the average resting blood pressure?
4. How many patients have fasting blood sugar above 120 mg/dl?
5. What types of chest pain are recorded?
6. What is the maximum heart rate achieved?
7. What percentage have exercise-induced angina?
8. What is the average cholesterol level?
9. How many patients have resting ECG result 2?
10. What is the distribution of major vessels colored by fluoroscopy?

### Medium
1. What is the correlation between age and cholesterol?
2. How is chest pain type distributed across age groups?
3. How does max heart rate vary with exercise-induced angina?
4. Is resting blood pressure different between men and women?
5. How are fasting blood sugar and heart disease related?
6. How does the number of major vessels relate to heart disease?
7. What is the average `oldpeak` for each chest pain type?
8. How is thalassemia type distributed among patients with heart disease?
9. What are the most common risk-factor combinations in patients with heart disease?
10. How do clinical measurements compare for patients with and without heart disease?

### Advanced
1. What is the combined effect of age, cholesterol and blood pressure on heart disease?
2. Which variable has the strongest correlation with heart disease?
3. Can logistic regression predict heart disease from all features?
4. How does the ST slope vary with chest pain type?
5. Survival rates by thalassemia type over time (see note below).

> **Note on question 5.** The data has no follow-up time or survival outcome, so survival cannot be analysed. The solution shows disease prevalence by age band for each thal code instead.

## 8. Deliverables

- Jupyter notebook with the full analysis (`notebooks/01_heart_health_eda.ipynb`)
- This case study
- Solution guide (`docs/solution_guide.md`)
- README

## 9. What you should learn

- How to check a dataset before trusting it (duplicates, invalid codes, label direction).
- How to compare groups with charts and simple tests.
- How to turn findings into plain-language actions.
- Where a dataset cannot answer a question, and how to say so.

## 10. Other things to keep in mind

- **Privacy and ethics:** patient data needs protection; models should be reviewed by clinicians before use.
- **Teamwork:** check findings with clinical experts.
- **Keep improving:** refresh the analysis when new data arrives.

## 11. Further reading

- [Kaggle: Heart disease dataset](https://www.kaggle.com/datasets/johnsmith88/heart-disease-dataset)
- [UCI Machine Learning Repository: Heart Disease](https://archive.ics.uci.edu/dataset/45/heart+disease) (the original dataset this is derived from)
- [pandas documentation](https://pandas.pydata.org/docs/), [seaborn gallery](https://seaborn.pydata.org/examples/index.html), [scikit-learn: logistic regression](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression)
