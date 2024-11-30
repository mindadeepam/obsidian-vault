#linear-regression 


### what is linear regression?
- its a statistical model that helps us model linear relatinship bw non-collinear and iid data.

### what are the assumptions we make about the data for linear regression?

1. there must be linear relationship bw features and target.
2. features are non-collinear
3. erros must be normally distributed for a fixed X.
4. variance and mean of errors/residuals is constant (Homoscedasticity).
5. errors must be iid. 
   - Independent: Each data point is independent of the others, meaning that the occurrence of one data point doesn’t influence the others.
   - Identically Distributed: All data points are drawn from the same probability distribution.

If any of these assumptions is violated (i.e., if there are nonlinear relationships between dependent and independent variables or the errors exhibit correlation, heteroscedasticity, or non-normality), then the forecasts, confidence intervals, and scientific insights yielded by a regression model may be (at best) inefficient or (at worst) seriously biased or misleading.

vs 

There are four assumptions associated with a linear regression model:
1. ~~**Linearity**: The relationship between X and the mean of Y is linear.~~
2. ~~**Homoscedasticity**: The variance of residual is the same for any value of X.~~
3. **Independence**: Observations are independent of each other.
4. **Normality**: For any fixed value of X, Y is normally distributed.
#not-really-sure 

### How to test for these assumptions?

see [[linear functions aka models#Statistical Tests in Linear Regression]]
also see [this](https://people.duke.edu/~rnau/testing.htm) seemingly great read. #to-read

### Why is multicolliearity an issue for statistical(linear) models?
- explainibility: the marginal effect of the dependent variables (coefficients) cant be reliably interpreted then.
- increases variance of coefficients due to same above reason. This makes model unstable and harms model generality.
- not really a problem as far as accuracy?

### how to solve for outliers?
ie how to make model ignore outliers, ie decrease model capacity

- use MAE instead of MSE as loss function, as mae is more robust to outliers, mse incr4. ***????*** #not-really-sure
- **Regularization** techniques. see [[#^regularization-helps-overfitting]]

### how to solve for multicolliearity?

- **Regularization:** Feature selection via ridge(l2) regression. (this way coeff would not become zero but correlated features would have low coeffs). this works because of l2 regularization and wont work in l1 because:
  
  *Codependence tends to increase coefficient variance see [[linear regression#^coeff-variance]], making coefficients unreliable/unstable, which hurts model generality. L2 reduces the variance of these estimates, which counteracts the effect of codependencies.* [ref](https://explained.ai/regularization/L1vsL2.html)
  
  *Something to consider when using L1 regularization is that when we have highly correlated features, the L1 norm would select only 1 of the features from the group of correlated features in an arbitrary nature, which is something that we might not want." [ref](https://neptune.ai/blog/fighting-overfitting-with-l1-or-l2-regularization#:~:text=The%20differences%20between%20L1%20and%20L2%20regularization%3A&text=The%20L1%20regularization%20solution%20is,has%20built%2Din%20feature%20selection.)* ^l2-for-collinearity
  
- if interpretibilty is not a concern, maybe PCA or even l1 regularization ***???*** 
- use algorithms less sensitive to collinearity like random forests.

### What is regularization?
- Regularization is a method that helps control the `capacity` of the model. i.e. the complexity/size/etc. it can reduce overfitting, make robust to outliers, alleviate collinearity.

- Some common methods include 
	- l1 regression
	- l2 regression
	- random dropping of parameters (dropout) 

- helps to decrease overfitting / increases robustness to outliers : 
  ***extreme coefficients are unlikely to yield models that generalize well.** The solution, therefore, is simply to constrain the magnitude of linear model coefficients so they don't get too big. Constraining the coefficients means not allowing them to reach their optimal position, at the minimum loss location. That means we pay a price for improved generality in the form of decreased accuracy (increase in bias). This is a worthwhile trade because, as we can see from this example, unregulated models on real data sets don't generalize well (they have terrible accuracy on validation sets).* ^regularization-helps-overfitting

### what is l1 l2 regularization?

l1 reg: penalizes magnitude of coeffs, weight update eqn $- \theta_{t+1} = \theta_{t} + \lambda|\theta|$
l2 reg: penalizes square of coeffs, weight update eqn $-> \theta_{t+1} = \theta_{t} + \lambda|\theta|^2$

**Differences:**
- l1 is more roubst as l2 is more sensitive to outliers: cost of outliers in l2 increases by $\text{pow}(2)$.
- l1 acts as feature selectors by pushing features to have 0 coeff. while ***l2 distributes weights among all features***, penalizing high magnitude (by square pow) and not pushing them to extremes. this happens more strongly if we increase power (l3,l4) see [[#Why dont we do l3, l4 .. regularization?]]  
- l1 gives sparse outputs.
- $x \to 0$ cases as much as l1.
- l2 aids in removing multicollinearity. see [[#^l2-for-collinearity]]

**References:**
- [ref](https://neptune.ai/blog/fighting-overfitting-with-l1-or-l2-regularization#:~:text=The%20differences%20between%20L1%20and%20L2%20regularization%3A&text=The%20L1%20regularization%20solution%20is,has%20built%2Din%20feature%20selection.)
- https://www.reddit.com/r/statistics/comments/gstpnx/d_l1_vs_l2_regularization/
- https://explained.ai/regularization/index.html
- https://explained.ai/regularization/L1vsL2.html

#### Why dont we do l3, l4 .. regularization? 

1. L3 and L4 norms demonstrate similar effects as L2, without offering significant new advantages (make weights close to 0). L1 regularization, in contrast, zeroes out weights and introduces sparsity, useful for feature selection.
2. Computational complexity is another vital aspect. Regularization affects the optimization process’s complexity. L3 and L4 norms are computationally heavier than L2, making them less feasible for most machine learning applications.
   [ref](https://towardsdatascience.com/courage-to-learn-ml-demystifying-l1-l2-regularization-part-3-ee27cd4b557a#:~:text=Regularization%20affects%20the%20optimization%20process's,for%20most%20machine%20learning%20applications.)

**convexity?????** #not-really-sure
This is because for powers greater than 2, the cost function/loss curve becomes non-convex whch introduces local minimas. see [[#^convexity-fucntion]]
Key Properties of Convex Functions:
  - Local Minima are Global Minima: If a function is convex, any local minimum is also a global minimum
  - Gradient Descent Guarantees: Convexity ensures that gradient descent methods will converge to the global minimum if the function is smooth. 
  - sum of convex functions is also convex.
mae and mse are convex & l1 and l2 are also convex. So any combination of them is convex making it the default choice mostly. 

> Suppose we train the same model with varying regularization terms $a*R_k$ where "$a$" is the coefficient and "$R_k$" is regularization using the k-norm.
> Higher values of "a" will drive the weights of the model down. This is kinda obvious, since bigger "a" means each weight contributes more to the loss.
> On the other hand, changing the values of k will not necessarily change the average weight amplitude, but will affect how the weights are distributed. Higher values of k will cause the model to have weights with more similar magnitudes, and smaller values of k will drive the magnitudes farther apart.
> For instance, using L1 regularization is useful (even though it is not smooth) because it often drives weights down to zero (so the corresponding parameters can be removed from the model). Using L2 or L3 on the other hand would keep weights at similar non-zero magnitudes.
> In the extreme, L(infinity) would incentivize the model towards having parameter weights that are identical across different parameters. [ref](https://www.reddit.com/r/MachineLearning/comments/13tq3p8/d_interview_question_what_happens_if_we_add_l3/)

not useful refs: [1](https://stats.stackexchange.com/questions/269298/why-do-we-only-see-l-1-and-l-2-regularization-but-not-other-norms), [this](https://stats.stackexchange.com/questions/269298/why-do-we-only-see-l-1-and-l-2-regularization-but-not-other-norms) in [this](https://stats.stackexchange.com/questions/268009/l-p-norms-what-is-special-about-p-2) seems mathematically sound, although i dont get it

#### why not penalize biases?

> In brief, the primary objective of regularization techniques like L1 and L2 is to predominantly prevent overfitting by regulating the magnitude of the model’s weights (personally, I think that’s why we call them regularizations). Conversely, biases have a relatively modest impact on model complexity, which typically renders the need to penalize them unnecessary. [ref](https://towardsdatascience.com/understanding-l1-l2-regularization-part-1-9c7affe6f920)

### What is overfitting? how to solve for it?

Overfitting occurs when a model learns the noise and details of the training data too well, resulting in poor generalization to new, unseen data.
ways to decrease overfitting:
- reduce size of model
- reduce complexity of model by regularization. but this has a increased bias ie reduced accuracy tradeoff. ie reduce accuracy for higher generalizability. see [[#^regularization-helps-overfitting]]

### how to solve for outliers?
ie how to make model ignore outliers, ie decrease model capacity.

- usually detect and remove using Z-score and other statistical tests that detect outliers in the data.

but sometimes outliers represent something meaningful and we cant simply remove them. So, to make our models robust to outliers, we can:

- use MAE instead of MSE as loss function, as mae is more robust to outliers, mse incr4. ***????*** #not-really-sure
- **Regularization** techniques. see [[#^regularization-helps-overfitting]]
- use loss fucntions that handle outliers well like huber loss [[#^huber-loss]] 


### what are the metrics?
[ref](https://medium.com/analytics-vidhya/mae-mse-rmse-coefficient-of-determination-adjusted-r-squared-which-metric-is-better-cd0326a5697e)
- mae: 
	- formula:  $MSE = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$
	- equal error regardless of magnitude of y.
	- robust to outliers
	- non-diff at 0!

- mse:  
	- formula: $MSE = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$ 
	- penalize large errors more, mkaing it sensitive to outliers
- rmse: bcuz mse is not interpretable since it has $\text{unit}^2$, so rmse provides all the desirable computational properties of mse and makes it more explainable.
- coeff of determination / $R^2$:  
	- $R^2 = 1 - ((y-y_{pred})^2/(y-y_{mean})^2)$ 
	- if model has 100 variance, how much of it does our model explain
	- limitation: R² always increases when you add more predictors, even if they are not statistically significant. This can lead to overfitting, where the model captures noise rather than the underlying relationship.
- adjusted $R^2$:
	- $R_{adj}^2 = 1 - \left( \frac{SS_{res}/(n-k-1)}{SS_{tot}/(n-1)} \right)$
	- or in terms of $R^2$ -> $R_{adj}^2 = 1 - \left( \frac{(1 - R^2)(n - 1)}{n - k - 1} \right)$
	- It accounts for the loss of degrees of freedom when adding predictors, making it a more reliable measure for comparing models with different numbers of predictors.




### Appendix

- **What is coefficient variance?**
	Coefficient Variance:
	- When the model is trained on different subsets of the data, the estimated coefficients might vary greatly if the features are highly correlated. This leads to high variance in the coefficients.
	- High variance means the coefficients change significantly depending on the sample of data used, making them unreliable and unstable.
	- These unstable coefficients hurt the model’s ability to generalize to unseen data, as the model overfits to the particular sample it was trained on.
	  
  **model variance vs coeff variance**: model variance measures the variability of the model's predictions for a given input when the model is trained on different subsets of the data. It reflects how much the model's output changes across different training samples. its a more broader concept.  ^coeff-variance

- **What is convexity?** 
  A function  f(x)  is called convex if, for any two points  $x_1$  and  $x_2$  in its domain, and for any  $\lambda$  in the range $[0, 1]$, the following inequality holds: 
  
  $f(\lambda x_1 + (1 - \lambda) x_2) \leq \lambda f(x_1) + (1 - \lambda) f(x_2)$ .
  
  This means that the line segment joining any two points on the graph of the function lies above or on the graph itself. ^convexity-fucntion
- **huber loss**
	The Huber loss function  $L_{\delta}(r)$  is defined as:
	$$L_{\delta}(r) =
	\begin{cases}
	\frac{1}{2} r^2 & \text{for } |r| \leq \delta, \\
	\delta (|r| - \frac{1}{2} \delta) & \text{for } |r| > \delta,
	\end{cases}
	$$
	where:
	•	 $r = y - \hat{y}$  is the residual (the difference between the true value  $y$  and the predicted value  $\hat{y}$ ).
	•	$\delta$ is a threshold parameter that controls the point where the loss transitions from quadratic (like MSE) to linear (like MAE).
	^huber-loss



## next
- trees, 
- statistical tests for linear models. [[linear functions aka models#Statistical Tests in Linear Regression]]