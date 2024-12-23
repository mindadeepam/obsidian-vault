
## bias

bias means the assumptions that a model makes for the data. Simpler the model, the more asumptions it is making. Generally when we have a model with poor performance on the train set we say it has high bias and look to increase the complexity of the model.

**To reduce bias, we want more complex models**
**Good training set performance means low bias**

## variance

variance refers to the stability/generalizability of the model. High variance means the model is very sensitive to small changes in the training data. If the model performs well on train but poorly on test data (assuming distribution is same..) we say that the model could not generalize well and look to either:
- increase data (if that data was not represented in the training)
- reduce model complexity (if current model performs well on train but not on test, it means overfitting.)



## underfitting

underfitting means low scores on train and test both. 
*underfitting means high bias low variance.* 
underfiting means comlpexity of model is not enoguh.
ie model cant capture the data distribution. if data is enough, then that means model bias is high. maybe model complexity maybe some other assumption. (size > archotecture).

## overfitting

low bias (ie training error is low) and high variance (ie high model complexity / high train + low test scores)

