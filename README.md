# Annual Income Prediction

## Table of contents

- [Project Overview](#project-overview)
- [Research Questions](#research-questions)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Preparation](#data-preparation)
- [Model Buidling: Linear Regression](#model-buidling-:linear-regression)
- [Model Evaluation](#model-evaluation)
- [Limitations](#limitations)
  
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

- ### Data Summmary and Data Distribution
![Data Summary](https://github.com/user-attachments/assets/709f68a7-9126-4d83-8eb2-66830b53045d)

The summary statistics has enabled us to know the measures of central tendency (Mean, Median, Mode), measures of dispersion (Range, Variance, Standard Deviation, Interquartile Range) which are foundational for exploring, understanding the data.
  
![Data Visualization](https://github.com/user-attachments/assets/45bd0540-afbf-4360-bb03-58a19ffb1869)

![Data Visualization 2](https://github.com/user-attachments/assets/adc0536f-b646-4e02-be51-4c598bbd197a)

We singled out some variables that are not normally distributed and can affect the model building phase. 

- ### Correlation Analysis
```R
install.packages("corrplot")
library(corrplot)
corr_matrix <- cor(Salary)
corrplot(corr_matrix, method = "color", type = "upper", tl.col = "black")
```
![Correlation Matrix](https://github.com/user-attachments/assets/246c8734-8f69-42e2-9621-12ca8cf1fd56)

```R
install.packages("ggplot2")
library(ggplot2)
ggplot(Salary1, aes(x = Salary1$AnnualIncomeNeeded, y = Salary1$CurrentSalary)) +
  geom_point() +
  geom_smooth(method = "lm", col = "blue") +
  labs(title = "Scatterplot with Trendline")
```
![Correlation 1](https://github.com/user-attachments/assets/4c59aa12-5c68-4fc6-914f-4edba5674706)
![Correlation 2](https://github.com/user-attachments/assets/70e29231-bda9-4098-8d92-fe91cdb21d27)
![Correlation 3](https://github.com/user-attachments/assets/a124f2bc-f771-41d0-b5aa-7d1dabb5bf30)

Through correlation analysis, we identified variables that are highly correlated to avoid multicollinearity in the model. For instance, the variables Years at the company and Years Current Role are  highly correlated. So we must include one of them in the model to avoid multicollinearity. On the other hand, the correlation analysis has helped identifying variables that are correlated with the target variable (Annual Income Needed). For example, there is a positive correlation between  Annual Income Needed (Target variable) and Current Salary. Variables highly correlated with the target variable are likely to be good predictors. Identifying them will help prioritize which features to include in the model.

## Data Preparation
- ### Data Cleaning/Checking for errors
```R
num_duplicates <- sum(duplicated(Salary1))
> print(num_duplicates)
[1] 0 
> num_missing_values <- sum(is.na(Salary1$CurrentSalary))
> print(num_missing_values)
[1] 0
```
The data is free from duplicated values and missing values

- ### Data Cleaning/ Fix Data Types

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
- ### Data Cleaning/Normality

We used Log Transformation technique to normalize the variable “Diff from Salary” that was first right skewed.
```R
Salary$DiffFromSalary <- log(Salary$DiffFromSalary) 
Salary$DiffFromSalary <- log(Salary$AnnualIncomeNeeded)
```
![Data Normality](https://github.com/user-attachments/assets/f685a13e-3254-4d35-83cc-15a748435f98)

## Model Buidling: Linear Regression

Linear regression, which is supervised machine learning algorithm is used for predicting the desire income (continuous dependent variable) based on one or more independent variables (predictors) in the data set.

- ### First Preliminary Model: with all the variables in the data set

```R
#Build the model
model <- lm(AnnualIncomeNeeded ~., data = Salary)
summary(model)

#Model visualization
install.packages("ggplot2")
library(ggplot2)
#Residuals vs. Fitted Values Plot
plot(model$fitted.values, resid(model), 
     main = "Residuals vs Fitted", 
     xlab = "Fitted Values", ylab = "Residuals",
     pch = 19, col = "red")
abline(h = 0, col = "blue", lwd = 2)

#QQ Plot (Normality of Residuals)
qqnorm(resid(model), main = "QQ Plot of Residuals")
qqline(resid(model), col = "blue", lwd = 2)
```
![First Model with all variables](https://github.com/user-attachments/assets/035a77cf-e738-4bb8-a6a9-d2b3490cfc6f)

![First Model Evaluation](https://github.com/user-attachments/assets/077ede83-0fb3-481c-8028-2242ec700be5)

The model outputs are red flag for biases and non-linearity in the dataset. The results of the model reveals that there are too many predictors in the dataset, what is true. With too many features, the data points become sparse making it harder to find meaningful relationships. The model appears to have an extremely high goodness of fit, but it is likely overfitting . Residual Standard Error (RSE: 7.717e-11) is very close to zero, meaning the residuals (errors) are extremely small. R² of 1 means the model explains 100% of the variance in the dependent variable (too good to be true). A huge F-statistic (1.648e+31) with an extremely small p-value (<2.2e-16) suggests that at least one of the predictors is statistically significant. 

- ### Applying Variance Inflation Factor to remove irrelevant factors
```R
install.packages("car")
library(car)
vif_values <- vif(model)
print(vif_values)
```
![Variance Indicator factor](https://github.com/user-attachments/assets/55423de8-5408-4bd7-abd9-a52c67e4dc0d)

We have typically removed features with high VIF (Variance Inflation Factor), as these indicate multicollinearity. A common threshold to flag high multicollinearity is a VIF greater than 5 or 10. PercentSalaryHike: VIF = 7.25 (indicates potential multicollinearity), TotalWorkingYears: VIF = 4.82 (moderate, but could be considered for further inspection), YearsAtCompany: VIF = 4.48 (moderate), YearsInCurrentRole: VIF = 2.68 (moderate, but not concerning), YearsWithCurrManager: VIF = 2.77 (moderate), DiffFromSalary: VIF = 12.49 (high and suggests significant multicollinearity), CurrentSalary: VIF = 9.62 (high and suggests significant multicollinearity). By eliminating irrelevant features, the model is likely to become simpler and less prone to overfitting, improving generalization to unseen data. Fewer predictors can lead to better predictive performance.
  
- ### 2nd Model with relevant features
```R
#Splitting the data into training data and testing data
install.packages("caret")
library(caret)
trainIndex <- createDataPartition(Salary$AnnualIncomeNeeded, p = 0.75, list = FALSE)
trainData <- Salary[trainIndex, ]
testData <- Salary[-trainIndex, ]
View(trainData)
View(testData)

#Building the model
model1<- lm(AnnualIncomeNeeded ~JobInvolvement+JobSatisfaction+PerformanceRating+RelationshipSatisfaction+WorkLifeBalance+ YearsAtCompany+
              YearsWithCurrManager+CurrentSalary  , data = trainData)
summary(model1)

#Visualize the model1
qqnorm(resid(model1), main = "QQ Plot of Residuals")
qqline(resid(model1), col = "blue", lwd = 2)
```
![Model1](https://github.com/user-attachments/assets/b018fada-6768-4adf-b16d-f13db740f3f5)

![Model 1 Visualization](https://github.com/user-attachments/assets/d914b97a-38d7-42f1-b610-f6ae9ece1436)

CurrentSalary is by far the single biggest driver of “income needed.” A one‑dollar increase in current salary is associated with about $1.22 more “needed” income. PerformanceRating also has a large and significant effect. Other satisfaction/involvement metrics (job involvement, job satisfaction, relationship satisfaction, work‑life balance) do not appear to have a predictive power on the target variable.
THe Q-Q plot shows reasonable normality with minor tail deviations, which is acceptable and the model looks good to proceed with. However, CurrentSalary is highly correlated with the target variable (Annual income Needed) revealing a potential data leakage. R² = 0.9877 is a sign that the model may be overfitting, performing unrealistically well during training, but poorly on unseen data. 

- ### Target Transformation

It seems AnnualIncomeNeeded might be tightly linked to CurrentSalary. If the response is self-reported (a survey question like "What do you think your income should be?"),people seem to base their answer directly on what they currently earn. The variable "DiffFromSalary" reveals the gap. DiffFromSalary = IncomeGap= AnnualIncomeNeeded - CurrentSalary. We will model the DiffFromSalary instead of AnnualIncomeNeeded.
```R
trainData$IncomeGap <- trainData$AnnualIncomeNeeded - trainData$CurrentSalary
 ModelZ<- lm(IncomeGap ~ JobInvolvement + JobSatisfaction+PerformanceRating+RelationshipSatisfaction+WorkLifeBalance+ YearsAtCompany+
               YearsWithCurrManager, data = trainData)
 summary(ModelZ)
```
![Income Gap Model](https://github.com/user-attachments/assets/4c26ae5c-1ccb-47ab-a9b9-2d1af44898d9)

R-squared: 0.1449: predictors explain ~14.5% of the variance in the income gap. That’s much better than before (0.0079), and shows there’s real signal in this model.

F-statistic p-value < 2.2e-16: The model is statistically significant overall.

PerformanceRating (Estimate: 9856.20, p < 2e-16): Employees with higher performance ratings report wanting ~$9,856 more in income, on average, all else being equal. This is strong and highly significant.

Employees who perform better feel they deserve significantly more income compared to what they currently earn .The other variables (satisfaction, involvement) might affect how people feel about their job, but they don’t seem to translate into higher income expectations, at least not directly.

- ### Interactions with Other Variables

Income Gap, Performance Rating, Job Satisfaction

```R
ggplot(trainData, aes(x = factor(PerformanceRating), y = IncomeGap)) +
   geom_boxplot(aes(fill = factor(JobSatisfaction)), alpha = 0.6) +
   labs(
     title = "Income Gap by Performance Rating and Job Satisfaction",
     x = "Performance Rating",
     y = "Income Gap ($)",
     fill = "Job Satisfaction"
   ) +
   theme_minimal()
```
![Interaction 1](https://github.com/user-attachments/assets/9156fd81-fd75-4476-bbd3-6f7c5dca8f3b)

For both Performance Ratings, individuals with Job Satisfaction 1 and 2 tend to have slightly higher or similar medians compared to those with Satisfaction 3 and 4. This might hint that income gap doesn’t decrease much even if people are more satisfied. Performance seems to have a stronger effect.

Income Gap, Performance Rating, Job Involvement  

```R
ggplot(trainData, aes(x = factor(PerformanceRating), y = IncomeGap)) +
   geom_boxplot(aes(fill = factor(JobInvolvement)), alpha = 0.6) +
   labs(
     title = "Income Gap by Performance Rating and Job Involvement",
     x = "Performance Rating",
     y = "Income Gap ($)",
     fill = "Job Involvement"
   ) +
   theme_minimal()
```

![Interraction 2](https://github.com/user-attachments/assets/23f6bb50-a648-490a-befa-70bbf7ede1b3)

At Performance Rating 3, the differences in Income Gap across Job Involvement levels are small (medians are pretty close). But at Performance Rating 4, higher Job Involvement (3 and 4) tends to correlate with a larger Income Gap, hinting that involvement might be rewarded more when performance is already high.

## Model Evaluation

- ###  Predict the Income Gap and Reconstruct the Predicted Income

```R
# Predict the Income Gap
 testData$PredictedGap <- predict(ModelZ, newdata = testData)
# Reconstruct the Predicted Income
 testData$PredictedAnnualIncome <- testData$CurrentSalary + testData$PredictedGap
```
- ###  Evaluation Metrics: RMSE (Root Mean Squared Error) and MAE (Mean Absolute Error)
```R
# Evaluation Metrics
 install.packages("caret")  
 library(caret)
 RMSE(testData$AnnualIncomeNeeded, testData$PredictedAnnualIncome)
 MAE(testData$AnnualIncomeNeeded, testData$PredictedAnnualIncome)
 
 # visualization
 ggplot(testData, aes(x = AnnualIncomeNeeded, y = PredictedAnnualIncome)) +
   geom_point(alpha = 0.4, color = "steelblue") +
   geom_abline(slope = 1, intercept = 0, color = "red", linetype = "dashed") +
   labs(
     title = "Predicted vs Actual Annual Income Needed",
     x = "Actual",
     y = "Predicted"
   ) +
   theme_minimal()
```
![Metrics](https://github.com/user-attachments/assets/43761bc2-9791-4b40-9cc8-5a937d7cf1cd)

The average error (~$6.9K–$8.4K) is relatively small compared to the IQR of ~$72.8K and the full range of ~$145K. This suggests that the model is capturing general trends in how income is perceived. RMSE (8403.822) is 6.12% of the median (137,280) and 6.15% of the mean (136,593). MAE (6936.483) is 5.05% of the median (137,280) and 5.08% of the mean (136,593).

![Evaluation visual](https://github.com/user-attachments/assets/bfb50a3a-49f4-4c0c-bc6a-3fb484b48ad6)

The dots are clustered tightly around the red dashed line, suggesting that the model’s predictions are very accurate. The graph shows that the model does a great job predicting annual income needed, predictions closely match actual values, with only small errors. 

## Limitations
Model shows solid average performance (5–6% error), but real-world deployment will demand a guard against bias, tail mis-predictions. It will be better to try a more powerful model (like a random forest or gradient boosting) to see if we can beat that ~$7K error.
