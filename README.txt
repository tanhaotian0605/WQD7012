MMSE Regression Handoff Package
=============================

Goal
----
Predict MMSE (Mini-Mental State Examination) as a regression task.

Files
-----
- train.csv: training split (features + MMSE)
- test.csv: test split (features + MMSE)
- train_index.csv / test_index.csv: row indices used for the split (strict reproducibility)
- preprocess.joblib: sklearn ColumnTransformer preprocessing object

Preprocessing rules
-------------------
- Dropped columns: PatientID, DoctorInCharge
- Excluded from features by default: Diagnosis (potential leakage; use only for ablation if needed)
- Categorical (One-Hot): Ethnicity, EducationLevel (handle_unknown='ignore')
- Numeric + Binary: median imputation + StandardScaler
- Categorical: most-frequent imputation + OneHotEncoder

Split strategy
--------------
- train/test = 80/20
- random_state = 42
- stratify: MMSE binned into 10 quantiles (pd.qcut)

Usage
-----
Load train.csv/test.csv, load preprocess.joblib, build:
Pipeline([('preprocess', preprocess), ('regressor', <your regressor>)])

Fit on train, evaluate on test.
