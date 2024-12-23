basically measures if 2 arrays are linearly related, perfect score of 1 means they are.
## 1. Autocorrelation Function (ACF) Calculation

### Introduction
The Autocorrelation Function (ACF) measures the correlation between observations of a time series separated by $k$ time units. It provides insights into the internal structure of the data, particularly how observations are related to each other over time.

### Mathematical Definition
The ACF at lag $k$, denoted as $\rho(k)$, is defined as:
$$
\rho(k) = \frac{\gamma(k)}{\gamma(0)}
$$
where $\gamma(k)$ is the autocovariance at lag $k$ and $\gamma(0)$ is the variance of the series.

The autocovariance $\gamma(k)$ is given by:
$$
\gamma(k) = \frac{1}{N} \sum_{t=1}^{N-k} (x_t - \mu)(x_{t+k} - \mu)
$$
> [!info] note the multiplication of values after subtracting mean aka Auto-Covariance

where:
- $N$ is the total number of observations.
- $x_t$ is the value of the time series at time $t$.
- $\mu$ is the mean of the time series.



$$
\gamma(0) = \frac{1}{N} \sum_{t=1}^{N} (x_t - \mu)^2
$$

### Example Calculation
Let's consider a simple time series: $\{2, 4, 6, 8, 10\}$.

1. **Calculate the Mean ($\mu$)**:
   $$
   \mu = \frac{1}{5} (2 + 4 + 6 + 8 + 10) = 6
   $$

2. **Calculate the Variance ($\gamma(0)$)**:
   $$
   \gamma(0) = \frac{1}{5} \left[ (2 - 6)^2 + (4 - 6)^2 + (6 - 6)^2 + (8 - 6)^2 + (10 - 6)^2 \right]
   $$
   $$
   \gamma(0) = \frac{1}{5} \left[ 16 + 4 + 0 + 4 + 16 \right] = \frac{40}{5} = 8
   $$

3. **Calculate the Autocovariance for Lag $k = 1$ ($\gamma(1)$)**:
   $$
   \gamma(1) = \frac{1}{4} \left[ (2 - 6)(4 - 6) + (4 - 6)(6 - 6) + (6 - 6)(8 - 6) + (8 - 6)(10 - 6) \right]
   $$
   $$
   \gamma(1) = \frac{1}{4} \left[ (-4)(-2) + (-2)(0) + (0)(2) + (2)(4) \right]
   $$
   $$
   \gamma(1) = \frac{1}{4} \left[ 8 + 0 + 0 + 8 \right] = \frac{16}{4} = 4
   $$

4. **Calculate the ACF for Lag $k = 1$ ($\rho(1)$)**:
   $$
   \rho(1) = \frac{\gamma(1)}{\gamma(0)} = \frac{4}{8} = 0.5
   $$

Thus, the ACF at lag 1 for the given time series is 0.5.

### Summary
The ACF provides a way to measure the linear relationship between lagged values of a time series. It is calculated using the autocovariance at different lags normalized by the variance. This example illustrates the step-by-step process for computing the ACF for a simple time series.


# 2. Cross correlation vs. Autocorrelation


### Differences
- **Definition**: Autocorrelation measures the correlation between observations of a time series at different time lags, while cross-correlation measures the correlation between two different time series at different time lags.
- **Formula**:
  - Autocorrelation at lag $k$ ($\rho(k)$) for a time series $x$:
    $$ \rho(k) = \frac{\gamma(k)}{\gamma(0)} $$
  - Cross-correlation at lag $k$ ($\rho_{xy}(k)$) between time series $x$ and $y$:
    $$ \rho_{xy}(k) = \frac{\gamma_{xy}(k)}{\sqrt{\gamma_x(0) \cdot \gamma_y(0)}} $$
- **Autocovariance vs. Cross-Covariance**:
  - Autocovariance $\gamma(k)$ measures the covariance of a time series with itself at lag $k$.
  - Cross-covariance $\gamma_{xy}(k)$ measures the covariance between two different time series $x$ and $y$ at lag $k$.
  $\gamma_{xy}(0)$ can be calculated using the following formula:

$$
\text{Cov}(X, Y) = \frac{1}{n} \sum_{i=1}^{n} (x_i - \bar{X})(y_i - \bar{Y})
$$
- **Normalization**:
  - Autocorrelation is normalized by the variance of the time series ($\gamma(0)$).
  - Cross-correlation is normalized by the square root of the product of the variances of the two time series ($\sqrt{\gamma_x(0) \cdot \gamma_y(0)}$).
- **Interpretation**:
  - Autocorrelation helps understand how a time series is related to its past observations.
  - Cross-correlation helps understand the relationship between two different time series.

### Summary
Autocorrelation and cross-correlation are both measures of correlation between time series data, but they differ in their definitions, formulas, and interpretations. Autocorrelation focuses on the internal structure of a single time series, while cross-correlation examines the relationship between two different time series.



# 3. Code

```python 
def get_most_important_lags(x, nlags=400, alpha=0.05, y=None) :
if y:
	cr_values, confint = ccf(x, y, nlags=400, alpha=0.05)
else:
	cr_values, confint = pacf(x, nlags=nlags, alpha=alpha)
significant_lags = [lag for lag, (lower, upper) in enumerate(confint)
	if lower > 0 or upper < 0]
return significant_lags
```

### Why does a confidence interval including 0 mean the difference is not significant?

see this [link](https://stats.stackexchange.com/questions/120949/why-does-a-confidence-interval-including-0-mean-the-difference-is-not-significan)

