# HealthCare-Patients-Emergency

## 1- Data Dictionary

| Field Name | Description | Data Type |Example|
|--|--|--|--|
| date | The date of the patient's record or appointment. | Date | 3/20/2020 |
| patient_id| Unique identifier assigned to each patient. | String |145-39-5406|
|patient_gender  | The gender of the patient. | String | M (Male) / F (Female) |
| patient_age | Age of the patient at the time of the record. | Integer |69|
| patient_sat_score | Satisfaction score (out of 10) provided by the patient, if available. | Integer / Null | 10 |
| full_name |The patient's name. Placeholder or anonymized for privacy.| String | H Glasspool |
| patient_race | The race or ethnicity of the patient. | String | White |
|patient_admin_flag| Indicates if the patient requires administrative attention (TRUE/FALSE). |  Boolean| FALSE |
|patient_waittime| The time (in minutes) the patient waited for their appointment. | Integer | 39 |
| department_referral | The department the patient was referred to, if any. | String | General Practice / None |
| moment | Indicates whether the appointment occurred in the morning (AM) or afternoon (PM). | String | AM |
