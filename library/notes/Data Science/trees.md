[ref](https://chatgpt.com/share/66f08d5f-70f4-800c-af98-80686868fa54)
### what are trees?

Decision Trees are if else statements that prgressively partition data on the value of its features and then based on some limting criteria, they divide tha data into buckets/classes. The conditions are greesily chosen to achieve the least entropy. ie those groups must have similar target value (in both regression and classification)

An interesting concept applied generally in classical ML is **ensembling**. It is based on the idea that multiple ~~dumb models can actually modle data well~~. There are concepts like bagging abd boosting that train ensembles of trees...

### why trees?

Trees because trees are simple and fast.
Trees beacause trees are explainable. This is big in industrial cases.
Trees because tabular me aur kya karoge bhai?


### do trees have assumptions on data being modelled?

Trees dont have assumptions on the data being modelled I believe.


#### do multicollinear features hurt trees? why?

no. multicolliearity hurts linear models because it makes coefficients of linear models unstable. Trees dont estimate coefficients. Moreover at each node, it selects 1 feature to split on. **Ensembling** also helps **but how ????????????????**


### how do classifcaiton trees work?
Decision trees for classification use a tree-like model to make decisions based on features. The tree is constructed by recursively splitting the dataset into subsets based on the attribute that maximizes information gain (or minimizes Gini impurity).

Construction Steps

1.	Select the Best Attribute: Use Information Gain or Gini Index to choose the attribute that provides the best split.
2.	Split the Dataset: Divide the dataset into subsets based on the selected attribute’s values.
3.	Create Child Nodes: Each subset becomes a child node.
4.	Repeat: For each child node, repeat the process (select the best attribute, split the dataset) until one of the stopping conditions is met:re
	•	All instances in a node belong to the same class.
	•	No more attributes are left to split.
	•	A predefined tree depth or minimum samples per leaf is reached.
5.	Assign Labels: Once the tree is fully grown, assign a class label to each leaf node based on the majority class of instances in that node.

### what is entropy?
Entropy measures the uncertainty or impurity in a dataset. In the context of decision trees, it’s used to quantify the amount of disorder in a set of examples. ie the more uncertainity, the more information there is.

eg if labels =[1,1,1,1,1,1], $H(S)$ = 0

formula is $$H(S) = -\sum_{i=1}^{C} p_i*log(p_i)$$ where $p$ = fraction of label $C_i$ in data.


### what is gini-index?

- **gini is a little more interpretable than entropy as there is no log scaling**
- formula:  $G(S) = 1 - \sum_{i=1}^{C} p_i^2$ 

### what is information gain?

we measure the best split of a node based on information gain (or loss if u look at it the other way). ie if by splitting on a feature $X_i$, entropy is reducing, that means model is explaining the variance of the data ...

$$IG(S, A) = H(S) - \sum_{v \in \text{Values}(A)} \frac{|S_v|}{|S|} H(S_v)$$

where:

•	 $H(S)$  is the entropy of the original dataset  $S$ . parent node.
•	 $S_v$  is the subset of  $S$  for which attribute  A  has value  v . ie a child node.
•	 $\text{Values}(A)$  is the set of all possible values for attribute  A .
•	 $|S_v|$  is the number of instances in subset  $S_v$ .
•	 $|S|$  is the number of instances in the original dataset  $S$ .

### how to handle overfitting in trees?



### how do regression trees work


# ensembles

### why ensemble?
### what is / why bagging?
### what is / why boosting?