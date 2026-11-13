# Phase 3: Clustering

## Goal of the phase

The purpose of Phase 3 is to group students who are **similar to each other** into a small number of **profiles**, and then to check whether the profiles have different dropout rates.

The key rule is that **`Target` is excluded from the clustering**. It is used only afterwards, to describe the profiles. If it were used to build the groups, the result would be circular: we would find groups with many dropouts because we put the dropouts there.

## Data and setup

The dataset has 4,424 students. The script loads it and restores the category order used in the other blocks.

**Variables used in the clustering** (as in the project plan):

| Group | Variables | Type |
|---|---|---|
| Profile | `Age_At_Enrollment`, `Admission_Grade`, `Prev_Qual_Grade` | numeric |
| Profile | `Gender_num` (1 = male), `Scholarship_Holder_num`, `Debtor_num`, `Tuition_Up_To_Date_num` (1 = yes) | binary |
| Behavior (academic results) | `Approval_Rate_1st`, `Approval_Rate_2nd` | numeric |

The semester grades (`Grade_1st`, `Grade_2nd`) are **not** used by default (see below).

**Preparation choices:**

- **Standardization.** The numeric variables are standardized with `StandardScaler` (z-score: mean 0, standard deviation 1). Without it, variables with larger numbers (age in years, grades on a 0–200 scale) would dominate the distances over variables on a 0–1 scale such as the approval rate.
- **Age in logarithm.** Age is skewed (median 20, mean 23.3, maximum 70). The logarithm (`AGE_TRANSFORM = "log"`) prevents a few very old students from moving the cluster centers.
- **Binary variables as 0/1**, as in the plan (`BINARY_MODE = "raw"`). A 0/1 variable has a smaller spread than a standardized one, so the binaries weigh less in the distances. This is why gender, scholarship, debtor and tuition status have little influence on the clusters (see the PCA loadings). Setting `BINARY_MODE = "scaled"` standardizes them too.
- **Students with a missing approval rate are excluded** from the scenario that uses academic results: 180 students (4,244 are used). They have 0 enrolled units in both semesters, so the approval rate is undefined; checking the dataset, all of them are in Animation and Multimedia Design. We decided not to invent a value for them.
- **Semester grades are not used by default.** Checking the dataset before writing the script, the grades are missing for 718 students (1st semester) and 870 (2nd semester), almost all students who passed no unit; for them the grade is undefined, not zero. The grades are also strongly correlated with the approval rates, so they add little information. With `USE_GRADES = True` the grades are added, missing values are filled with the median and a 0/1 flag `<grade>_missing` is added.

## Two scenarios and the problem of circularity

Excluding `Target` is not enough to avoid circularity: some variables are partly **consequences** of dropping out. A student who leaves early passes few units, and may have unpaid fees. The script therefore runs two scenarios:

| Scenario | Variables | What it shows |
|---|---|---|
| `enrollment` | The 7 profile variables only | What can be said using information available **at enrollment** |
| `full` | The 7 profile variables + the two approval rates (as in the project plan) | A fuller description of students, but with partial circularity |

In the `full` scenario, the dropout rate of a low-performance cluster is high almost by construction, and it should not be presented as an independent finding.

---

## Choosing k

For k from 2 to 8, the script computes three criteria:

- **Elbow method (inertia):** the total squared distance between students and their cluster center. It always decreases when k grows; the elbow is where the decrease slows down.
- **Silhouette score** (from −1 to 1): how much a student is closer to their own cluster than to the nearest other one. Higher means better-separated clusters.
- **Stability:** the K-Means partition is compared with 5 runs from different random starts using the adjusted Rand index (ARI; 1 = identical partitions).

**Results (from running the script):**

| k | Enrollment: inertia | Enrollment: silhouette | Enrollment: stability | Full: inertia | Full: silhouette | Full: stability |
|---:|---:|---:|---:|---:|---:|---:|
| 2 | 11,741 | 0.241 | 0.999 | 16,831 | 0.355 | 0.997 |
| 3 | 8,938 | **0.276** | 1.000 | 13,654 | 0.234 | 1.000 |
| 4 | 7,762 | 0.228 | 0.982 | 11,831 | **0.247** | 1.000 |
| 5 | 7,001 | 0.230 | 0.996 | 10,860 | 0.209 | 0.987 |
| 6 | 6,536 | 0.175 | 0.774 | 10,086 | 0.195 | 0.978 |
| 7 | 6,127 | 0.180 | 0.912 | 9,509 | 0.193 | 0.945 |
| 8 | 5,766 | 0.191 | 0.982 | 9,054 | 0.188 | 0.944 |

- The inertia drops most from k = 2 to k = 3 (−24% in `enrollment`, −19% in `full`) and by smaller steps afterwards, so the elbow alone does not settle k.
- The silhouette scores are low (0.23–0.28 for the chosen values): the clusters are **not well separated**.
- The stability is close to 1 for the chosen values, so the result is reproducible; the least stable value is k = 6 in `enrollment` (0.774).
- **Rule used:** the k with the best silhouette among k ≥ 3, which gives **k = 3 for `enrollment` and k = 4 for `full`**. In the `full` scenario k = 2 has a higher silhouette (0.355), but with only two clusters the segmentation would have too few profiles; the two-cluster solution was not inspected. The rule is a choice, and k can be changed in the `K_FINAL` setting.

---

## Final model and PCA

The final model is a K-Means with 50 random starts (`n_init = 50`, `random_state = 42`). Clusters are numbered by size, so cluster 0 is always the largest.

The clustering takes place in the full space of the variables. The **PCA** only projects the students on two axes to draw them: the two components explain **72.1%** of the variance in `enrollment` and **64.4%** in `full`. Two clusters that overlap in the plot may still be separated in the full space.

How to read the axes (loadings of the PCA):

| Scenario | PC1 | PC2 |
|---|---|---|
| `enrollment` | Admission grade (0.69) and previous qualification grade (0.70) | Age (0.96) |
| `full` | Approval rates (0.60 and 0.60), age with a negative sign (−0.33) | Admission grade (0.65) and previous qualification grade (0.65) |

In both scenarios the binary variables have loadings close to zero.

---

## Cluster profiles

The names below are **provisional**: they summarize the heatmap and should be checked and possibly changed. In the code they are set in `CLUSTER_NAMES` and are valid only for the default run (`random_state = 42`, k = 3 and k = 4).

### Scenario `enrollment` (k = 3)

| Cluster | Students | Share | Age | Admission grade | Previous grade | Male | Scholarship | Debtor | Tuition up to date |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 Young, average grades | 2,213 | 50.0% | 20.0 | 119.6 | 126.1 | 0.33 | 0.29 | 0.10 | 0.90 |
| 1 Young, high grades | 1,378 | 31.1% | 20.5 | 140.5 | 145.7 | 0.31 | 0.28 | 0.09 | 0.93 |
| 2 Mature students | 833 | 18.8% | 36.5 | 124.3 | 128.4 | 0.48 | 0.10 | 0.18 | 0.75 |

(Values for the binary variables are shares; the other columns are means in original units.)

**Interpretation:** the clusters are distinguished mainly by **age** (the mature cluster) and by **grades** (the two young clusters). The two young clusters have almost the same gender, scholarship, debtor and tuition shares. The mature cluster has fewer scholarship holders (10%), more debtors (18%) and fewer students with tuition up to date (75%).

### Scenario `full` (k = 4)

| Cluster | Students | Share | Age | Admission grade | Previous grade | Male | Scholarship | Debtor | Tuition up to date | Approval 1st | Approval 2nd |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| 0 Young, on track | 1,767 | 41.6% | 19.9 | 119.9 | 126.0 | 0.29 | 0.33 | 0.08 | 0.94 | 0.87 | 0.84 |
| 1 Bright and on track | 1,115 | 26.3% | 20.2 | 139.9 | 145.9 | 0.26 | 0.31 | 0.08 | 0.95 | 0.90 | 0.87 |
| 2 Passing almost nothing | 873 | 20.6% | 26.7 | 122.8 | 130.3 | 0.53 | 0.06 | 0.22 | 0.67 | 0.14 | 0.08 |
| 3 Mature, on track | 489 | 11.5% | 37.6 | 127.1 | 128.5 | 0.40 | 0.13 | 0.12 | 0.88 | 0.84 | 0.82 |

**Interpretation:** the clusters are distinguished by **academic results** (cluster 2 passes almost no unit, with approval rates of 0.14 and 0.08), by **grades at entry** (cluster 1) and by **age** (cluster 3). Cluster 2 also has the lowest share of scholarship holders (6%), the highest share of debtors (22%) and the lowest share of students with tuition up to date (67%). Its mean age (26.7) is above that of the young clusters, so it includes some older students; this was not broken down further.

---

## Dropout rate by cluster

`Target` is used here for the first time. The script draws a bar chart of the dropout rate per cluster (with the number of students and the overall rate as a reference line), a stacked bar chart of the three outcomes per cluster, and runs chi-square tests with Cramér's V.

**Results (from running the script):**

| Scenario | Cluster | Students | Dropouts | Dropout rate |
|---|---|---:|---:|---:|
| `enrollment` | 0 Young, average grades | 2,213 | 638 | 28.8% |
| `enrollment` | 1 Young, high grades | 1,378 | 308 | 22.4% |
| `enrollment` | 2 Mature students | 833 | 475 | 57.0% |
| `full` | 0 Young, on track | 1,767 | 277 | 15.7% |
| `full` | 1 Bright and on track | 1,115 | 138 | 12.4% |
| `full` | 2 Passing almost nothing | 873 | 775 | 88.8% |
| `full` | 3 Mature, on track | 489 | 154 | 31.5% |

The overall dropout rate is 32.1% in `enrollment` (4,424 students) and 31.7% in `full` (4,244 students).

| Scenario | Test | χ² | Degrees of freedom | p | Cramér's V |
|---|---|---:|---:|---|---:|
| `enrollment` | Cluster × Target (3 outcomes) | 338.94 | 4 | < 0.001 | 0.196 (weak) |
| `enrollment` | Cluster × Dropout (yes/no) | 308.23 | 2 | < 0.001 | 0.264 (weak) |
| `full` | Cluster × Target (3 outcomes) | 1,802.36 | 6 | < 0.001 | 0.461 (moderate) |
| `full` | Cluster × Dropout (yes/no) | 1,716.23 | 3 | < 0.001 | 0.636 (strong) |

The smallest expected counts are 149.5 and 88.3, so the chi-square conditions are met.

**Interpretation:**

- In `enrollment` the mature cluster has a dropout rate (57.0%) about twice that of the two young clusters (28.8% and 22.4%). This is consistent with the over-23 admission route in Block C. The association is weak.
- In `full` the cluster that passes almost nothing has a dropout rate of 88.8%. **This is not an independent finding**: approval rates are partly a consequence of dropping out, so the result is high almost by construction.
- In `full`, the mature students who are on track (cluster 3) have a dropout rate (31.5%) close to the overall average, and the young students on track have the lowest rates (15.7% and 12.4%). The difference between the two on-track young clusters is small (3.3 points).

---

## Phase 3: overall conclusions

| Scenario | Main result | Strength |
|---|---|---|
| `enrollment` (k = 3) | A mature cluster (19% of students) has a 57.0% dropout rate, against 28.8% and 22.4% for the two young clusters | Weak (V = 0.264 for dropout yes/no) |
| `full` (k = 4) | A cluster that passes almost nothing (21%) has an 88.8% dropout rate; the other three clusters are between 12.4% and 31.5% | Strong (V = 0.636), but partly circular |

### Main message

Two dimensions describe the students: **age** (mature students) and **first-year academic results**. Using only information available at enrollment, mature students are the profile with the highest dropout, in line with the over-23 route of Block C. Adding academic results, a group that passes almost no units appears, with a very high dropout rate.
