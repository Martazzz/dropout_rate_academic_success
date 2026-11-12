# Phase 4: Random Forest — what we did and why

Analysis notebook: `03_random_forest.ipynb`. Input file: `student_dropout_cleaned.csv` (output of the cleaning phase). Figures are saved in `figures/phase4_RF/`, tables in `results/`.

---

## Goal

Predict `Target` (Dropout / Enrolled / Graduate) and answer: **how early can dropout be predicted, and how much does the prediction improve as more information becomes available?**

## Four scenarios (cumulative)

| Scenario | Variables |
|---|---|
| **1a. At enrolment (no economic)** | `Gender`, `Age_At_Enrollment`, `Marital_Group`, `Displaced`, `Admission_Grade`, `Prev_Qual_Grade`, `Application_Mode_Group`, `Application_Order`, `Course_Name`, `Daytime_Evening`, `Parent_Education` |
| **1b. At enrolment (+ economic)** | 1a + `Debtor`, `Tuition_Up_To_Date`, `Scholarship_Holder` |
| **2. After 1st semester** | 1b + `Enrolled_1st`, `Approved_1st`, `Grade_1st`, `Approval_Rate_1st` |
| **3. After 2nd semester** | 2 + `Enrolled_2nd`, `Approved_2nd`, `Grade_2nd`, `Approval_Rate_2nd` |

Performance variables are left out of the scenarios in which they do not exist yet, so the model cannot "cheat". The economic variables are tested separately (1a vs 1b) because they may be recorded after enrolment and be partly a consequence of dropout.

## Choices made

| Choice | Decision | Why |
|---|---|---|
| Missing values | `Grade_*` and `Approval_Rate_*` filled with 0 | as decided in Phase 0 (0 = no unit passed; 180 students enrolled in 0 units) |
| `Passed_None_*` | not used | redundant with `Enrolled_*` + `Approved_*`; its missing values would be ambiguous |
| Encoding | one-hot inside the pipeline | fitted only on training folds, no leakage |
| Split | 80/20 stratified, `random_state=42`, same for all scenarios | comparable results |
| Cross-validation | stratified 5-fold on the training set | model selection; the test set is used only once |
| Imbalance | `class_weight="balanced"` | Graduate 50%, Dropout 32%, Enrolled 18% |
| Tuning | grid on `min_samples_leaf` (1, 3, 5, 10) and `max_features` (sqrt, 0.3), scored with F1 macro | small grid, avoids overfitting the CV |
| Baselines | majority class; logistic regression (balanced, standardised) | to see whether the forest adds value |
| Main metric | **F1 macro** (+ per-class precision/recall, confusion matrix) | accuracy alone is misleading with unbalanced classes |
| Importance | permutation importance on the test set, **by block** and **by variable** | correlated variables share credit; MDI shown only as a check |

## Results (test set, 885 students)

| Scenario | Majority F1 | Log. reg. F1 | **Random Forest F1** | RF accuracy | RF CV F1 (mean ± std) |
|---|---|---|---|---|---|
| 1a | 0.22 | 0.49 | **0.48** | 0.52 | 0.514 ± 0.019 |
| 1b | 0.22 | 0.53 | **0.54** | 0.57 | 0.559 ± 0.018 |
| 2 | 0.22 | 0.65 | **0.67** | 0.71 | 0.687 ± 0.022 |
| 3 | 0.22 | 0.70 | **0.69** | 0.75 | 0.720 ± 0.016 |

Binary version (Dropout vs rest), Random Forest: ROC-AUC 0.73 / 0.80 / 0.90 / 0.91 and recall of Dropout 63% / 66% / 78% / 80% in scenarios 1a / 1b / 2 / 3.

## Key messages

1. The earlier the prediction, the harder: F1 macro 0.48 at enrolment, 0.69 after the 2nd semester.
2. The 1st semester is the turning point (+0.13 F1 macro); the 2nd semester adds only +0.03.
3. Economic variables add ≈ 0.06 F1 macro at enrolment, mainly `Tuition_Up_To_Date` (caveat: possible consequence of dropout).
4. The Random Forest does not clearly beat logistic regression (differences within the CV noise).
5. `Enrolled` is the hardest class (F1 0.30 → 0.47): it is not a final outcome.
6. Most important variables: age and course at enrolment; `Tuition_Up_To_Date` when available; `Approval_Rate_1st` after the 1st semester. Consistent with Blocks A, B and D.

## Limits

One 20% test set (differences ≈ 0.02-0.03 are noise); performance variables are consequences as much as predictors; one institution; unbalanced classes; ambiguous `Enrolled`; prediction ≠ causation.
**Ethics:** `Gender` and `Age_At_Enrollment` are in the model; an early-warning system must be used to **offer support**, never to exclude students.

## Files

| File | Content |
|---|---|
| `F1_model_comparison.png` | accuracy and F1 macro, baselines vs Random Forest, by scenario |
| `F2_precision_recall_by_class.png` | precision and recall by class and scenario |
| `F3_confusion_matrices.png` | confusion matrices (row-normalised) |
| `F4_block_importance.png` | permutation importance by block of variables |
| `F5_top_variables.png` | top-10 single variables (scenarios 1b, 2, 3) |
| `F6_binary_dropout.png` | Dropout vs rest: ROC-AUC, recall, precision |
| `results/phase4_*.csv` | all metric and importance tables |
