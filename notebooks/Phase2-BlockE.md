# Block E: Degree course

The purpose of Block E is to understand **how much the dropout rate changes from one degree course to another**, and whether these differences are linked to the performance of the course, to the gender of the students and to the schedule (daytime or evening).

The block does not try to explain why a course has a high or low dropout rate. A course is not a single factor: it also stands for the kind of students it attracts (their gender, their age, their previous grades), so a difference between courses can reflect the course itself, the students who enroll in it, or both. The analysis therefore describes where dropout is higher or lower, without treating the course as the cause.

The reference variable of the block is `Dropout_Binary`:

- `1`: the student dropped out (1,421 students, 32.1% of the sample);
- `0`: the student graduated or is still enrolled (3,003 students).

`Enrolled` is not a final outcome, because these students have not finished yet: for them the value 0 simply means that they have not dropped out *so far*.

## Data and setup

Each script loads the cleaned dataset and restores the category order used throughout the project:

```python
df = pd.read_csv("student_dropout_cleaned.csv")
df = apply_order(df)
```

The dataset has 4,424 students enrolled in **17 degree courses**, identified by `Course` (numeric code) and `Course_Name` (label). Two of them are evening courses, `Management (evening)` and `Social Service (evening)`: the variable `Daytime_Evening` coincides exactly with these two courses (268 + 215 = 483 evening students), so it must be read together with the course and not as a separate factor.

The scripts define the same outcome colours as the rest of the project (Dropout dark red `#9b4141`, Enrolled grey `#b8b8b8`, Graduate teal `#298c8c`). Block E mostly uses the dropout colour for the dropout rate, and these specific palettes:

| Chart | Colours |
|---|---|
| E1 | Dark red (`#9b4141`) |
| E2 | Blue points (`#0072B2`), dark red regression line |
| E3 | Women `#CC79A7` (pink), men `#0072B2` (blue) |
| E4 | Daytime `#0072B2` (blue), evening `#E69F00` (orange) |

The scripts create a `figures` folder, but in the version supplied the `plt.savefig` lines are commented out, so the charts are only displayed (see the last section).

**Important notes about the sample:**

| Question | Students used | Notes |
|---|---:|---|
| E1 | 4,424 | All students and all 17 courses |
| E2 | 4,244 | 180 students with no first-semester approval rate are excluded (see E2) |
| E3 | 3,964 | Only the 13 courses with at least 30 women and 30 men |
| E4 | 1,218 | Only the four variants of Social Service and Management |

---

## How we analyse the data

For each question the scripts combine a chart and a statistical test. Since the dataset contains thousands of students, even a small difference can be statistically significant, so the document also reports an **effect size** to judge whether the difference or relationship is substantial.

| Test | Used for | Effect size | How to read it |
|---|---|---|---|
| Chi-square | Two categorical variables | Cramér's V (phi for 2 × 2 tables) | < 0.1 negligible; 0.1–0.3 weak; 0.3–0.5 moderate; > 0.5 strong |
| Spearman correlation | Two numeric variables | Rho (ρ) | < 0.1 negligible; 0.1–0.3 weak; 0.3–0.5 moderate; > 0.5 strong |

**What the scripts print and what was added.** The scripts print the test statistic, the p-value and the minimum expected count. Cramér's V and phi are **not** printed: they were computed separately on the same data. The same applies to everything labelled **Additional check** below.

**A note on the scripts' wording.** E3 and E4 print the label "Adjusted p-value", but no multiple-testing correction is applied in the code: the values are the **raw** p-values of the chi-square tests. For E3 (13 tests) the correction matters, so the additional check below reports Holm-adjusted p-values.

---

## E1. Dropout rate by course

**Question:** Which courses have the highest dropout rate and which the lowest?

**Variables:** `Course` compared with `Dropout_Binary`.

The script counts students and dropouts per course, sorts the courses by dropout rate, prints the highest and the lowest, plots a sorted horizontal bar chart and runs a chi-square test on the 17 × 2 contingency table.

**Results (from running the script):**

| Course | Students | Dropouts | Dropout rate |
|---|---:|---:|---:|
| Biofuel Production Technologies | 12 | 8 | 66.7% |
| Equinculture | 141 | 78 | 55.3% |
| Informatics Engineering | 170 | 92 | 54.1% |
| Management (evening) | 268 | 136 | 50.8% |
| Basic Education | 192 | 85 | 44.3% |
| Agronomy | 210 | 86 | 41.0% |
| Oral Hygiene | 86 | 33 | 38.4% |
| Animation and Multimedia Design | 215 | 82 | 38.1% |
| Tourism | 252 | 96 | 38.1% |
| Advertising and Marketing Management | 268 | 95 | 35.4% |
| Management | 380 | 134 | 35.3% |
| Social Service (evening) | 215 | 71 | 33.0% |
| Journalism and Communication | 331 | 101 | 30.5% |
| Veterinary Nursing | 337 | 90 | 26.7% |
| Communication Design | 226 | 51 | 22.6% |
| Social Service | 355 | 65 | 18.3% |
| Nursing | 766 | 118 | 15.4% |

- The course with the highest dropout rate is Biofuel Production Technologies (66.7%) but it has only 12 students; the highest rate among larger courses is Equinculture (55.3%, 141 students). The lowest is Nursing (15.4%, 766 students).
- Chi-square: χ² = 298.27, 16 degrees of freedom, `p < 0.001` (statistically significant).
- **Additional check:** Cramér's V = 0.260, a **weak** association (close to the upper end of the band). The smallest expected count is 3.85 because of Biofuel Production Technologies, which is slightly below the usual threshold of 5. Restricting the test to the 15 courses with at least 100 students gives the same result (χ² = 291.12, V = 0.259).

**Interpretation:** the dropout rate changes a lot from one course to another: among the courses with at least 100 students it ranges from 15.4% to 55.3%, about 40 percentage points, and 12 of the 17 courses are above the overall rate of 32.1%. The two evening courses are both above the average. However, the association is weak in statistical terms, so the course alone does not separate dropouts from non-dropouts, and the differences partly reflect who enrolls in each course (see E3 and E4).

---

## E2. Course dropout rate and course performance

**Question:** Is the dropout rate of a course associated with the average performance of the course?

**Variables:** for each course, the dropout rate and the mean first-semester approval rate (`Approval_Rate_1st`).

The approval rate is the number of approved units divided by the number of enrolled units. The script removes students with no approval rate, aggregates the data at **course level** (one point per course) and draws a labelled scatter plot. It then runs a Spearman correlation and draws a second chart with a linear regression line.

**Results (from running the script):**

| Course | Students used | Dropout rate | Mean approval rate (1st semester) |
|---|---:|---:|---:|
| Biofuel Production Technologies | 12 | 66.7% | 0.55 |
| Equinculture | 141 | 55.3% | 0.59 |
| Informatics Engineering | 170 | 54.1% | 0.46 |
| Management (evening) | 268 | 50.8% | 0.60 |
| Basic Education | 192 | 44.3% | 0.74 |
| Agronomy | 210 | 41.0% | 0.64 |
| Oral Hygiene | 86 | 38.4% | 0.68 |
| Tourism | 252 | 38.1% | 0.72 |
| Advertising and Marketing Management | 268 | 35.5% | 0.72 |
| Management | 380 | 35.3% | 0.65 |
| Social Service (evening) | 215 | 33.0% | 0.71 |
| Journalism and Communication | 331 | 30.5% | 0.80 |
| Veterinary Nursing | 337 | 26.7% | 0.74 |
| Communication Design | 226 | 22.6% | 0.84 |
| Social Service | 355 | 18.3% | 0.80 |
| Nursing | 766 | 15.4% | 0.83 |
| Animation and Multimedia Design | 35 | 14.3% | 0.91 |

- Spearman correlation across the 17 courses: ρ = −0.875, `p < 0.001` (p = 4.3 × 10⁻⁶), a **strong** negative correlation: courses where students pass fewer units in the first semester have a higher dropout rate.
- **Additional check:** the linear fit used for the regression line corresponds to about 11 points lower dropout rate for every 10-point increase in the average approval rate (descriptive only).

**Important detail about the sample (additional check):** the 180 students with no approval rate are students enrolled in 0 units in the first semester, and **all of them belong to Animation and Multimedia Design**. Removing them leaves only 35 of the 215 students of that course, so its dropout rate in E2 is 14.3%, against 38.1% in E1, and it appears as the course with the highest approval rate (0.91). This point is probably an artefact of the exclusion. The correlation remains strong with more robust choices: ρ = −0.75 (p = 0.0005) when the dropout rate of each course is taken from the full sample, and ρ = −0.83 (p = 0.0003) when only the 14 courses with at least 100 students are kept.

**Interpretation:** courses with weaker first-year performance have higher dropout. This should be read with caution for three reasons. First, the two quantities are partly linked by construction: students who drop out pass fewer units (Block D), so courses with many dropouts have a lower average approval rate. Second, the correlation is computed on 17 courses and describes courses, not individual students. Third, the point of Animation and Multimedia Design is affected by the exclusion above.

---

## E3. Gender gap by course

**Question:** Does the gap between men and women in dropout change from course to course?

**Variables:** `Gender` and `Course` compared with `Dropout_Binary`.

Courses with few students of one gender give unstable rates, so the script keeps only the courses with **at least 30 women and 30 men** (`MIN_STUDENTS = 30`): 13 courses and 3,964 students. The excluded courses are Basic Education (183 women, 9 men), Biofuel Production Technologies (3, 9), Informatics Engineering (7, 163) and Oral Hygiene (67, 19). The script plots a grouped horizontal bar chart (women and men for each course, sorted by overall dropout rate) and runs one chi-square test per course.

**Results (from running the script, with phi and Holm-adjusted p-values added):**

| Course | Women (n) | Men (n) | Dropout women | Dropout men | Gap (men − women) | p (raw) | p (Holm, additional) | phi (additional) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Social Service | 309 | 46 | 13.9% | 47.8% | +33.9 | < 0.001 | < 0.001 | 0.28 |
| Veterinary Nursing | 277 | 60 | 21.3% | 51.7% | +30.4 | < 0.001 | < 0.001 | 0.25 |
| Management | 218 | 162 | 24.8% | 49.4% | +24.6 | < 0.001 | < 0.001 | 0.25 |
| Management (evening) | 132 | 136 | 40.9% | 60.3% | +19.4 | 0.002 | 0.021 | 0.19 |
| Agronomy | 61 | 149 | 27.9% | 46.3% | +18.4 | 0.021 | 0.145 | 0.16 |
| Equinculture | 79 | 62 | 48.1% | 64.5% | +16.4 | 0.076 | 0.379 | 0.15 |
| Advertising and Marketing Management | 146 | 122 | 28.1% | 44.3% | +16.2 | 0.009 | 0.068 | 0.16 |
| Communication Design | 130 | 96 | 16.9% | 30.2% | +13.3 | 0.028 | 0.167 | 0.15 |
| Nursing | 617 | 149 | 13.1% | 24.8% | +11.7 | < 0.001 | 0.006 | 0.12 |
| Tourism | 152 | 100 | 34.2% | 44.0% | +9.8 | 0.152 | 0.607 | 0.09 |
| Social Service (evening) | 173 | 42 | 31.2% | 40.5% | +9.3 | 0.336 | 0.672 | 0.07 |
| Journalism and Communication | 216 | 115 | 27.8% | 35.7% | +7.9 | 0.175 | 0.607 | 0.08 |
| Animation and Multimedia Design | 98 | 117 | 36.7% | 39.3% | +2.6 | 0.805 | 0.805 | 0.02 |

(The dropout rates by gender are shown in the chart but are not printed by the script; they were computed separately.)

- Men drop out more than women in **all 13 courses**, with a gap between 2.6 and 33.9 percentage points.
- The gap is statistically significant (raw p < 0.05) in 8 of the 13 courses. After correcting for 13 tests (Holm), it remains significant in 5: Social Service, Veterinary Nursing, Management, Nursing and Management (evening).
- All minimum expected counts are above 5 (the smallest is 8.4, Social Service).
- **Additional check (overall):** in the full sample, 25.1% of women and 45.1% of men drop out (Cramér's V = 0.20, weak).

**Interpretation:** the direction of the gap is the same everywhere, but its size changes a lot: it is very large in Social Service, Veterinary Nursing and Management, and small in Animation and Multimedia Design, Journalism and Communication, Social Service (evening) and Tourism. A non-significant result does not mean that there is no gap: several of these courses have few men (for example 42 in Social Service (evening)), so the estimates are uncertain. The scripts test each course separately and do not include a formal test of the gender × course interaction, so the statement "the gap varies by course" is based on the descriptive differences. Gender is also strongly linked to the course itself (for example Informatics Engineering is almost entirely male, Nursing mostly female), so the gap partly reflects the composition of each course.

---

## E4. Evening and daytime versions of the same course

**Question:** Do evening courses have a different dropout rate from the same courses in the daytime?

**Variables:** `Course_Name` (Social Service and Management, evening and daytime) and `Daytime_Evening` compared with `Dropout_Binary`.

The script keeps the four variants (`Social Service`, `Social Service (evening)`, `Management`, `Management (evening)`), merges each evening course with its daytime version into a common `Course_Base`, plots a grouped bar chart (course × attendance mode) and runs one chi-square test per course (daytime against evening).

**Results (from running the script):**

| Course | Daytime: students (dropout rate) | Evening: students (dropout rate) | Gap (evening − daytime) | χ² | p |
|---|---|---|---:|---:|---:|
| Social Service | 355 (18.3%) | 215 (33.0%) | +14.7 | 15.16 | < 0.001 |
| Management | 380 (35.3%) | 268 (50.8%) | +15.5 | 14.87 | < 0.001 |

- Both differences are statistically significant. All expected counts are well above 5. Both stay significant even with a correction for the two tests.
- **Additional check:** the effect is **weak** in both courses (phi = 0.17 and 0.16); the odds of dropping out in the evening course are about 2.2 times (Social Service) and 1.9 times (Management) those of the daytime course.

**Additional check (age of the students):** evening students are much older than daytime students: the median age is 33 against 19 in Social Service, and 31 against 20 in Management. Splitting the students at age 23, the evening courses do not have a higher dropout rate among students aged 23 or older: in Social Service 36.6% in the evening (186 students) against 41.3% in the daytime (46 students), and in Management 50.6% (255) against 60.4% (106). Among students younger than 23 the evening groups are very small (29 and 13 students), so no reliable comparison is possible.

**Interpretation:** both evening courses have a dropout rate about 15 percentage points higher than their daytime versions. However, the difference cannot be attributed to the schedule: evening courses attract older students, who drop out more whatever the schedule (consistent with the over-23 admission route in Block C and with Block A). The age comparison is based on small subgroups, so it should be presented as a warning against a causal reading, not as proof that the schedule has no effect.

---

## Block E: overall conclusions

| Section | Main result | Effect size / strength |
|---|---|---|
| E1 — Dropout by course | Dropout ranges from 15.4% (Nursing) to 55.3% (Equinculture) among courses with at least 100 students | Weak (Cramér's V = 0.26) |
| E2 — Course dropout and approval rate | Courses with lower first-semester approval have higher dropout (17 courses) | Strong negative (ρ = −0.875; −0.83 with courses of at least 100 students) |
| E3 — Gender gap by course | Men drop out more in all 13 courses (+2.6 to +33.9 points); significant in 8 courses, 5 after correction | Weak overall (V = 0.20); phi from 0.02 to 0.28 by course |
| E4 — Evening and daytime | Evening versions have about 15 points higher dropout, but evening students are much older | Weak (phi = 0.17 and 0.16) |
