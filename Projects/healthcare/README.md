# Diabetic Patient 30-Day Readmission Analysis

**Answer in brief:** Follow up first with patients discharged to rehab or with 2 or more prior inpatient stays (the High tier). That group is 5% of patients and is readmitted at 23%, compared with under 7% in the Standard tier. The two factors stack, and both hold within every age group.

## Overview

Hospitals can face financial penalties for some preventable readmissions, and follow-up care can help prevent many of them. This project analyzes encounters for 69,982 diabetic patients to find which factors are linked to returning within 30 days, and which patients a hospital should follow up with first.

**Business question:** Which diabetic patient and hospital-encounter factors are linked to 30-day readmission, and which patients should a hospital prioritize for follow-up to prevent avoidable readmissions?

## Key Findings

- About 9% of patients (8.98%) are readmitted within 30 days.
- **Prior inpatient stays and discharge destination are the strongest factors.** Readmission rises from 8.1% with no prior stays to 26.4% with 3 or more, and from 6.9% for patients sent home to 26.4% for patients sent to rehab.
- Age, length of stay, diagnosis and diabetes medication add smaller differences (about 3 to 5 points).
- Admission type, admission source, gender and number of procedures show little relationship.
- **Simple risk tiers work:** 23.1% readmission in the High tier versus 6.8% in the Standard tier.
- **Risk factors stack,** so patients with both a high-risk discharge and prior stays should be followed up first.
- Both main factors hold within every age group, so they are not just a stand-in for age.

## Dataset

- **Source:** Diabetes 130-US Hospitals dataset (encounters from 130 US hospitals, 1999 to 2008)
- **Raw file:** 101,766 rows (one per hospital visit) and 50 columns, covering 71,518 unique patients
- **After cleaning:** 69,982 patients (each patient's first visit) and 50 columns
- **Target:** `readmitted_30d` (1 if the patient returned within 30 days, 0 otherwise)
- **Tools:** Python (pandas) in Jupyter Notebook, one notebook per question; Power BI for the dashboard; GitHub

## Data Cleaning

Full notebook: [diabetic_healthcare_cleaned.ipynb](notebooks/diabetic_healthcare_cleaned.ipynb)

**What the raw data looked like**

- 13 columns are numbers and 37 are text. Three columns hold numeric codes (`admission_type_id`, `discharge_disposition_id`, `admission_source_id`), which are translated with `IDS_mapping.csv`.
- Missing values: weight 96.9%, max_glu_serum 94.7%, A1Cresult 83.3%, medical_specialty 49.1%, payer_code 39.6%, race 2.2%, diag_3 1.4%, diag_2 0.4%, diag_1 21 rows.
- The original `readmitted` column has three values: NO (54,864), >30 (35,545) and <30 (11,357), a raw 30-day rate of 11.16%.
- Some count columns have extreme values with a median of 0 (prior inpatient stays up to 21, ER visits up to 76, outpatient visits up to 42). These are grouped as "3 or more" or "2 or more" in the analysis so a few extreme patients do not distort results.

**Cleaning steps**

| Step | Result | Why |
|---|---|---|
| Dropped `weight` and `payer_code` | 50 to 48 columns | Missing for 96.9% and 39.6% of patients |
| Labeled blanks instead of removing rows | No rows lost | A blank `A1Cresult` or `max_glu_serum` means the test was not ordered ("Not Tested"). Blank `medical_specialty` became "Not Specified", `race` "Unknown", `diag_2` and `diag_3` "None". Dropping these rows would have removed most of the data |
| Removed patients with no primary diagnosis | 101,766 to 101,745 rows | 21 patients; no blanks remained |
| Removed patients who died or went to hospice (discharge codes 11, 13, 14, 19, 20, 21) | 101,745 to 99,323 rows | 2,422 patients who cannot be readmitted |
| Kept only each patient's first visit | 99,323 to 69,982 rows | Removed 29,341 repeat visits so no patient is counted twice |
| Created `readmitted_30d` | 6,283 of 69,982 readmitted (8.98%) | The target for every question below |
| Grouped `diag_1` into 9 categories | 694 ICD-9 codes to 9 groups | Used the grouping from the original study. Circulatory is the largest group (21,392 patients) |

The 48 remaining columns plus `readmitted_30d` and the diagnosis category make the final 50.

**Why it matters:** keeping only first visits and removing patients who died lowers the rate from 11.16% to 8.98%, so every number below describes a patient's first visit in this data.

## Q1. Are prior hospital visits linked to readmission?

Notebook: [01_prior_visits.ipynb](notebooks/01_prior_visits.ipynb)

**Answer: yes, prior inpatient stays are the strongest single warning sign.**

- Prior overnight (inpatient) stays: 8.13% with 0, 12.88% with 1, 18.52% with 2, 26.37% with 3 or more. That is over 3 times the rate of patients with none.
- Prior ER visits: 8.74% with 0, 11.23% with 1, 14.45% with 2 or more.
- Prior outpatient visits matter less: 8.80% with 0 versus about 10% with 1 or more.

## Q2. Does the length and complexity of the stay matter?

Notebook: [02_hospital_dtay.ipynb](notebooks/02_hospital_dtay.ipynb)

**Answer: somewhat. Longer, more complex stays point to sicker patients, but the gaps are smaller than for prior stays.**

- Days in hospital: 7.16% for 1 to 2 days, 8.48% for 3 to 4, 10.55% for 5 to 7, 11.79% for 8 to 14.
- Number of diagnoses (6.19% to 10.05%), medications (7.30% to 10.46%) and lab tests (7.63% to 10.23%) show the same upward pattern.
- Number of procedures shows no clear pattern (8.73%, 9.62%, 8.83%, 8.98%).

## Q3. Does where the patient goes after discharge matter?

Notebook: [03_admission_discharge.ipynb](notebooks/03_admission_discharge.ipynb)

**Answer: yes, this shows the biggest differences in the project. Patients sent to rehab are readmitted nearly 4 times as often as patients sent home.**

| Discharge destination (groups of 400+ patients) | Readmission rate | Patients |
|---|---|---|
| Rehab facility | 26.36% | 1,411 |
| Other inpatient institution | 20.59% | 913 |
| Another short-term hospital | 13.70% | 1,540 |
| Skilled nursing facility | 13.42% | 8,788 |
| Home with home health | 9.54% | 8,290 |
| Home | 6.95% | 44,320 |

How the patient arrived tells us little: admission type (Emergency 9.18%, Urgent 8.97%, Elective 8.29%) and admission source (8.07% to 9.39%) are both flat. Where they go next tells us a lot.

## Q4. Do age, gender and race matter?

Notebook: [04_Demographics.ipynb](notebooks/04_Demographics.ipynb)

**Answer: age adds a modest amount of risk. Gender and race do not meaningfully separate risk.**

- Age: from about 7% for ages 20 to 59 to 10.79% for ages 80 to 90.
- Gender is nearly identical (female 9.04%, male 8.91%).
- Race differences are small among large groups (Caucasian 9.19%, African American 8.67%). Smaller race groups were too small to compare reliably.

## Q5. Does the primary diagnosis matter?

Notebook: [05_main_diagnosis.ipynb](notebooks/05_main_diagnosis.ipynb)

**Answer: a little. Diagnosis creates a spread of about 3.5 points, smaller than prior stays or discharge destination.**

| Primary diagnosis | Readmission rate |
|---|---|
| Injury | 10.80% |
| Circulatory (largest group, 21,392 patients) | 9.68% |
| Diabetes | 9.12% |
| Neoplasms | 9.06% |
| Genitourinary | 8.98% |
| Other | 8.98% |
| Musculoskeletal | 8.39% |
| Digestive | 8.01% |
| Respiratory | 7.30% |

## Q6. Do testing and diabetes medication matter?

Notebook: [06_diabetes_care.ipynb](notebooks/06_diabetes_care.ipynb)

**Answer: yes. Patients who need more diabetes treatment are higher risk, and A1C testing is linked to lower readmission mainly when diabetes is the reason for the stay.**

- Medication change: 9.44% versus 8.60% with no change.
- On diabetes medication: 9.42% versus 7.56% not on medication.
- Insulin: no insulin 8.29%, steady 9.26%, dose up 9.84%, dose down 10.54%.
- A1C test: about 8.4% when tested versus 9.11% when not tested. For patients admitted mainly for diabetes the gap is large: 7.23% tested versus 10.17% not tested.

## Q7. Which patients should be prioritized? (Risk Tiers)

Notebook: [07_prioritized_patients.ipynb](notebooks/07_prioritized_patients.ipynb)

Plain rules based on Q1 to Q6:

- **High:** discharged to rehab, or 2 or more prior inpatient stays
- **Medium:** 1 prior inpatient stay, or discharged to another facility
- **Standard:** everyone else

| Tier | Patients | Share of Patients | Readmission Rate | Share of All Readmissions |
|---|---|---|---|---|
| High | 3,779 | 5.4% | 23.07% | 13.9% |
| Medium | 15,830 | 22.6% | 12.56% | 31.6% |
| Standard | 50,373 | 72.0% | 6.80% | 54.5% |

**Answer: the High tier is readmitted at about 3.4 times the rate of the Standard tier, and about 1 in 4 High-tier patients returns.**

- High and Medium together are 28% of patients but hold 45.5% of all readmissions.
- More than half of readmissions still occur in the Standard tier because it is 72% of patients. Tiers focus effort but will not catch every readmission.

## Q8. Do risk factors stack?

Notebook: [08_stacked.ipynb](notebooks/08_stacked.ipynb)

**Answer: yes. Within every discharge group, more prior stays means higher readmission.**

| Discharge | 0 Stays | 1 Stay | 2+ Stays |
|---|---|---|---|
| Home | 6.2% | 10.9% | 18.3% |
| Home with home health | 8.7% | 10.3% | 21.7% |
| Other facility | 12.5% | 19.8% | 27.0% |
| Rehab | 25.7% | n/a | n/a |

Groups with fewer than 100 patients are not shown.

- For patients sent home, the rate roughly triples from 6.2% to 18.3%.
- A patient sent home with 2 or more prior stays (18.3%) is at higher risk than a patient sent to another facility with no prior stays (12.5%).
- Rehab patients are at 25.7% even with no prior stays.
- The lowest-risk group (home, no prior stays, 6.2%) and the highest reliable group (other facility, 2 or more stays, 27.0%) differ by more than 4 times.

## Q9. Do the findings hold within age groups?

Notebook: [09_all_age_groups.ipynb](notebooks/09_all_age_groups.ipynb)

Older patients are readmitted more often and are more likely to go to facilities, so discharge destination and prior stays were checked within three age groups (under 60, 60 to 79, 80 and over). Every group had at least 229 patients.

**Answer: yes, they hold. The two main factors are not just a stand-in for age.**

- Rehab discharges were readmitted at 25.8% (under 60), 28.2% (60 to 79) and 23.1% (80 and over), compared with 6.0%, 7.5% and 8.2% for patients sent home.
- Prior stays show the same pattern in every age group: the rate rose from 6.3% (0 stays) to 19.7% (2 or more) for patients under 60, and from 9.7% to 24.4% for patients 80 and over.
- A patient under 60 with 2 or more prior stays (19.7%) is at about twice the risk of an 80-year-old with none (9.7%). Age adds only a small amount on top, so the tiers are built on discharge destination and prior stays.

## Factor Ranking

*[Add the ranking result from the final notebook here: which factors separate the lowest and highest readmission groups the most, among groups of 400+ patients.]*

## Dashboard

*[Add Power BI screenshot here]*

## Recommendations

1. **Screen at discharge with two questions:** where is the patient going, and how many overnight stays have they had in the past year? Both are known at discharge, so no new data is needed.
2. **Start with the High tier** (rehab discharges or 2 or more prior inpatient stays). It is about 5% of patients and is readmitted at 23%, roughly 2.6 times the overall rate. Give this group the most intensive follow-up, such as a phone call within a few days of discharge and a scheduled appointment.
3. **Extend to the Medium tier if capacity allows.** High and Medium together are 28% of patients and hold 45.5% of all readmissions, so this is the practical target for a hospital that can reach about 1 in 4 patients.
4. **Do not expect the tiers to catch everything.** The High tier holds only 13.9% of readmissions, and over half occur in the Standard tier. Pair the tiers with a general discharge process for everyone.
5. **Test routine A1C testing for patients admitted mainly for diabetes.** Readmission was 7.23% with an A1C test versus 10.17% without. This is an association, so a small trial is needed to see whether testing actually reduces readmissions.
6. **Do not use admission type, admission source, gender or number of procedures as screening criteria.** They show little or no relationship with readmission.
7. **Pilot before rolling out.** This data is from 1999 to 2008, so check the tiers on current patients and compare readmission rates for patients who receive follow-up with those who do not.

## Limitations

- Tier rules were chosen after looking at this same data, so the rates may be slightly optimistic on new patients.
- The data shows association, not cause. Rehab discharge marks sicker patients; it does not cause readmission.
- The data covers 1999 to 2008 and may not reflect current practice.
- Several groups (some races, rehab patients with prior stays) are too small to compare.

## Files

- `notebooks/`: the cleaning notebook (`diabetic_healthcare_cleaned`) and one notebook per question (01 to 09 answer Q1 to Q9)
- `datasets/`: raw data, `diabetic_clean.csv` (cleaned) and `diabetic_dashboard.csv` (cleaned plus risk tier, age group, prior stays and discharge group columns used by Power BI)
- `dashboard/`: Power BI file and screenshot
