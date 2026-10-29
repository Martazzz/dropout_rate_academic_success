# Data cleaning: what we did and why
Cleaning in notebook: `01_cleaning_eda.ipynb`. Clean file: `student_dropout_cleaned.csv`.

---

## The clean dataset

| | |
|---|---|
| Students (rows) | 4424 (none was removed) |
| Columns | 41 |
| Missing values | only the intended ones |

The 41 columns are:

| Group | Columns |
|---|---|
| **Original** | the 22 columns we keep (21 variables + `Target`)|
| **Decoded or grouped** | `Course_Name`, `Marital_Group`, `Application_Mode_Group`, `Application_Order_Group`, `Parent_Education` | 
| **Derived** (created by us) | `Dropout_Binary`, `Approval_Rate_1st/2nd`, `Passed_None_1st/2nd`, `Grade_Change`, `Age_Group`, `Admission_Grade_Band`, `Financial_Risk` |
| **0/1 numeric codes** | `Gender_num`, `Displaced_num`, `Debtor_num`, `Tuition_Up_To_Date_num`, `Scholarship_Holder_num` |


To use the clean data file in the other notebooks you need to:
```python
data = apply_order(pd.read_csv("student_dropout_cleaned.csv"))
```
---

## Choices made

### Selecting the columns (22 out of 37)
We removed 15 columns because they were uninformative or redundant:

| Removed column | Reason |
|---|---|
| Nacionality | almost all students have the same value |
| International | very few students|
| Educational special needs | very few students |
| Previous qualification | almost everyone has the same value |
| Mother's/Father's occupation | a lot of different codes |
| Curricular units (credited) | no relationship with the outcome |
| Curricular units (evaluations) | redundant with "enrolled" and "approved" |
| Curricular units (without evaluations) | uninformative |
| Unemployment rate, Inflation rate, GDP | they depend on the enrollment period and show no relationship with dropout |

### Checking missing values and duplicates
The original file has **no missing values and no duplicated rows**. 

### Decoding the categorical variables
Almost all categorical variables are stored as numeric codes. We translate them into labels using the dictionary in the dataset paper:
- `Gender`: 1 = Male, 0 = Female
- `Displaced`, `Debtor`, `Tuition_Up_To_Date`, `Scholarship_Holder`: 1 = Yes, 0 = No
- `Daytime_Evening`: 1 = Daytime, 0 = Evening
- `Course` → `Course_Name` (17 courses)

### Grouping categories with too many values
For readable charts we deacided to group some similar values.
- `Marital_Status` → `Marital_Group`: Single / Not single
- `Application_Mode` → `Application_Mode_Group`: 18 different codes into 5 groups (`General contingent`: ordinary admission, `Over 23 years old`: access reserved for people over 23, `Transfer or change`: transfer or change of course/institution, `Prior diploma holder`: people who already have another qualification, `Special or other`: special contingents, ordinances and other routes)
- `Application_Order` → `Application_Order_Group`: `0-1`, `2`, `3+`
- Parents' qualification → `Parent_Education`

### Grades and approval rate
This is the most important choice. `Grade_1st` and `Grade_2nd` are the **average grade of the passed units**. In the file the value 0 appears only when the approved units are 0 (718 students in the 1st semester, 870 in the 2nd): it means **"no unit passed"**. If we kept the 0, the grade averages would be much lower than the real ones. So:
- a grade of 0 becomes a **missing value** (`NaN`)
- the information "no unit passed" is kept by the addition variable `Passed_None_*`

180 students are registered at the university but in 0 units in a semester. For them the approval rate cannot be computed, it stays as a missing value (`NaN`).

---

## The variables we created

- `Dropout_Binary`: it's 1 if the student dropped out (`Target` = Dropout), 0 in all other cases (Enrolled and Graduate).

- `Approval_Rate_1st`, `Approval_Rate_2nd` (approval rate): it is computed as approved units ÷ enrolled units, for each semester. It goes from 0 to 1. 0 = no unit passed, 1 = all the units the student was enrolled in were passed, 0.5 = half of them were passed.**Special case:** if the enrolled units are 0, the formula would be 0 ÷ 0, so the value is missing.

- `Passed_None_1st`, `Passed_None_2nd`: it is "Yes" if the student was enrolled in at least one unit and passed none in the semester, "No" if they passed at least one. Empty for students enrolled in 0 units.

- `Grade_Change`: it is computed as `Grade_2nd` − `Grade_1st`. If it's positive:the grade improved in the 2nd semester, negative:it got worse. This variable is defined only for students with a grade in both semesters.

- `Age_Group` and `Admission_Grade_Band`: `Age_Group`: age at enrollment in four bands chosen by us: 17-19, 20-24, 25-29, 30. `Admission_Grade_Band`: the admission grade split into four quartiles. The real ranges are Q1 = 95-117.9, Q2 = 118-126.1, Q3 = 126.2-134.8, Q4 = 134.9-190.

- `Financial_Risk`: it is computed as the sum of three conditions (1 fo each if they're true):
    1. the student is a debtor (`Debtor` = Yes);
    2. the tuition fees are not up to date (`Tuition_Up_To_Date` = No);
    3. the student has no scholarship (`Scholarship_Holder` = No).
    
    The results go from 0 to 3. **It's important that:** tuition fees up to date may be a consequence of dropout (someone who leaves may stop paying).

- The numeric codes:`Gender_num`, `Displaced_num`, `Debtor_num`, `Tuition_Up_To_Date_num`, `Scholarship_Holder_num` are **0/1** versions of the binary variables. 

- `Parent_Education`: to each qualification code of the mother and the father we assign a level (`Basic`, `Secondary`, `Higher`, or unknown). Then we take the highest level between the two parents. **Levels:**
    - `Basic`: education up to the 9th year of school;
    - `Secondary`: high school and technical/professional;
    - `Higher`: university education.
---

## Remaining missing values

| Column | Missing | Why |
|---|---|---|
| `Grade_1st` | 718 | no unit passed in the 1st semester |
| `Grade_2nd` | 870 | no unit passed in the 2nd semester |
| `Approval_Rate_1st`, `Passed_None_1st` | 180 | enrolled in 0 units in the 1st semester |
| `Approval_Rate_2nd`, `Passed_None_2nd` | 180 | enrolled in 0 units in the 2nd semester |
| `Grade_Change` | 912 | the grade is missing in at least one of the two semesters |

How to handle the missing values for Clustering and Random Forest: fill `Grade_*` with 0 (the original value, meaning "no unit passed") and `Approval_Rate_*` with 0 for the 180 students with no enrolled units, keeping the `Approved_*` and `Passed_None_*` variables.

