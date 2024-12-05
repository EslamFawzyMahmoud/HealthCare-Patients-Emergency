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


## Report View
1- we calculated the total patients using (Total Patients measure) using [Card Visual]
```
Total Patients = 
    COUNTROWS('Patients Dataset')
```

2- Calculated Administrative Schedule Percentage Using (% Administrative Schedule) using [Card Visual]
```
% Administrative Schedule = 
    DIVIDE(
        COUNTROWS(
            FILTER('Patients Dataset',
                    'Patients Dataset'[patient_admin_flag] = TRUE()
                )
            ),
            [Total Patients]
        )
```

3- Calculated Non Administrative Schedule Percentage Using (% NonAdministrative Schedule) using [Card Visual]
```
% NonAdministrative Schedule = 
    DIVIDE(
        COUNTROWS(
            FILTER('Patients Dataset',
                    'Patients Dataset'[patient_admin_flag] = FALSE()
                )
            ),
            [Total Patients]
        )
```
4- Calculated the Average Satisfaction score by the patient using (Average Satisfaction Score)
```
Average Satisfaction Score = 
    CALCULATE(
        AVERAGE('Patients Dataset'[patient_sat_score]),
        'Patients Dataset'[patient_sat_score] <> BLANK()
    )
```

5- Calculate the Percentage of patients didn't rate the Satisfaction Score using (% No Rating)
```
% No Rating = 
    VAR _NoRating = 
        CALCULATE(
            [Total Patients],
            'Patients Dataset'[patient_sat_score] = BLANK()
        )
    RETURN DIVIDE(_NoRating,[Total Patients])
```

6- Calculate the average time the patients waited for their appointment (Average Wait time)
```
Average Wait time = 
    AVERAGE('Patients Dataset'[patient_waittime])
```

7- Calculate the percentage of patients who didn't refer for any department (% Un Referred Patients)

```
% Un Referred Patients = 
    VAR _FilterPatient = 
        CALCULATE(
            [Total Patients],
            'Patients Dataset'[department_referral]="none"
        )
    RETURN 
        DIVIDE(_FilterPatient,[Total Patients])
```

8- Calculate the percentage of patients who referred to department using (% Referred Patients)
```
% Referred Patients = 
    VAR _FilterPatient = 
        CALCULATE(
            [Total Patients],
            'Patients Dataset'[department_referral]<>"none"
        )
    RETURN 
        DIVIDE(_FilterPatient,[Total Patients])
```

9- Calculate the percentage of gender (Male , Female , Unknown) using (% Male Visit ,% Female Visit ,% UnKnown Visit)
```
% Male Visit = 
    DIVIDE(
        CALCULATE(
            [Total Patients],'Patients Dataset'[patient_gender]= "M"
        ),
        [Total Patients]
    )
```

```
% Female Visit = 
    DIVIDE(
        CALCULATE(
            [Total Patients],'Patients Dataset'[patient_gender]= "F"
        ),
        [Total Patients]
    )
```
```
% UnKnown Visit = 
    DIVIDE(
        CALCULATE(
            [Total Patients],'Patients Dataset'[patient_gender]= "NC"
        ),
        [Total Patients]
    )
```

10- Create Moment slicer to filter the report if I want to show the number (AM or PM)

11- Create Column Chart to visual the (Total Patients) and {WeekType} , and the chart show that the most of patients comes in weekday ***6574 patients** Unlike weekend **2642 patients**.

12- Create clustered bar chart to visual the (Total Patients) by {Age Group}, and the chart shows the most of total patients is **Adult** and lowest number is **Infancy**.

13- Create Line chart to visual the (Total Patients) by year 2019 and 2020. In *2019* the total visits is **4338 visits** and *2020* the total visits is **4878 visits**.

14- Create stacked bar chart to visual the total patients by department referral. The chart shows the **None** is the most visits and **Renal** the Lowest visits.

15- Create Area chart to visual th total patients by Month. The **(Aug)** is the month will the most patients visited by **1024** .  The **(Feb)** is the month will the Lowest patients visited by **431**.

16- Create Matrix to visual the {Patient race} in Rows and {Age Buckets} in columns using **(Average Wait time)** and **(Average Satisfaction Score)** parameter and create slicer to choose (Avg. Satisfaction OR Avg. Wait time).
	- If you choose Avg. Wait time the caption will change (HeatMap Caption) to match the parameter
```
HeatMap Caption = 
    VAR _SelectedMeasure = 
        SELECTEDVALUE(Parameter[Parameter Order])
    RETURN 
        IF(_SelectedMeasure = 0,
            "The Darkest GREEN on the Scale denotes LOW WAIT TIME on the Age-Group",
            "Patient are most STIFIED when the Scale shows the Darkest GREEN on the Age-Group")
```

 

















