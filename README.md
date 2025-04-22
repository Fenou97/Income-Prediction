# Monthly Income Prediction

## Project Overview

By engaging stakeholders, we clarified that high employee turnover rates are costly, and understanding the salary range employees would expect in order to stay could improve retention. "What salary predicts employee retention based on historical data?"

## Research Question
- What is the relationship between salary and employee retention across different job roles and departments?
- What salary ranges (minimum, maximum) are associated with the highest retention rates for specific job roles or levels?

## Data Sources
The data used "Salary.csv" is from HR web-based desktop system that contains information about all employees, current and past, including their Income and contains 24 variables such as Annual Income Needed (Character), Age (Integer),  Distance From Home (Integer), Education (Integer), Environment Satisfaction (Integer), Job Involvement (Integer), Job Level (Integer), Job Satisfaction (Integer), Number of Companies Worked (Integer), Average Over Time (Integer), Percent Salary Hike (Numeric), Performance Rating (Integer), Relationship Satisfaction (Integer), Stock Option (Integer), Total Working Years (Integer), Training Times Last Year (Integer), Work Lile Balance (Integer), Years At Company (Integer), Years In Current Role (Integer), Years Since Last Promotion (Integer), Years With Current Manager (Integer), Difference From Salary (Character), Current Salary (Character). 

![Data Structure](https://github.com/user-attachments/assets/be344378-3c33-4889-9e20-fd94d8ede59b)

## Tools
- Excel for data Assessment
- R for data manipualtion and data modeling

## Exploratory Data Analysis

- Data Summmary and Data Distribution
![Data Summary](https://github.com/user-attachments/assets/709f68a7-9126-4d83-8eb2-66830b53045d)

The summary statistics has enabled us to know the measures of central tendency (Mean, Median, Mode), measures of dispersion (Range, Variance, Standard Deviation, Interquartile Range) which are foundational for exploring, understanding the data.
  
![Data Visualization](https://github.com/user-attachments/assets/45bd0540-afbf-4360-bb03-58a19ffb1869)

![Data Visualization 2](https://github.com/user-attachments/assets/adc0536f-b646-4e02-be51-4c598bbd197a)

We singled out some variables that are not normally distributed and can affect the model building phase. 


## Data Preparation
- Data Cleaning/Checking for errors
```R
num_duplicates <- sum(duplicated(Salary1))
> print(num_duplicates)
[1] 0 
> num_missing_values <- sum(is.na(Salary1$CurrentSalary))
> print(num_missing_values)
[1] 0
```
The data is free from duplicated values and missing values

- Data Cleaning/ Fix Data Types

From the data structure above, we see that the variables Annual Income Needed, Current Salary and Difference from Salary are stored as character and need to be converted to the appropriate data type for our analysis.
```R
Salary$AnnualIncomeNeeded <- as.numeric(Salary$AnnualIncomeNeeded)
Warning message:
NAs introduced by coercion 
> Salary$DiffFromSalary <- as.numeric(Salary$DiffFromSalary)
Warning message:
NAs introduced by coercion 
> Salary$CurrentSalary <- as.numeric(Salary$CurrentSalary)
Warning message:
NAs introduced by coercion
```
There is an issue concerning the data that is preventing the conversion. So we remove Special Characters (e.g., $ or ,), Trim Whitespace, Handle Empty or Non-Numeric Values and finally convert the variables into numeric.
```R
# Remove Special Characters (e.g., $ or ,):
Salary$AnnualIncomeNeeded <- gsub("[\\$,]", "", Salary$AnnualIncomeNeeded)
Salary$DiffFromSalary <- gsub("[\\$,]", "", Salary$DiffFromSalary)
Salary$CurrentSalary <- gsub("[\\$,]", "", Salary$CurrentSalary)

# Trim White space:
Salary$AnnualIncomeNeeded <- trimws(Salary$AnnualIncomeNeeded)
Salary$DiffFromSalary <- trimws(Salary1$DiffFromSalary)
Salary$CurrentSalary <- trimws(Salary1$CurrentSalary)

# Handle Empty or Non-Numeric Values
Salary$AnnualIncomeNeeded <- ifelse(Salary$AnnualIncomeNeeded == "", NA, Salary$AnnualIncomeNeeded)
Salary$DiffFromSalary <- ifelse(Salary$DiffFromSalary == "", NA, Salary$DiffFromSalary)
Salary$CurrentSalary <- ifelse(Salary$CurrentSalary == "", NA, Salary$CurrentSalary)

# Do the conversion
Salary$AnnualIncomeNeeded <- as.numeric(Salary$AnnualIncomeNeeded)
Salary$DiffFromSalary <- as.numeric(Salary$DiffFromSalary)
Salary$CurrentSalary <- as.numeric(Salary$CurrentSalary)
```
- Data Cleaning/Normality

We used Log Transformation technique to normalize the variable “Diff from Salary” that was first right skewed.
```R
Salary$DiffFromSalary <- log(Salary$DiffFromSalary) 
Salary$DiffFromSalary <- log(Salary$AnnualIncomeNeeded)
```
![Data Normality](https://github.com/user-attachments/assets/f685a13e-3254-4d35-83cc-15a748435f98)

  
  
