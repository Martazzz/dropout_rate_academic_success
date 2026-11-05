# Block D: First-year performance — what we did and why

Analysis notebook: `02d_block_D.ipynb`. Input file: `student_dropout_cleaned.csv` (output of the cleaning phase).

---

## Goal of the block

The purpose of Block D is to understand **when the differences between students who drop out and students who graduate become visible**, by looking at their performance during the first two semesters.

The block does not try to explain why students drop out. Semester performance can also be a **consequence** of a student leaving the university: for example, someone who is about to drop out may stop taking or passing exams. The analysis therefore describes when warning signs appear, without treating them as proof of causation.

The three possible outcomes are:

- `Dropout`: students who left the university;
- `Enrolled`: students who are still enrolled and have not completed their studies;
- `Graduate`: students who completed their studies.

`Enrolled` is not a final outcome, because these students have not finished yet.

## Data and setup

The notebook loads the cleaned dataset and restores the category order used throughout the project:

```python
data = apply_order(pd.read_csv("student_dropout_cleaned.csv"))
```

The same outcome order and colours are used in all the project notebooks:

| Outcome | Colour |
|---|---|
| Dropout | Orange |
| Enrolled | Grey |
| Graduate | Blue |

The notebook creates a `figures` folder and saves the charts there. It does not create a new cleaned dataset: it uses the file produced in Phase 0.

**Important note about the sample:** 180 students were enrolled in 0 units in each semester. Their approval rate cannot be calculated, so they are excluded from analyses that require it. The notebook prints the number of observations (`n`) used in the statistical comparisons.

---

## How we analyse the data

For each question, the notebook combines a chart, a statistical test and an effect size. This is important because the dataset contains thousands of students: even a small difference can be statistically significant. We therefore use the **effect size** to understand whether the difference or relationship is actually substantial.

| Test | Used for | Effect size | How to read it |
|---|---|---|---|
| Kruskal–Wallis | A numeric variable compared across the three outcomes | Epsilon squared | < 0.01 negligible; 0.01–0.06 small; 0.06–0.14 medium; > 0.14 large |
| Chi-square | Two categorical variables | Cramer's V | < 0.1 negligible; 0.1–0.3 weak; 0.3–0.5 moderate; > 0.5 strong |
| Spearman correlation | Two numeric variables | Rho (ρ) | < 0.1 negligible; 0.1–0.3 weak; 0.3–0.5 moderate; > 0.5 strong |

The helper functions in the notebook also print sample sizes and p-values. A result of `p < 0.001` indicates a statistically significant difference or association, but it does not by itself tell us how large or important that result is.

---

## D1. Approved units in the two semesters

**Question:** How many of the units they enrolled in do students who later drop out pass, compared with students who graduate?

**Variables:** `Approval_Rate_1st` and `Approval_Rate_2nd`, compared with `Target`.

The approval rate is the number of approved units divided by the number of enrolled units. It ranges from 0 to 1 (or 0% to 100%). For each semester, the notebook prints the mean and median by outcome, runs a Kruskal–Wallis test and plots the mean approval rate for the three outcomes.

**Draft result written in the notebook (to be checked by running it):**

- Dropouts pass on average 41% of their enrolled units in the 1st semester and 31% in the 2nd.
- Graduates pass 93% in both semesters.
- Students who are still enrolled pass 71% in the 1st semester and 67% in the 2nd.
- The differences are statistically significant (`p < 0.001`) and large (epsilon squared = 0.46 in the 1st semester and 0.55 in the 2nd).

The gap between dropouts and graduates becomes wider in the second semester: graduates remain stable, while the average approval rate of dropouts decreases.

**Why this matters:** the approval rate in the first year is a strong signal of academic difficulty, and the difference is already visible in the first semester.

---

## D2. Grades by outcome

**Question:** Are the grades different for dropouts, enrolled students and graduates?

**Variables:** `Grade_1st` and `Grade_2nd`, compared with `Target`.

The notebook uses boxplots to compare the grade distributions and runs a Kruskal–Wallis test for each semester. It also prints the percentage of students in each outcome group who have a grade and can therefore enter the comparison.

**Important detail about the grades:** `Grade_1st` and `Grade_2nd` are the average grades of the units a student passed. A student who passed no units has no grade (`NaN`), not a grade of zero. Therefore, the comparison does not include all students, and the groups are not equally represented.

**Draft result written in the notebook (to be checked by running it):**

- Graduates have a median grade of 13.0 in both semesters.
- Dropouts have a median grade of 12.0; students still enrolled have a median between 12.0 and 12.1.
- The differences are statistically significant (`p < 0.001`), but their effect is medium (epsilon squared = 0.13 in the 1st semester and 0.14 in the 2nd), much smaller than the differences in approval rates in D1.
- Only 60% of dropouts have a 1st-semester grade and 49% have a 2nd-semester grade, compared with 97% of graduates.

**Interpretation:** among students who passed at least one unit, graduates have slightly higher grades. However, grades alone do not describe all dropouts: many of them have no grade because they passed no units.

---

## D3. Can dropouts be spotted after the 1st semester?

**Question:** Can students who will eventually drop out already be identified from their first-semester performance?

**Variables:** `Approval_Rate_1st`, grouped into four bands, and `Passed_None_1st`, compared with `Dropout_Binary`.

The notebook creates a temporary variable called `Approval_Band_1st`. The four bands are:

| Band | Meaning |
|---|---|
| `None passed` | The student passed no enrolled unit |
| `Under 50%` | The student passed less than half of the enrolled units |
| `50-99%` | The student passed at least half, but not all, of the enrolled units |
| `All passed` | The student passed all enrolled units |

These bands are created inside the notebook for this analysis; they are not added to the cleaned CSV. The notebook calculates the dropout rate in each band and uses chi-square tests with Cramer's V to measure the association. It also checks what proportion of all dropouts would be identified by two simple warning rules: passing no units, or passing less than half of the enrolled units.

**Draft result written in the notebook (to be checked by running it):**

| First-semester approval band | Dropout rate |
|---|---:|
| None passed (n = 538) | 92% |
| Under 50% (n = 310) | 73% |
| 50–99% | 28% |
| All passed (n = 1728) | 9% |

The overall dropout rate in the dataset is about 32%. The association is strong and statistically significant (`p < 0.001`, Cramer's V = 0.61).

However, the first semester does not identify every future dropout: 35% of all dropouts passed no unit, while 51% passed less than half of their enrolled units. In other words, about half of the students who eventually drop out do not show this specific warning sign after the first semester.

**Interpretation:** first-semester performance is a strong early warning signal, but it is not sufficient on its own to identify all students at risk.

---

## D4. Do students who struggle in the 1st semester recover?

**Question:** Do students with weak first-semester performance improve in the second semester?

This question is investigated in two ways.

### Change in grades

**Variable:** `Grade_Change`, calculated as `Grade_2nd` minus `Grade_1st`, compared with `Target`.

A positive value means that the grade improved in the second semester; a negative value means that it decreased. The notebook plots the mean change by outcome, with a 95% confidence interval, and runs a Kruskal–Wallis test.

**Limitation:** `Grade_Change` exists only for students who have a grade in both semesters. Students who passed no units in one of the semesters do not have a grade for that semester, so this analysis mainly describes students who were already managing to pass units.

**Draft result written in the notebook (to be checked by running it):**

- Mean grade change for dropouts: −0.08 points.
- Mean grade change for students still enrolled: −0.07 points.
- Mean grade change for graduates: +0.04 points.
- The differences are statistically significant (`p < 0.001`), but the effect is negligible (epsilon squared = 0.005). The median change is 0 in all three groups.

Grades therefore remain almost unchanged between the two semesters, regardless of the final outcome.

### Recovery among students with a weak 1st semester

To look directly at students who struggled, the notebook selects those who passed less than 50% of their enrolled units in the first semester and checks whether they passed at least 50% in the second semester. Students with no second-semester approval rate are excluded from this recovery calculation.

It also compares the final outcomes of students who passed less than 50% in the first semester with those who passed at least 50%, using a chi-square test.

**Draft result written in the notebook (to be checked by running it):**

- Of the 848 students who passed less than half of their units in the 1st semester, only 86 (10%) passed at least half in the 2nd semester.
- Among the students who struggled in the first semester, 85% dropped out. The dropout rate among students who passed at least half was 18%.
- Recovery is reported for 40% of students still enrolled and 4% of dropouts.
- Only 8 graduates are in the group that struggled in the first semester, which is too small a group for firm conclusions.
- The association between first-semester performance band and outcome is strong (`p < 0.001`, Cramer's V = 0.59).

**Interpretation:** recovery is uncommon among students who pass less than half of their units in the first semester. The result reinforces the importance of early academic performance, while the very small number of graduates in the struggling group limits what we can conclude about them.

---

## D5. Does the admission grade predict the 1st-semester grade?

**Question:** Is a student's admission grade related to their academic grade in the first semester?

**Variables:** `Admission_Grade` (scale 0–200) and `Grade_1st` (scale 0–20). The grade comparison includes only students who have a first-semester grade. The notebook also checks the relationship between `Admission_Grade` and `Approval_Rate_1st`.

The analysis uses a scatter plot and Spearman correlation. Spearman is used because the semester grade is not normally distributed. The notebook also prints the median first-semester grade for each admission-grade quartile.

**Draft result written in the notebook (to be checked by running it):**

- Admission grade and first-semester grade have a weak positive association (Spearman rho = 0.29, `p < 0.001`, n = 3706).
- The median first-semester grade increases from 12.1 in the lowest admission-grade quartile to 13.3 in the highest.
- The relationship between admission grade and first-semester approval rate is even weaker (rho = 0.17).

**Interpretation:** students with higher admission grades tend to obtain slightly higher first-semester grades, but the relationship is weak and the scatter plot shows considerable variation. Admission grade alone is therefore a weak indicator of university performance.

---

## Block D: overall conclusions

| Section | Main result | Effect size / strength |
|---|---|---|
| D1 — Approved units | Dropouts pass 41% of their units in the 1st semester and 31% in the 2nd; graduates pass 93% in both | Large (epsilon squared = 0.46 and 0.55) |
| D2 — Grades | Graduates have slightly higher grades (median 13.0 vs 12.0), but only students with a grade are compared | Medium (epsilon squared = 0.13 and 0.14) |
| D3 — Early warning | Dropout rate is 92% among students who passed no unit and 9% among those who passed all units | Strong (Cramer's V = 0.61) |
| D4 — Recovery | Only 10% of students who passed less than half in the 1st semester pass at least half in the 2nd | Negligible for grade change (epsilon squared = 0.005); strong association for first-semester performance and outcome (Cramer's V = 0.59) |
| D5 — Admission grade | Admission grade has a weak positive relationship with first-semester grade | Weak (Spearman rho = 0.29) |

### Main message

The clearest difference between students who drop out and those who graduate is the **share of enrolled units they pass**, not the grades they obtain in the units they manage to pass. The first semester already provides a strong warning signal, and students who struggle at that stage rarely recover in the second semester. However, around half of the dropouts do not show the simple warning signs based on first-semester approval rate, so other variables are needed for prediction.

### What to show in the presentation

The notebook suggests using these two charts for the slides:

1. The grouped bar chart of D1, showing the difference in approval rate between dropouts and graduates across the two semesters.
2. The D3 chart showing the dropout rate for each first-semester approval band.

The remaining analyses can be kept in the appendix.

### How this connects to Phase 4

The first-semester approval rate is a strong predictive signal, but it does not identify every dropout. Phase 4 (Random Forest) can compare prediction at three stages: at enrollment, after the first semester and after the second semester. This makes it possible to see how prediction changes as more academic information becomes available.

### Limitations to mention

- Academic performance may partly be a consequence of dropping out, not only a cause.
- Grades are available only for students who passed at least one unit in the relevant semester.
- The 180 students enrolled in 0 units in a semester have no approval rate and are excluded from analyses that use it.
- Students still marked as `Enrolled` have not completed their studies; their outcome may change later.
- The data come from a single Portuguese institution, so the results should not automatically be generalised to every university.
- The analyses show associations, not causal effects.

---

## Figures created by the notebook

When the notebook is run, it saves the following charts in the `figures` folder:

| File | Content |
|---|---|
| `blockD_approval_rate_by_outcome.png` | Mean approval rate by outcome and semester (D1) |
| `blockD_grades_by_outcome.png` | Grade distributions by outcome for the two semesters (D2) |
| `blockD_dropout_by_first_semester_approval.png` | Dropout rate by first-semester approval band (D3) |
| `blockD_grade_change_by_outcome.png` | Mean grade change between semesters by outcome (D4) |
| `blockD_struggling_first_semester.png` | Outcome composition for students below and above the 50% first-semester threshold (D4) |
| `blockD_admission_vs_grade.png` | Admission grade against first-semester grade (D5) |

---

## Before using the numbers in a presentation

The numerical results in this document reproduce the **draft conclusions written in `02d_block_D.ipynb`**. The notebook does not contain saved execution outputs, and the cleaned CSV was not included with the two supplied files. Run the notebook with the correct `student_dropout_cleaned.csv` and check the printed tables, sample sizes, p-values and effect sizes before treating these figures as final.
