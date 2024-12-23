
# linear regression
#linear-regression
Lets assume we are talking bout linear regression for now. That means we want to learn a function $f$ for a dataset $X,y$  such that. 
$$f(X) = y$$
$$XW^T + b = y$$


Typically we solve for the vector W by gradient descent. 

Linear Regression assumes the following is true for dataset $X$:

1. **Linearity**: The relationship between the independent variables and the dependent variable is linear. This means that the change in the dependent variable is proportional to the change in the independent variables.
2. There is no multicolliearity between the features - 
3. The errors are ***iid***, ie independent and identically distributed - 
4. The errors are distributed normally. This assumption is necessary for performing valid hypothesis tests and constructing confidence intervals.
5. **Homoscedasticity**: The variance of the errors is constant across all levels of the independent variables. This means that the spread of residuals should be the same for all predicted values.

see them explained by chatgpt [here](https://chatgpt.com/share/f1ed425d-9ed9-4cae-befa-69832aa009c2)


---
# Statistical Tests in Linear Regression
#statistical-tests

After fitting a linear regression model, it's important to assess the model and its coefficients to ensure they are statistically significant and to understand their contributions to the model. Two common tests for this purpose are t-tests for individual coefficients and the F-test for overall model significance. Additionally, we'll discuss some other diagnostic tests and checks you should perform.

### 1. t-tests for Individual Coefficients
#t-test
#### Purpose
The t-test for individual coefficients evaluates whether each predictor variable (independent variable) in the regression model significantly contributes to the prediction of the dependent variable.
#### Hypotheses
- **Null Hypothesis (H₀)**: The coefficient of the predictor variable is zero (\(\beta_i = 0\)).
- **Alternative Hypothesis (H₁)**: The coefficient of the predictor variable is not zero (\(\beta_i \neq 0\)).

#### Test Statistic
The test statistic for the t-test is calculated as:

$$[ t = \frac{\hat{\beta}_i}{\text{SE}(\hat{\beta}_i)}]$$

where $(\hat{\beta}_i)$ is the estimated coefficient and $(\text{SE}(\hat{\beta}_i))$ is the standard error of the estimated coefficient.

#### Interpretation
- A high absolute value of the t-statistic (or a low p-value) indicates that the predictor variable significantly contributes to the model.
- Typically, a p-value less than 0.05 is considered statistically significant.

### 2. F-test for Overall Model Significance

#### Purpose
The F-test evaluates whether the regression model provides a better fit to the data than a model with no predictors (intercept only model).

#### Hypotheses
- **Null Hypothesis (H₀)**: All regression coefficients are zero (\(\beta_1 = \beta_2 = \ldots = \beta_k = 0\)).
- **Alternative Hypothesis (H₁)**: At least one of the regression coefficients is not zero.

#### Test Statistic
The test statistic for the F-test is calculated as:

$$[ F = \frac{\text{MSR}}{\text{MSE}} = \frac{\frac{\text{SSR}}{k}}{\frac{\text{SSE}}{n-k-1}} ]$$

where:
- \(\text{MSR}\) is the mean square regression (sum of squares for regression divided by the number of predictors).
- \(\text{MSE}\) is the mean square error (sum of squares for error divided by the degrees of freedom).
- \(\text{SSR}\) is the sum of squares for regression.
- \(\text{SSE}\) is the sum of squares for error.
- \(k\) is the number of predictors.
- \(n\) is the number of observations.

#### Interpretation
- A high F-statistic (or a low p-value) indicates that the model is significantly better than the null model (intercept only model).
- Typically, a p-value less than 0.05 is considered statistically significant.

### Other Diagnostic Checks and Tests

#### 3. Checking Residuals

- **Normality of Residuals**: Use the Shapiro-Wilk test or visual methods like Q-Q plots and histograms to check if residuals are normally distributed.
- **Homoscedasticity**: Use the Breusch-Pagan test or plot residuals against fitted values to check for constant variance of residuals.
- **Independence of Residuals**: Use the Durbin-Watson test to check for autocorrelation in residuals.

#### 4. Multicollinearity

- **Variance Inflation Factor (VIF)**: Calculate VIF for each predictor. A VIF value greater than 10 indicates high multicollinearity.

### Practical Example in Python

Here’s how you might perform these tests using Python with the `statsmodels` library:

```python
import numpy as np
import statsmodels.api as sm
import statsmodels.stats.api as sms

# Generate example data
np.random.seed(0)
X = np.random.rand(100, 2)
X = sm.add_constant(X)
beta = np.array([1, 2, 3])
y = X @ beta + np.random.normal(size=100)

# Fit the linear regression model
model = sm.OLS(y, X).fit()

# Summary of the regression model
print(model.summary())

# t-tests for individual coefficients
print("t-values:", model.tvalues)
print("p-values:", model.pvalues)

# F-test for overall model significance
print("F-statistic:", model.fvalue)
print("F-p-value:", model.f_pvalue)

# Shapiro-Wilk test for normality of residuals
shapiro_test = sms.shapiro(model.resid)
print("Shapiro-Wilk test statistic:", shapiro_test[0])
print("p-value:", shapiro_test[1])

# Breusch-Pagan test for homoscedasticity
bp_test = sms.het_breuschpagan(model.resid, model.model.exog)
print("Breusch-Pagan test statistic:", bp_test[0])
print("p-value:", bp_test[1])

# Durbin-Watson test for autocorrelation
dw_test = sms.durbin_watson(model.resid)
print("Durbin-Watson test statistic:", dw_test)

# VIF for multicollinearity
from statsmodels.stats.outliers_influence import variance_inflation_factor

vif = [variance_inflation_factor(X, i) for i in range(X.shape[1])]
print("VIF values:", vif)
```

### When to Perform These Tests

You should perform these statistical tests after fitting the regression model to ensure that the model assumptions are met and to validate the significance and reliability of your model results. If any assumptions are violated, you may need to take corrective actions such as transforming variables, removing multicollinear predictors, or using robust standard errors.

### Summary

- **t-tests for Individual Coefficients**: Assess the significance of each predictor.
- **F-test for Overall Model Significance**: Assess the overall fit of the model.
- **Residual Checks**: Ensure normality, homoscedasticity, and independence of residuals.
- **Multicollinearity Check**: Assess collinearity among predictors using VIF.

These tests and diagnostics help ensure that your regression model is valid and reliable for making inferences and predictions.