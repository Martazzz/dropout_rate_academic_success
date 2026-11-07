# Block C: Academic background and enrollment — what we did and why

Analysis scripts: `C1.py`, `C2.py`, `C3.py`, `C4.py` (one script per question). Input file: `student_dropout_cleaned.csv` (output of the cleaning phase).

---

## Goal of the block

The purpose of Block C is to understand whether **the academic background a student brings to university and the way they entered it** are associated with the final outcome. The block looks at four aspects: the admission grade, the grade of the previous qualification, the order in which the course was chosen in the application, and the admission route (application mode).

The block does not try to explain why students drop out. These variables describe the situation at enrollment, so they are not a consequence of dropping out, but they are also not isolated factors: they overlap with each other and with student age (Block A). The analysis therefore describes associations and, where two variables overlap, checks whether the same effect is being counted twice.

The three possible outcomes are:

- `Dropout`: students who left the university;
- `Enrolled`: students who are still enrolled and have not completed their studies;
- `Graduate`: students who completed their studies.

`Enrolled` is not a final outcome, because these students have not finished yet.

## Data and setup

Each script loads the cleaned dataset and restores the category order used throughout the project:

```python
df = pd.read_csv("student_dropout_cleaned.csv")
df = apply_order(df)
```

The dataset has 4,424 students. The outcome distribution is:

| Outcome | Students | Share |
|---|---:|---:|
| Graduate | 2,209 | 49.9% |
| Dropout | 1,421 | 32.1% |
| Enrolled | 794 | 18.0% |

The same outcome order and colours are defined at the top of every script:

| Outcome | Colour |
|---|---|
| Dropout | Dark red (`#9b4141`) |
| Enrolled | Grey (`#b8b8b8`) |
| Graduate | Teal (`#298c8c`) |

The bar charts of the dropout rate use the dropout colour (C1, C3) or a single blue (`#0072B2`, C4). The scripts create a `figures` folder, but in the version supplied the `plt.savefig` lines are commented out, so the charts are only displayed (see the last section).

**Important note about the sample:** none of the variables used in Block C has missing values, so all four analyses use the full sample of 4,424 students. Unlike Block D, no student is excluded.

---

## How we analyse the data

For each question the scripts combine a chart and a statistical test. Since the dataset contains thousands of students, even a small difference can be statistically significant, so the document also reports an **effect size** to judge whether the difference is substantial.

| Test | Used for | Effect size | How to read it |
|---|---|---|---|
| Kruskal–Wallis | A numeric variable compared across the three outcomes | Epsilon squared | < 0.01 negligible; 0.01–0.06 small; 0.06–0.14 medium; > 0.14 large |
| Chi-square | Two categorical variables | Cramér's V | < 0.1 negligible; 0.1–0.3 weak; 0.3–0.5 moderate; > 0.5 strong |

**What the scripts print and what was added.** The scripts print the test statistic, the degrees of freedom and the p-value. The p-values appear as `0.000000`, which means p < 0.000001. The effect sizes (epsilon squared = (H − k + 1) / (n − k); Cramér's V) are **not** printed by the scripts: they were computed separately on the same data. The same applies to everything labelled **Additional check** below.

A significant Kruskal–Wallis test only says that at least one outcome group differs from the others. For C1 and C2 the additional checks therefore include pairwise Mann–Whitney tests with a Bonferroni correction (3 comparisons).

---

## C1. Admission grade and dropout

**Question:** Is a higher admission grade associated with fewer dropouts?

**Variables:** `Admission_Grade` (scale 0–200) compared with `Target`; the grade is also split into four quartile bands (`Admission_Grade_Band`).

The script computes the quartiles with `pd.qcut`, so each band holds about 1,100 students. It prints descriptive statistics by outcome, draws a notched boxplot of the admission grade by outcome (the notches show an approximate confidence interval of the median), plots the dropout rate by quartile and runs a Kruskal–Wallis test.

**Results (from running the script):**

| Outcome | n | Mean | Median |
|---|---:|---:|---:|
| Dropout | 1,421 | 124.96 | 123.6 |
| Enrolled | 794 | 125.53 | 124.1 |
| Graduate | 2,209 | 128.79 | 127.4 |

| Admission grade band | Grade range | Students | Dropout rate |
|---|---|---:|---:|
| Q1 (lowest) | 95.0–117.9 | 1,111 | 39.5% |
| Q2 | 118.0–126.1 | 1,109 | 31.0% |
| Q3 | 126.2–134.8 | 1,101 | 29.3% |
| Q4 (highest) | 134.9–190.0 | 1,103 | 28.6% |

- Kruskal–Wallis: H = 73.23, `p < 0.001` (statistically significant).
- **Additional check:** epsilon squared = 0.016, a **small** effect. Pairwise tests show that graduates differ from both dropouts and enrolled students (both `p < 0.001`), while dropouts and enrolled students do not differ from each other (p = 0.70). For quartile × dropout (yes/no), Cramér's V = 0.094 (negligible, at the boundary of weak).

**Interpretation:** a higher admission grade is associated with fewer dropouts, but the relationship is not linear. The gap is concentrated in the lowest quartile (39.5%), about 11 percentage points above the highest one (28.6%). Between the second, third and fourth quartiles the dropout rate varies by only about 2.5 points. The effect is small: the admission grade alone is a weak indicator of who will drop out.

---

## C2. Grade of the previous qualification

**Question:** Is the grade of the previous qualification associated with the outcome?

**Variables:** `Prev_Qual_Grade` (scale 0–200) compared with `Target`.

The script prints descriptive statistics by outcome, draws a notched boxplot and runs a Kruskal–Wallis test.

**Results (from running the script):**

| Outcome | n | Mean | Median | Min–max |
|---|---:|---:|---:|---|
| Dropout | 1,421 | 131.11 | 133.1 | 95.0–190.0 |
| Enrolled | 794 | 131.21 | 130.0 | 96.0–190.0 |
| Graduate | 2,209 | 134.08 | 133.1 | 97.0–184.4 |

- Kruskal–Wallis: H = 63.68, `p < 0.001` (statistically significant).
- **Additional check:** epsilon squared = 0.014, a **small** effect. As in C1, graduates differ from dropouts and from enrolled students (both `p < 0.001`), while dropouts and enrolled students do not differ (p = 1.0 after correction).

The medians of dropouts and graduates are identical (133.1), so the difference is not in the median but in the rest of the distribution. **Additional check:** the first quartile is 122.0 for dropouts and 126.0 for graduates, and the third quartile is 137.0 against 141.0.

**Interpretation:** graduates tend to have slightly higher previous-qualification grades, but the effect is small and of the same size as for the admission grade in C1. **Additional check:** the two grades are correlated (Spearman rho = 0.58), so C1 and C2 are not two independent pieces of evidence: they partly describe the same academic background.

---

## C3. Application order

**Question:** Do students admitted to a course of higher preference drop out less?

**Variables:** `Application_Order_Group` (three groups: `0-1`, `2`, `3+`) compared with `Dropout_Binary`.

The script counts students and dropouts per group, plots the dropout rate and runs a chi-square test on the contingency table (group × dropout yes/no). Lower application-order values correspond to a course chosen with higher preference, which is the direction assumed by the research question.

**Results (from running the script):**

| Application order | Students | Dropouts | Dropout rate |
|---|---:|---:|---:|
| 0–1 | 3,027 | 1,053 | 34.8% |
| 2 | 547 | 150 | 27.4% |
| 3+ | 850 | 218 | 25.7% |

- Chi-square: χ² = 31.75, 2 degrees of freedom, `p < 0.001` (statistically significant).
- **Additional check:** Cramér's V = 0.085, a **negligible** association.

The direction is the **opposite of what the question assumed**: students in the highest-preference group (0–1) drop out *more* often than those who got a lower-preference course. Almost all students of the `0-1` group have the raw value 1 (3,026 out of 3,027).

**Additional check (confounding with the admission route and age):** the application-order groups differ strongly in age. The share of students aged 17–19 is 33.6% in the `0-1` group, 57.4% in group `2` and 73.2% in group `3+`; the share aged 30 or more is 22.0%, 6.4% and 1.1%. When the comparison is restricted to the **general contingent** route only (2,704 students), the dropout rates are almost identical: 23.2% (0–1), 23.3% (2) and 25.5% (3+), with χ² = 1.69, p = 0.43 and Cramér's V = 0.025.

**Interpretation:** the apparent association between application order and dropout is small and, once the admission route is held fixed, it disappears. It most likely reflects *who* applies in which order (older students and students entering through other routes are concentrated in the `0-1` group), not a real effect of course preference. This result should be read together with C4 and Block A.

---

## C4. Admission route

**Question:** Is the admission route (general contingent, over 23, transfer, etc.) associated with the outcome?

**Variables:** `Application_Mode_Group` (five groups) compared with `Target`.

The script builds the table of students and the row percentages of the three outcomes by route, plots the dropout rate by route as a sorted horizontal bar chart with percentage labels, and runs a chi-square test on the 5 × 3 contingency table (route × outcome).

**Results (from running the script):**

| Admission route | Students | Dropout | Enrolled | Graduate |
|---|---:|---:|---:|---:|
| General contingent | 2,704 | 23.9% | 17.6% | 58.5% |
| Over 23 years old | 785 | 55.4% | 15.4% | 29.2% |
| Transfer or change | 449 | 37.6% | 22.1% | 40.3% |
| Prior diploma holder | 387 | 39.3% | 18.9% | 41.9% |
| Special or other | 99 | 19.2% | 26.3% | 54.5% |

- Chi-square: χ² = 338.14, 8 degrees of freedom, `p < 0.001` (statistically significant). All expected counts are above 5.
- **Additional check:** Cramér's V = 0.195 (**weak**) on the three outcomes and 0.261 (weak, close to the upper end of the band) for dropout yes/no. The standardized residuals show that the association is driven mainly by the over-23 route, which has many more dropouts than expected (+11.5), and by the general contingent, which has fewer (−7.6).

More than half of the students admitted through the over-23 route drop out (55.4%), against 23.9% in the general contingent, a gap of about 31 percentage points. The `Special or other` group has only 99 students, so its rates are unstable.

**Additional check (overlap with student age, see Block A):** the over-23 route is made almost entirely of mature students (58.5% are aged 30+ and 31.5% aged 25–29; median age 32, against 19 in the general contingent). Restricting the comparison to students aged 23 or older, the dropout rates across routes are much closer: 55.4% (over 23), 55.6% (prior diploma holders), 46.2% (transfer or change) and 45.9% (general contingent, only 133 students). Among students younger than 23 the rates are about 23% for the general contingent and for prior diploma holders and 27% for transfers.

**Interpretation:** the admission route is the clearest signal of the block, and the over-23 route in particular is associated with a much higher dropout rate. However, a large part of this gap is shared with age: mature students drop out more whichever route they used. The effect of the route and the effect of age should not be counted twice when the results of Blocks A and C are combined.

---

## Block C: overall conclusions

| Section | Main result | Effect size / strength |
|---|---|---|
| C1 — Admission grade | Dropout rate is 39.5% in the lowest quartile and 28.6% in the highest; the gap is concentrated in Q1 | Small (epsilon squared = 0.016) |
| C2 — Previous qualification grade | Graduates have slightly higher grades (mean 134.1 against 131.1 for dropouts); the medians coincide | Small (epsilon squared = 0.014) |
| C3 — Application order | Dropout is 34.8% (order 0–1), 27.4% (2) and 25.7% (3+): opposite to the expected direction; disappears within the general contingent | Negligible (Cramér's V = 0.085; 0.025 within the general contingent) |
| C4 — Admission route | Over-23 route: 55.4% dropout, against 23.9% in the general contingent | Weak (Cramér's V = 0.195 on the outcomes; 0.261 on dropout yes/no) |
