# Predict Students' Dropout and Academic Success
Project of scientific data visualization made by the students Crippa, Zanchetta and Zribi

We analyse the **Predict Students' Dropout and Academic Success** dataset (UCI): 4,424 students of a Portuguese institution, 17 degree courses, with the final outcome of each student (**Dropout**, **Enrolled** or **Graduate**). About 1 student in 3 drops out.

## Goal

1. **Understand** which factors go with dropout.
2. **Describe** the main student profiles.
3. **Predict** how early dropout can be spotted.

## Method

The project follows six steps:

| Step | What we do |
|---|---|
| 0. Cleaning | Recode variables, build new ones, handle missing values |
| 1. EDA | Dropout rates by group, correlations, first signals |
| 2. Tests | Chi-square, Kruskal-Wallis and Spearman, always with an effect size (Cramér's V, epsilon squared, rho), in five blocks: **A** demographics, **B** economic situation, **C** academic background and enrollment, **D** first-year performance, **E** degree course |
| 3. Clustering | K-Means on student profiles; `Target` is used only afterwards to describe the clusters |
| 4. Random Forest | Prediction in four cumulative scenarios (at enrollment, + economic, after 1st semester, after 2nd semester), compared with baselines |
| 5. Conclusions | Main findings, limits |

## Main results

- **First-year performance is the strongest signal**: 92% of the students who passed no unit in the 1st semester dropped out, against 9% of those who passed all (Cramér's V = 0.61).
- **Economic situation**: 87% dropout when tuition fees are not up to date, against 25%.
- **Clustering**: at enrollment, the mature-student profile has the highest dropout (57%).
- **Early prediction**: F1 macro 0.48 at enrollment, 0.67 after the 1st semester, 0.69 after the 2nd. The 1st semester is the turning point.


## Repository content

- `student_dropout_cleaned.csv`: dataset after the cleaning phase
- Notebooks and scripts for each phase (cleaning, EDA, blocks A-E, clustering, Random Forest)
- `figures/`: charts used in the presentation
- Presentation 

## Authors

Crippa, Zanchetta, Zribi
