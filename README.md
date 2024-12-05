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

## 2- Table View

### 1) In Patient DataSet
we create two columns:
1- Age Buckets : Calculate **Patient Age** and put in range , For example "11:20"
 
``` 
Age Buckets =
SWITCH(
	TRUE(),
	'Patients Dataset'[patient_age]<=10 , "0-10",
	'Patients Dataset'[patient_age]<=20 , "11-20",
	'Patients Dataset'[patient_age]<=30 , "21-30",
	'Patients Dataset'[patient_age]<=40 , "31-40",
	'Patients Dataset'[patient_age]<=50 , "41-50",
	'Patients Dataset'[patient_age]<=60 , "51-60",
	'Patients Dataset'[patient_age]<=70 , "61-70",
	"70+"
)
```
2- Age Group : Classification **Patient Age** to (Infancy , Early Childhood , Middle Childhood , Teenager and Adult)
 
``` 
Age Group = 
    VAR _PatientAge = 'Patients Dataset'[patient_age]
    RETURN IF(
        _PatientAge <=2 ,"Infancy",
        IF(
            _PatientAge <=6 , "Early Childhood",
            IF(
                _PatientAge<=12 , "Middle Childhood",
                IF(
                    _PatientAge <=18 , "Teenager",
                    "Adult"
                )
            )
        )
    )
```
### 2) Date DataSet
1- Create Date Column from 1/1/2019 to 31/12/2020
2- From Date column , we calculated other columns (Year , Month , WeekType , WeekDay , MonthNum)

```
Date = 
    ADDCOLUMNS(
        CALENDARAUTO(),
        "Year" , YEAR([Date]),
        "Month" , FORMAT([Date],"mmm"),
        "WeekType",IF(
                    WEEKDAY([Date])=1,
                    "Weekend",
                    IF(WEEKDAY([Date])=7,
                            "Weekend",
                            "Weekday")),
        "WeekDay" ,FORMAT([Date],"ddd"),
        "MonthNum" , MONTH([DATE])
        )
```
### 3) Create CF Max Point (Year)
Calculate number od patients for each year (2019 and 2020)
```
CF Max Point(Year) = VAR _PatientTable = 
       CALCULATETABLE(
                ADDCOLUMNS(
                SUMMARIZE('Date','Date'[Year]),
               "@Total_Patients",[Total Patients]
                ),
               ALLSELECTED()
           )
    RETURN _PatientTable
```
### 4) Create CF Max Point (Month)
Calculate number od patients for each Month
```
CF Max Point(Month) = VAR _PatientTable = 
       CALCULATETABLE(
                ADDCOLUMNS(
                SUMMARIZE('Date','Date'[Month]),
               "@Total_Patients",[Total Patients]
                ),
               ALLSELECTED()
           )
    RETURN _PatientTable
```
### 5) Create Calculation Table : to save all measures in this table , And we will explain each measure later
