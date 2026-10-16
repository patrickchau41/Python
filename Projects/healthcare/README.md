# Diabetic Patient 30-Day Readmission Analysis

**Answer in brief:** A hospital should follow up first with patients discharged to rehab or with 2 or more prior inpatient stays (the High tier). That group is 5% of patients and is readmitted at 23%, compared with under 7% in the Standard tier. The two factors stack, and both hold within every age group.

## Project Overview

Hospitals can face financial penalties for some preventable readmissions, and follow-up care can help prevent many of them. This project analyzes hospital encounters for diabetic patients to find which patient and hospital-stay factors are linked to 30-day readmission, and which patients a hospital should prioritize for follow-up.

**Business question:** Which diabetic patient and hospital-encounter factors are linked to 30-day readmission, and which patients should a hospital prioritize for follow-up to prevent avoidable readmissions?

## Dataset

- **Source:** Diabetes 130-US Hospitals dataset (encounters from 130 US hospitals, 1999 to 2008)
- **Size after cleaning:** 69,982 patients (each patient's first visit), 50 columns
- **Target:** `readmitted_30d` (1 if the patient returned within 30 days, 0 otherwise)
- **Overall 30-day readmission rate:** 8.98% (6,283 of 69,982)
- **Cleaning steps:** see Data Cleaning below

## Data Inspection

Before cleaning, the raw file was inspected with `df.shape`, `df.head()`, `df.info()` and a count of missing values per column.

**Results**

- The raw file has 101,766 rows and 50 columns. Each row is one hospital visit (an encounter) for a diabetic patient.
- There are only 71,518 unique patients, so about 30,000 rows are repeat visits by patients who already appear in the data.
- 13 columns are numbers (visit counts, days in hospital, ID codes) and 37 are text (age ranges, diagnosis codes, medication results, the readmission label).
- Three columns hold numeric codes instead of names: `admission_type_id`, `discharge_disposition_id` and `admission_source_id`. They are translated with the `IDS_mapping.csv` lookup file.
- Missing values by column: weight 96.9%, max_glu_serum 94.7%, A1Cresult 83.3%, medical_specialty 49.1%, payer_code 39.6%, race 2.2%, diag_3 1.4%, diag_2 0.4%, diag_1 21 rows.
- The original `readmitted` column has three values: NO (54,864), >30 (35,545) and <30 (11,357). The raw 30-day readmission rate is 11.16%.
- Some count columns have extreme values with a median of 0: prior inpatient stays go up to 21, ER visits up to 76 and outpatient visits up to 42.

**Insights**

- `weight` and `payer_code` are too empty to use and are dropped.
- A blank `A1Cresult` or `max_glu_serum` most likely means the test was not ordered, so those blanks are kept as a "Not Tested" group instead of being removed.
- Repeat visits mean one patient could be counted many times, so only each patient's first visit is kept.
- The readmission rate falls from 11.16% in the raw file to 8.98% after cleaning. This is expected, because removing patients who died and repeat visits changes who is counted.
- Rare high visit counts are grouped into "3 or more" or "2 or more" during the analysis so a few extreme patients do not distort the results.

## Data Cleaning

View the full notebook: [diabetic_healthcare_cleaned.ipynb](notebooks/diabetic_healthcare_cleaned.ipynb)

**Results**

- Dropped `weight` and `payer_code`. They were missing for 96.9% and 39.6% of patients and could not be used (50 columns to 48).
- Filled blanks with a label instead of removing rows. A blank `A1Cresult` or `max_glu_serum` means the test was not ordered, so those became "Not Tested". Blank `medical_specialty` became "Not Specified", blank `race` became "Unknown", and blank `diag_2` and `diag_3` became "None". Dropping those rows would have removed most of the data.
- Removed 21 patients with no primary diagnosis (101,766 to 101,745 rows). No blanks remained after this step.
- Removed 2,422 patients who died or went to hospice (discharge codes 11, 13, 14, 19, 20, 21), leaving 99,323 rows. These patients cannot be readmitted.
- Kept only each patient's first visit (99,323 to 69,982 rows). This removed 29,341 repeat visits so no patient is counted more than once.
- Created the target column `readmitted_30d`, which is 1 if the patient was readmitted within 30 days and 0 otherwise. Result: 6,283 readmitted patients out of 69,982, or 8.98%.
- Grouped the primary diagnosis into 9 categories. `diag_1` held 694 different ICD-9 codes, so they were grouped into Circulatory, Respiratory, Digestive, Diabetes, Injury, Musculoskeletal, Genitourinary, Neoplasms and Other, using the grouping from the original study. Circulatory is the largest group at 21,392 patients. (The 48 remaining columns plus the target and diagnosis category make the final 50.)

**Insights**

- The final dataset has 69,982 unique patients and 50 columns.
- Each cleaning choice affects the result. Keeping only first visits and removing patients who died lowers the readmission rate from 11.16% in the raw file to 8.98%, so the project's numbers describe a patient's first visit in this data.

## Tools Used

- Python (pandas) in Jupyter Notebook for cleaning and analysis, one notebook per question
- Power BI for the dashboard
- GitHub for version control

## Q1. Are prior hospital visits linked to readmission?

View the notebook: [01_prior_visits.ipynb](notebooks/01_prior_visits.ipynb)

**Results**

- Readmission rises steadily with prior overnight (inpatient) stays: 8.13% with 0, 12.88% with 1, 18.52% with 2, and 26.37% with 3 or more.
- Prior ER visits show the same pattern: 8.74% with 0, 11.23% with 1, 14.45% with 2 or more.
- Prior outpatient visits matter less: 8.80% with 0 versus about 10% with 1 or more.

**Insights**

- Prior inpatient stays are the strongest single warning sign. Patients with 3 or more stays are readmitted at over 3 times the rate of patients with none.

## Q2. Does the length and complexity of the stay matter?

View the notebook: [02_hospital_dtay.ipynb](notebooks/02_hospital_dtay.ipynb)

**Results**

- Readmission rises with days in the hospital: 7.16% for 1 to 2 days, 8.48% for 3 to 4, 10.55% for 5 to 7, and 11.79% for 8 to 14.
- It also rises with the number of diagnoses (6.19% to 10.05%), medications (7.30% to 10.46%) and lab tests (7.63% to 10.23%).
- Number of procedures shows no clear pattern (8.73%, 9.62%, 8.83%, 8.98%).

**Insights**

- Longer, more complex stays point to sicker patients, but the differences (about 3 to 5 points) are smaller than those for prior inpatient stays.

## Q3. Does where the patient goes after discharge matter?

View the notebook: [03_admission_discharge.ipynb](notebooks/03_admission_discharge.ipynb)

**Results**

Discharge destination shows the biggest differences in the project. Groups with at least 400 patients:

| Discharge destination | Readmission rate | Patients |
|---|---|---|
| Rehab facility | 26.36% | 1,411 |
| Other inpatient institution | 20.59% | 913 |
| Another short-term hospital | 13.70% | 1,540 |
| Skilled nursing facility | 13.42% | 8,788 |
| Home with home health | 9.54% | 8,290 |
| Home | 6.95% | 44,320 |

Admission type is flat (Emergency 9.18%, Urgent 8.97%, Elective 8.29%), and so is admission source (8.07% to 9.39%).

**Insights**

- Patients sent to rehab are readmitted nearly 4 times as often as patients sent home.
- How the patient arrived (admission type and source) tells us little. Where they go next tells us a lot.

## Q4. Do age, gender and race matter?

View the notebook: [04_Demographics.ipynb](notebooks/04_Demographics.ipynb)

**Results**

- Readmission rises with age, from about 7% for ages 20 to 59 to 10.79% for ages 80 to 90.
- Gender is nearly identical (female 9.04%, male 8.91%).
- Race differences are small among the large groups (Caucasian 9.19%, African American 8.67%). Smaller race groups were too small to compare reliably.

**Insights**

- Age adds a modest amount of risk. Gender and race do not meaningfully separate risk in this data.

## Q5. Does the primary diagnosis matter?

View the notebook: [05_main_diagnosis.ipynb](notebooks/05_main_diagnosis.ipynb)

**Results**

Diagnoses were grouped from ICD-9 codes into 9 categories. Readmission by primary diagnosis:

| Primary diagnosis | Readmission rate |
|---|---|
| Injury | 10.80% |
| Circulatory (21,392 patients, the largest group) | 9.68% |
| Diabetes | 9.12% |
| Neoplasms | 9.06% |
| Genitourinary | 8.98% |
| Other | 8.98% |
| Musculoskeletal | 8.39% |
| Digestive | 8.01% |
| Respiratory | 7.30% |

**Insights**

- Diagnosis creates a spread of about 3.5 points, smaller than prior stays or discharge destination.

## Q6. Do testing and diabetes medication matter?

View the notebook: [06_diabetes_care.ipynb](notebooks/06_diabetes_care.ipynb)

**Results**

- Patients with a medication change were readmitted more often (9.44% versus 8.60%).
- Patients on diabetes medication were readmitted more often (9.42% versus 7.56%).
- Insulin status shows a gradient: no insulin 8.29%, steady 9.26%, dose up 9.84%, dose down 10.54%.
- Patients who had an A1C test were readmitted less often (about 8.4%) than patients not tested (9.11%). Among patients whose main diagnosis is diabetes, the gap is large (7.23% tested versus 10.17% not tested).

**Insights**

- Patients who need more diabetes treatment are higher risk. A1C testing is linked to lower readmission mainly when diabetes is the reason for the stay.

## Q7. Which patients should be prioritized? (Risk Tiers)

View the notebook: [07_prioritized_patients.ipynb](notebooks/07_prioritized_patients.ipynb)

Patients were sorted into three tiers with plain rules based on Q1 to Q6:

- **High:** discharged to rehab, or 2 or more prior inpatient stays
- **Medium:** 1 prior inpatient stay, or discharged to another facility
- **Standard:** everyone else

| Tier | Patients | Share of Patients | Readmission Rate | Share of All Readmissions |
|---|---|---|---|---|
| High | 3,779 | 5.4% | 23.07% | 13.9% |
| Medium | 15,830 | 22.6% | 12.56% | 31.6% |
| Standard | 50,373 | 72.0% | 6.80% | 54.5% |

**Results**

- The High tier is readmitted at 23.07%, about 3.4 times the rate of the Standard tier (6.80%).
- High and Medium together are 28% of patients but hold 45.5% of all readmissions.

**Insights**

- A hospital with limited follow-up capacity can start with the High tier, where about 1 in 4 patients returns.
- More than half of readmissions still occur in the Standard tier, because it is 72% of patients. Tiers help focus effort but will not catch every readmission.

## Q8. Do risk factors stack?

View the notebook: [08_stacked.ipynb](notebooks/08_stacked.ipynb)

Readmission rate by discharge destination and prior inpatient stays:

| Discharge | 0 Stays | 1 Stay | 2+ Stays |
|---|---|---|---|
| Home | 6.2% | 10.9% | 18.3% |
| Home with home health | 8.7% | 10.3% | 21.7% |
| Other facility | 12.5% | 19.8% | 27.0% |
| Rehab | 25.7% | n/a | n/a |

Groups with fewer than 100 patients are not shown.

**Results**

- Within every discharge group, more prior stays means higher readmission. For patients discharged home, the rate roughly triples from 6.2% to 18.3%.
- Rehab patients are at 25.7% even with no prior stays.

**Insights**

- The two factors add up. A patient discharged home with 2 or more prior stays (18.3%) is at higher risk than a patient sent to another facility with no prior stays (12.5%).
- The lowest-risk group (home, no prior stays, 6.2%) and the highest reliable group (other facility, 2 or more stays, 27.0%) differ by more than 4 times.

## Q9. Do the findings hold within age groups?

View the notebook: [09_all_age_groups.ipynb](notebooks/09_all_age_groups.ipynb)

Because older patients are readmitted more often and are more likely to go to facilities, discharge destination and prior stays were checked within three age groups (under 60, 60 to 79, 80 and over). Every group had at least 229 patients.

**Results**

- Rehab discharges were readmitted at 25.8% (under 60), 28.2% (60 to 79) and 23.1% (80 and over), compared with 6.0%, 7.5% and 8.2% for patients discharged home.
- Prior stays showed the same pattern in every age group. The rate rose from 6.3% (0 stays) to 19.7% (2 or more) for patients under 60, and from 9.7% to 24.4% for patients 80 and over.

**Insights**

- Discharge destination and prior stays are not just stand-ins for age. A patient under 60 with 2 or more prior stays (19.7%) is at about twice the risk of an 80-year-old with none (9.7%).
- Age adds a small amount of risk on top of these two factors, so the risk tiers are built on discharge destination and prior stays.

## Factor Ranking

*[Add the ranking result from the final notebook here: which factors separate the lowest and highest readmission groups the most, among groups of 400+ patients.]*

## Dashboard

*[Add Power BI screenshot here]*

## Recommendations

1. Flag patients at discharge using two questions: where is the patient going, and how many overnight stays have they had in the past year.
2. Start follow-up with the High tier (rehab discharges and patients with 2 or more prior stays), where about 1 in 4 patients is readmitted.
3. Extend to the Medium tier if capacity allows, which brings coverage to about 45% of readmissions.
4. Look at diabetes patients with no A1C test, who have a higher readmission rate when diabetes is the main diagnosis.
5. Do not spend effort on admission type, admission source or number of procedures, which show little or no relationship with readmission.

## Limitations

- Tier rules were chosen after looking at this same data, so the rates may be slightly optimistic on new patients.
- The data shows association, not cause. Rehab discharge marks sicker patients; it does not cause readmission.
- The data covers 1999 to 2008 and may not reflect current practice.
- Several groups (some races, rehab patients with prior stays) are too small to compare.

## Conclusions

- About 9% of patients are readmitted within 30 days.
- Prior inpatient stays and discharge destination are the strongest factors, followed by age, length of stay and medication use.
- Admission type, admission source, gender and procedures show little relationship.
- Simple risk tiers separate patients well: 23% readmission in the High tier versus under 7% in the Standard tier.
- Risk factors stack, so patients with both a high-risk discharge and prior stays should be the first to receive follow-up.
- The two main factors hold within every age group, so they are not simply a stand-in for age.

## Files

- `notebooks/`: the cleaning notebook (`diabetic_healthcare_cleaned`) and one notebook per question (01 to 09 answer Q1 to Q9)
- `datasets/`: raw data, `diabetic_clean.csv` (cleaned) and `diabetic_dashboard.csv` (cleaned plus risk tier, age group, prior stays and discharge group columns used by Power BI)
- `dashboard/`: Power BI file and screenshot
