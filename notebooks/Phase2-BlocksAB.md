# Blocks A and B: demographics and economic situation

Notebook: `02a_blocks_A_B.ipynb`. Data: `student_dropout_cleaned.csv` (4424 students). Charts saved in `figures/` (`blocks_AB_...`).

---

## What we did and why

Blocks A and B follow the EDA. The EDA showed *which* variables seem related to dropout, without any test. Here we check each question with a chart, a statistical test and the **size of the effect**. With thousands of students almost every difference is statistically significant, so the conclusion is based on the size of the effect and on the percentages, not on the p-value alone.

- **Block A (demographics):** gender, age at enrollment, marital status, displaced.
- **Block B (economic situation):** scholarship, debtor, tuition fees up to date, parents' education and a combined financial risk score.

The reference variable is always `Target` (`Dropout`, `Enrolled`, `Graduate`). For each categorical variable the chart is a horizontal stacked bar at 100% (one bar per category, with the number of students `n` next to the label), so groups of different size can be compared.

### Tests used

| Test | When | What it answers | Effect size |
|---|---|---|---|
| **Chi-square test of independence** | two categorical variables | is the outcome associated with the category? | **Cramér's V** (0 to 1) |
| **Kruskal-Wallis test** | a numeric variable compared across the 3 outcomes | do the 3 groups have different values? | **epsilon squared** |
| **Spearman correlation** | two ordered variables | does one tend to grow when the other grows? | **rho** (-1 to +1) |

Kruskal-Wallis is used instead of ANOVA because age is not normally distributed (long tail of older students). Spearman is used instead of Pearson because the financial risk is an ordered count (0-3), not a continuous measure.

| Size of the effect | Cramér's V | epsilon squared | rho |
|---|---|---|---|
| negligible | < 0.1 | < 0.01 | < 0.1 |
| weak | 0.1-0.3 | 0.01-0.06 | 0.1-0.3 |
| moderate | 0.3-0.5 | 0.06-0.14 | 0.3-0.5 |
| strong | > 0.5 | > 0.14 | > 0.5 |

---

## Results at a glance

| No. | Question | Test | Result | Effect size |
|---|---|---|---|---|
| A1 | Gender and outcome | chi-square: chi2 = 233.3, dof = 2, p < 0.001 | men drop out 45%, women 25% | V = 0.23 (weak) |
| A2 | Age at enrollment and outcome | chi-square on age groups: chi2 = 452.0, dof = 6, p < 0.001; Kruskal-Wallis on age: H = 375.1, p < 0.001 | dropout 21% (17-19), 29% (20-24), 58% (25-29), 54% (30+) | V = 0.23 (weak); epsilon squared = 0.08 (moderate) |
| A3 | Marital status and outcome | chi-square: chi2 = 57.5, dof = 2, p < 0.001 | not single 47%, single 30% | V = 0.11 (weak) |
| A4 | Displaced and outcome | chi-square: chi2 = 57.8, dof = 2, p < 0.001 | displaced 28%, not displaced 38% | V = 0.11 (weak) |
| B1 | Scholarship and outcome | chi-square: chi2 = 409.9, dof = 2, p < 0.001 | scholarship holders 12%, others 39% | V = 0.30 (weak to moderate) |
| B2 | Debtors and outcome | chi-square: chi2 = 259.3, dof = 2, p < 0.001 | debtors 62%, others 28% | V = 0.24 (weak) |
| B3 | Tuition fees and outcome | chi-square: chi2 = 823.6, dof = 2, p < 0.001 | fees not up to date 87%, up to date 25% | V = 0.43 (moderate) |
| B4 | Parents' education and outcome | chi-square: chi2 = 104.7, dof = 6, p < 0.001 (all groups); chi2 = 20.9, dof = 4, p < 0.001 (without Unknown) | 32%, 30%, 31% for Basic, Secondary, Higher | V = 0.11 (weak)|
| B5 | Financial risk and outcome | chi-square: chi2 = 1038.1, dof = 6, p < 0.001; Spearman: rho = 0.42, p < 0.001 | dropout 9%, 29%, 69%, 89% for 0, 1, 2, 3 factors | V = 0.34 (moderate); rho = 0.42 (moderate) |

---

## Block A: demographics

### A1. Gender and outcome
**What we did:** stacked bar chart of the outcome by gender and a chi-square test with Cramér's V.
**Result:** 45% of men (n = 1556) dropped out, against 25% of women (n = 2868), a difference of 20 percentage points. Only 35% of men graduated, against 58% of women. The share still enrolled is similar (20% and 17%).
**Conclusion:** men drop out much more often than women. The association is statistically significant (p < 0.001) but weak in size (V = 0.23). Gender may be related to other variables, such as the course, so this is not a causal effect.

### A2. Age at enrollment and outcome
**What we did:** stacked bar chart by age group, boxplot of the age by outcome, chi-square test on the age groups and Kruskal-Wallis test on the age in years.
**Result:** the dropout rate is 21% for students aged 17-19 (n = 1952) and 29% for 20-24 (n = 1334), but it jumps to 58% for 25-29 (n = 427) and 54% for 30+ (n = 711): it more than doubles from age 25. The median age is 23 for dropouts, 20 for students still enrolled and 19 for graduates.
**Conclusion:** students who enroll at 25 or older drop out more than twice as often as those under 20. The association is significant and weak by age group (V = 0.23) and medium on the age in years (epsilon squared = 0.08). Age overlaps with the "over 23" admission route and with evening attendance, which describe partly the same students: the effect should not be counted three times.

### A3. Marital status and outcome
**What we did:** stacked bar chart and chi-square test, using the grouped variable `Single` / `Not single`.
**Result:** 47% of students who are not single dropped out (n = 505), against 30% of single students (n = 3919). Graduates are 38% and 51%.
**Conclusion:** students who are not single drop out more often. The association is significant but weak (V = 0.11). The "not single" group is small and older on average, so part of the difference may be due to age.

### A4. Displaced and outcome
**What we did:** stacked bar chart and chi-square test. We use the term "displaced" as in the dataset, without interpreting it, because the documentation does not define it.
**Result:** displaced students drop out less (28%, n = 2426) than the others (38%, n = 1998). Graduates are 55% and 44%.
**Conclusion:** displaced students have better outcomes. The association is significant but weak (V = 0.11).

---

## Block B: economic situation

### B1. Scholarship and outcome
**What we did:** stacked bar chart and chi-square test with Cramér's V.
**Result:** 12% of scholarship holders dropped out (n = 1099), against 39% of the others (n = 3325). Graduates are 76% and 41%.
**Conclusion:** scholarship holders drop out far less often. The association is significant and between weak and moderate (V = 0.30). A scholarship may depend on merit, so it can also reflect academic performance: it should not be read as the effect of the money alone.

### B2. Debtors and outcome
**What we did:** stacked bar chart and chi-square test.
**Result:** 62% of debtors dropped out (n = 503), against 28% of the others (n = 3921). Only 20% of debtors graduated, against 54%.
**Conclusion:** debtors drop out more than twice as often. The association is significant and weak (V = 0.24).

### B3. Tuition fees and outcome
**What we did:** stacked bar chart and chi-square test.
**Result:** 87% of students whose tuition fees are not up to date dropped out (n = 528), against 25% of those who are in order (n = 3896). Only 5% of the first group graduated, against 56%.
**Conclusion:** it is the strongest association of the two blocks (V = 0.43, moderate). However, someone who leaves the university may stop paying, so the variable is partly a **consequence** of dropout and not only a cause.

### B4. Parents' education and outcome
**What we did:** stacked bar chart by the highest education level of the parents (`Basic`, `Secondary`, `Higher`, `Unknown`) and chi-square test.
**Result:** the dropout rate is almost identical: 32% (Basic, n = 2220), 30% (Secondary, n = 1311), 31% (Higher, n = 793). Students of parents with higher education are more often still enrolled (23%) and less often graduates (45%), but the effect is negligible (V = 0.05).
**Conclusion:** the education of the parents makes almost no difference to the dropout rate. 

### B5. Financial risk and outcome
**What we did:** the variable `Financial_Risk` counts three risk factors (debtor, tuition fees not up to date, no scholarship) from 0 to 3. We drew the dropout rate for each level (bar chart) and ran a chi-square test and a Spearman correlation with `Dropout_Binary`.
**Result:** the dropout rate is 9% for students with no risk factor (n = 992), 29% with one (n = 2731), 69% with two (n = 478) and 89% with all three (n = 223). Graduates fall from 79% to 3%.
**Conclusion:** the dropout risk grows with each additional financial risk factor. The association is significant and moderate (V = 0.34, rho = 0.42). Since tuition fees and debts may partly be a consequence of dropout, the score describes a risk profile, not a cause.

---

## Key findings of blocks A and B

1. **The economic situation is the strongest signal of the two blocks.** Tuition fees not up to date (V = 0.43), the financial risk score (V = 0.34) and the scholarship (V = 0.30) have the largest effects, the dropout rate goes from 9% (no risk factor) to 89% (three factors).
2. **Age matters from 25:** the dropout rate more than doubles (21% under 20, 54-58% from 25).
3. **Men and students who are not single drop out more often**, with weak effects (V = 0.23 and 0.11).
4. **Displaced students drop out less**, a weak effect (V = 0.11) that we describe without interpretation.
5. **Parents' education makes no relevant difference** (30-32% in all the known levels, V = 0.05).
