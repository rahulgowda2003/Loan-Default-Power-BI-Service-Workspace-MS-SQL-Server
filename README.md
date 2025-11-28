# Loan Default and Financial Insight Report -| Power BI Service | MS SQL Server| power BI Desktop |

A comprehensive analytical dashboard/report exploring loan defaults, borrower demographics, financial behavior, and credit risk using Power BI.

The report is created by using MS SQL Server, Power BI Service and Power BI Desktop.

## Problem Statement

This dashboard provides a multi-page analytical view of borrower profiles and loan performance.
It helps banking and lending institutions understand:

- Loan distribution across purposes, age groups, and education
- Impact of employment type on default rate
- Credit score influence on loan amount
- Year-over-year change in loan and default patterns
- High-risk segments and financial behaviors
- Loan distribution by mortgage status, dependents, and marital status

The dashboard supports risk assessment, customer profiling, portfolio monitoring, and loan policy optimization.


### Steps Followed

- step 1 : Connected Standard mode gateway to Power BI service
- step 2 : Creating workspace named "Dataflow" in Power BI service, connecting Dataflow with SQL server
- step 3 : Importing data to Power BI service through Dataflow workspace and then importing data to Power BI Desktop
- step 4 : Opened power query editor and in view tab, under Data preview section, check "column distribution", "column quality" and "column profile" options.

  By default, column profile will be opened only for 1000 rows, so you will need to select "column profiling based on entire dataset".

  It was observed that in none of the columns errors and no empty values were present.
- step 5 : Creating report by inserting three pages namely "Loan Default Overview", "Application demographic and Financial profile" and "Financial Risk Matrix"
- step 6 : Under Loan Deafult Overview,  Created new columns named "YEAR", "Age group", "Credit score bins" and "Income bracket" using DAX to extract year values from date column

a) Year = YEAR(Loan_default[Loan_Date_DD_MM_YYYY])

b) Age group = 
IF(Loan_default[Age] <=19,"Teen",
IF(Loan_default[Age] <=39,"Adult",
IF(Loan_default[Age] <=59,"Middle Age Adult",
"Senior Citizen")))

c) Credit score bins = 
IF(Loan_default[CreditScore] <=400 ,"Very Low",
IF(Loan_default[CreditScore] <=450 ,"Low",
IF(Loan_default[CreditScore] <= 650 , "Medium",
"High")))

d) Income bracket = 
SWITCH(
    TRUE(),
    Loan_default[Income] <= 30000,"Low Income",
    Loan_default[Income] >= 30000 && Loan_default[Income] <=60000,"Medium Income",
    Loan_default[Income] >=60000,"High")

Created new table named "Measure's Table 1" to create and store DAX measures used in "Loan Default Overview" page. The DAX measures as follows:

1] Avg income EmpolyementType = 
CALCULATE(AVERAGE(Loan_default[Income]),ALLEXCEPT(Loan_default,Loan_default[EmploymentType]))

2] Avg loan AgeGroup = 
AVERAGEX(VALUES(Loan_default[Age group]),
AVERAGE(Loan_default[LoanAmount]))

3] Default rate EmployementType = 
VAR totalrecords = COUNTROWS(ALL(Loan_default))
VAR defaultcases = COUNTROWS(FILTER(Loan_default,'Loan_default'[Default] = TRUE()))
RETURN
CALCULATE(DIVIDE(defaultcases,totalrecords),ALLEXCEPT(Loan_default,Loan_default[EmploymentType]))*100

4] Default rate Year = 
VAR totaloan = CALCULATE(COUNTROWS(Loan_default),ALLEXCEPT(Loan_default,Loan_default[Year]))
VAR default = CALCULATE(COUNTROWS(FILTER(Loan_default,Loan_default[Default] = TRUE())),ALLEXCEPT(Loan_default,Loan_default[Year]))
RETURN
DIVIDE(default,totaloan)*100

5] Default rate Year = 
VAR totaloan = CALCULATE(COUNTROWS(Loan_default),ALLEXCEPT(Loan_default,Loan_default[Year]))
VAR default = CALCULATE(COUNTROWS(FILTER(Loan_default,Loan_default[Default] = TRUE())),ALLEXCEPT(Loan_default,Loan_default[Year]))
RETURN
DIVIDE(default,totaloan)*100

  created five charts as follows:

  a) Loan Amount by Purpose
  
  b) Average Income by Employement Type

  c) Default Rate by Employement Type (%)
  
  d) Average Loan  by Age Group
  
  e) Default Rate by Year (%)

- step 7 : Under Application demographic and Financial profile, created new table named "Measure's Table 2" to create and store DAX measure as follows:

  1] Avg LoanAmount by CreditScore = 
AVERAGEX(FILTER(Loan_default,Loan_default[Credit score bins] = "High"),Loan_default[LoanAmount])

  2] Median by credit score bins = 
MEDIANX(Loan_default,Loan_default[LoanAmount])

  3] Total Loan(credit bin) = 
CALCULATE(SUM(Loan_default[LoanAmount]),Loan_default[Age group] = "Adult",ALLEXCEPT(Loan_default,Loan_default[Age],Loan_default[Age group],Loan_default[CreditScore],Loan_default[Credit score bins]))

  4] Total Loan(education type) = 
COUNTROWS(FILTER(Loan_default,NOT(ISBLANK(Loan_default[LoanID]))))

  5] Total Loan(medium age adults) = 
CALCULATE(SUM(Loan_default[LoanAmount]),Loan_default[Age group] = "Middle Age Adult")

Created five charts namely:

a)Median Loan Amount by Credit Score Category

b)Avg Loan Amount by High Credit Score by Age group and Marital Status

c)Total Loan Taken by Adults by Credit Score

d)Total Loan(Middle Age Adults) by has mortgage/Dependents

e)Total Loan by Education

- step 8 : Under Financial Risk Matrix, created a new table named "Measure's Table 3" to create and store DAX measures as follows:

  1] YOY default loan change = 
DIVIDE(
    CALCULATE(COUNTROWS(FILTER(Loan_default,Loan_default[Default] = TRUE())),
    Loan_default[Year] = YEAR(MAX(Loan_default[Loan_Date_DD_MM_YYYY]))) -
    
    CALCULATE(COUNTROWS(FILTER(Loan_default,Loan_default[Default] = TRUE())),
    Loan_default[Year] = YEAR(MAX(Loan_default[Loan_Date_DD_MM_YYYY])) - 1),


    CALCULATE(COUNTROWS(FILTER(Loan_default,Loan_default[Default] = TRUE())),
    Loan_default[Year] = YEAR(MAX(Loan_default[Loan_Date_DD_MM_YYYY])) - 1),0)*100

  2] YOY loanAmt change = 
VAR currentyear = CALCULATE(
    SUM(Loan_default[LoanAmount]),
    Loan_default[Year] = YEAR(MAX(Loan_default[Loan_Date_DD_MM_YYYY])))
VAR prevyear = CALCULATE(
    SUM(Loan_default[LoanAmount]),
    Loan_default[Year] = YEAR(MAX(Loan_default[Loan_Date_DD_MM_YYYY])) - 1)

RETURN

DIVIDE(currentyear - prevyear,prevyear,0)*100

  3] YTD LoanAmount = 
CALCULATE(SUM(Loan_default[LoanAmount]),
DATESYTD(Loan_default[Loan_Date_DD_MM_YYYY].[Date]),
ALLEXCEPT(Loan_default,Loan_default[Credit score bins],Loan_default[MaritalStatus]))

Created four charts:

a) Year on Year Loan Amount change by Year (%)

b) Year on Year Default Loan Change by Year (%)

c) Year Till Date Loan Amount by Credit score bins and Marital Status

d) Decision Tree

- step 9 : Setting schedule refresh in Power BI service and publishing the report the Dataflow workshop


Report Snapshot (Power BI Desktop)

![Dashboard_upload]()

![Dashboard_upload]()

![Dashboard_upload]()


# Insights

a) Loan Default and Overview

1] Loan Amount by Purpose

Shows total loan volume (in millions) for major loan purposes:

- Purpose	Loan Amount
- Home	6545M
- Business	6522M
- Education	6511M
- Auto	6501M
- Other	6498M

All loan purposes show a similar distribution, with Home loans slightly leading.

2] Average Income by Employment Type

Employment Type	Avg Income

- Full-time	82,890.30
- Part-time	82,389.36
- Self-employed	82,446.71
- Unemployed	82,272.37

Income variation is small across employment types, suggesting a balanced applicant pool.

3] Default Rate by Employment Type (%)

Employment Type	Default Rate

- Unemployed	3.39%
- Part-time	3.01%
- Self-employed	2.86%
- Full-time	2.36%

Full-time applicants have the lowest default rate; unemployed have the highest.

4] Average Loan by Age Group

Age Group	Average Loan

- Adult	127,901
- Middle Age Adult	127,459
- Senior Citizen	127,355
- Teen	126,674

Loan amounts remain consistent across age groups with slight dips for Teens.

5] Default Rate by Year (%)

Year	Default Rate

- 2013	11.62%
- 2014	11.50%
- 2015	11.70%
- 2016	Nil
- 2017	11.50%
- 2018	11.60%

(As shown in the tooltip for 2018)

Default rates hover around 11.5%–11.7%, indicating stable risk over the years.

b) Application Demographic & Financial Profile

1] Median Loan Amount by Credit Score Category

Credit Score	Median Loan

- Low	128,397
- Medium	127,764
- Very Low	127,515
- High	127,149

Higher credit scores correlate with slightly lower loan amounts.

2] Avg Loan by High Credit Score, Age Group & Marital Status

- Loan average values (approx 127K–129K) across:
- Age groups: Adult, Middle Age Adult, Senior Citizen, Teen
- Marital status: Divorced, Married, Single

Loan amounts remain very consistent regardless of family status or age for high-score applicants.

3] Total Loan Taken by Adults by Credit Score

Credit Score	Total Loan

- Medium	4.6bn
- High	4.5bn
- Very Low	2.3bn
- Low	1.1bn

Higher credit groups borrow substantially more.

4] Total Loan (Middle Age Adults) by Mortgage & Dependents

- All combinations show approx 3.1bn:

Has Mortgage	Has Dependents	Loan Amount

- For has Mortgage and has dependents	3.1bn
- For has Mortgage and not having dependents	3.1bn
- For not having Mortgage and has dependents	3.1bn
- For not having Mortgage and not having  dependents 3.1bn

Mortgage & dependent status do not significantly impact loan value for this segment.

5] Total Loan by Education

Education	Loan Amount

- Bachelor's	64,366
- High School	63,903
- Master's	63,541
- PhD	63,537

Loan amounts decrease slightly with higher education levels.

c) Financial Risk Matrix

1] Year-on-Year Loan Amount Change (%)

For Year	and % Change

- 2013	0.00
- 2014	-1.53
- 2015	1.30
- 2016	-0.01
- 2017	-1.08
- 2018	1.73

Loan amounts show fluctuation, with growth spikes in 2015 and 2018.

2] Year-on-Year Default Loan Change (%)

For Year and	% Change

- 2013	0.00
- 2014	-2.57
- 2015	2.70
- 2016	0.82
- 2017	-2.83
- 2018	1.89

Default rate volatility indicates shifting borrower behavior and economic conditions.

3] Year-Till-Date Loan Amount by Credit Score & Marital Status

Flow values include:

- Medium : ~0.64bn, 0.67bn
- High : ~0.65bn, 0.67bn
- Very Low : ~0.32bn, 0.33bn
- Low : ~0.17bn

Categories split by Marital Status: Divorced, Married, Single

High and medium credit borrowers dominate loan volume.

4] Loan Amount by Income Bracket & Employment Type (Tree Visual)

Income Brackets

Bracket	Loan Amount

- High	217,314,09302
- Medium	72,125,26643
- Low	36,329,44627
- Employment Type Breakdown
- Type	Loan Amount
- Full-time	544,426,1752
- Part-time	543,780,5960
- Self-employed	542,759,5051

Higher income groups contribute the majority of total loan volume.

# Key Insights Across All Pages

a) Employment

- Full-time workers have the lowest default rate.
- Unemployed applicants are highest risk.

b) Credit Score

- Medium and High credit groups borrow the most.
- Very-low and low groups contribute to less than half the loan volume.

c) Age Group

- Loan amounts stay relatively stable across age categories.

d) Education

- Bachelor’s applicants borrow slightly more.

e) Risk Trends

- Default and loan YoY values fluctuate, indicating sensitive borrower behavior.

f) Demographics

- Marital status does not strongly impact loan value.
- Mortgage & dependents show similar loan volumes.



  


 
