# EDA: what we did and why
EDA in notebook: `01_cleaning_eda.ipynb`. Data: `student_dropout_cleaned.csv`. Charts saved in `figures/`.

---

The EDA is the first look at the clean data, **from the general to the particular**. We want to know who the students are, how many of them drop out, and which variables seem related to dropout. 

In the charts we used:
- the **dropout rate** (percentage of students who dropped out), instead of counts, because the groups have very different sizes.
- the number of students of each group (`n`) is always shown, because a rate on a small group is not reliable.


| Step | Chart | File in `figures/` |
|---|---|---|
| 1. Outcome distribution | bar chart | `eda_target.png` |
| 2. Sample composition | horizontal bar charts | `eda_composition.png`, `eda_courses.png` |
| 3. Numeric variables | histograms | `eda_numeric.png` |
| 4. Dropout rate by category | horizontal bar charts | `eda_dropout_rate_grid.png` |
| 5. Dropout rate by course | horizontal bar chart | `eda_dropout_by_course.png` |
| 6. Correlations | heatmap + bar chart | `eda_heatmap.png`, `eda_corr_dropout.png` |
| 7. Results and age by outcome | boxplots | `eda_boxplots_by_outcome.png` |

---

### 1. Outcome distribution
We counted the students of each outcome (`Dropout`, `Enrolled`, `Graduate`). We have one bar per outcome, with number and percentage. The colors are fixed in the whole project (red = dropout, gray = enrolled, teal = graduate).

The chart shows that 50% of the students graduated (2209), 32% dropped out (1421) and 18% are still enrolled (794). `Enrolled` is not a final outcome: these students have not finished yet.

### 2. Sample composition
For each categorical variable we counted the students in each category (horizontal bars, with the number next to the bar). A second chart does the same for the courses.

The charts show that:
- 65% of the students are women, most are single (89%), have no debts (89%) and have tuition fees up to date (88%).
- 76% are 25 or younger, but 16% are 30 or older (the age distribution has a long tail).
- Only 25% have a scholarship, 11% attend evening classes.
- Parents: the highest education level is `Basic` for 50% of the students, `Secondary` for 30% and `Higher` for 18%.
- The ordinary admission route (`General contingent`) is 61% of the students.
- Nursing is the largest course (17% of the students).

### 3. Numeric variables
We produced a histogram for age, admission grade, previous qualification grade, approved units, semester grades and approval rate of the 1st semester.

From the charts we see that:
- Age is strongly skewed to the right: most students are young.
- Admission and previous qualification grades have a regular shape, with a slight tail towards high values.
- Semester grades are concentrated around 12-13 (on a scale from 0 to 20).
- The approval rate of the 1st semester is concentrated on high values: about 41% of the students enrolled in at least one unit passed all their units, and about 13% passed none.

### 4. Dropout rate by category
For each categorical variable we computed the dropout rate. The **dashed black line** is the dropout rate of **all** students (32%) and it is the reference: a bar longer than the line means a group that drops out more than the average student, a shorter bar means less.

The chart shows that:
| Variable | Dropout rate | 
|---|---|
| Tuition fees up to date | No: 87% (n=528), Yes: 25% | 
| Passed no unit in the 1st semester | Yes: 92% (n=538), No: 23% | 
| Financial risk (0-3 factors) | 0: 9%, 1: 29%, 2: 69%, 3: 89% | 
| Debtor | Yes: 62%, No: 28% | 
| Scholarship holder | Yes: 12%, No: 39% | 
| Age group | 17-19: 21%, 20-24: 29%, 25-29: 58%, 30+: 54% | 
| Application mode | Over 23: 55%, General contingent: 24% | 
| Gender | Male: 45%, Female: 25% | 
| Marital status | Not single: 47%, Single: 30% | 
| Schedule | Evening: 43%, Daytime: 31% | 
| Displaced | Yes: 28%, No: 38% | 
| Admission grade | Q1: 40%, Q2: 31%, Q3: 29%, Q4: 29% |
| Application order | 0-1: 35%, 2: 27%, 3+: 26% | 
| Parents' education | Basic 32%, Secondary 30%, Higher 31% | 

### 5. Dropout rate by course
We shown the dropout rate of each course, sorted from the lowest to the highest. We show only the courses with at least 100 students (two courses are left out because they are too small).

We see that the dropout rate goes from 15% in Nursing to 55% in Equinculture (Informatics Engineering 54%, Management evening 51%). The evening courses are higher than the same courses in the daytime (Management: 51% vs 35%, Social Service: 33% vs 18%). 

### 6. Correlations
We computed the Spearman correlation between the numeric variables (and the 0/1 versions of the binary ones) and `Dropout_Binary`. Spearman is used because it works with ordered values and does not need a normal distribution. The results are from -1 to +1. 
- Around 0 = no relationship
- Positive(red) = when the variable increases, dropout tends to increase
- Negative (blue) = when it increases, dropout tends to decrease. 

From the heatmap we see that:
- The strongest relationships are with the **academic progress**: approval rate in the 2nd (-0.65) and 1st (-0.59) semester, approved units in the 2nd (-0.58) and 1st (-0.52) semester.
- Then tuition fees up to date (-0.43), age at enrollment (+0.28), grades (-0.25 and -0.24), scholarship (-0.25), debtor (+0.23), gender (male = 1: +0.20).

### 7. Results and age by outcome
We did a boxplot of approved units, grades, age and admission grade for each outcome, plus the median of each group. The box contains the central 50% of the students, the line inside is the median. If two boxes are at the same height, the groups are similar. 

We see that the median number of approved units in the 1st semester is 2 for students who dropped out, 5 for those still enrolled and 6 for graduates, in the 2nd semester it is 0, 4 and 6. Grades are only slightly different (12 for dropouts, 13 for graduates). The median age is 23 for dropouts, 20 for enrolled and 19 for graduates. The admission grade is similar in the three groups (123.6, 124.1, 127.4).


## Key findings

1. **Half of the students graduate, one in three drops out**
2. **The strongest signal is academic progress**: students who pass no unit in the 1st semester drop out 92% of the time, against 23% for the others.
3. **The economic situation is strongly associated with dropout**: tuition fees not up to date (87%), debts (62%), no scholarship (39%) the risk grows with the number of factors (from 9% to 89%).
4. **Age matters**: students who enroll at 25 or older drop out more
5. **Men and not-single students drop out more** than women and single students.
6. **The course makes a difference**, some courses have a higher dropout rate (15%:Nursing (lower), 55%:Equinculture (higher)).
7. **Weak variables are:** parents' education, previous qualification grade and application order.

---

## What is better to analyise in the next step?

| Block | Signals found in the EDA to investigate |
|---|---|
| **A** (Demographics) | age, gender, marital status, displaced |
| **B** (Economic situation) | tuition fees, debtor, scholarship, financial risk |
| **C** (Admission and path) | admission grade (lowest quartile), application mode, application order |
| **D** (First-year performance) | approval rate, "no unit passed", grades, when the differences become visible |
| **E** (Course) | differences between courses and between evening and daytime courses |

