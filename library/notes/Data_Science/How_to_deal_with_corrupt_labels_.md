Dealing with Corrupt Labels in Machine Learning: Challenges and Solutions

Introduction

Machine learning has rapidly become an essential part of various industries, from healthcare to finance, due to its ability to derive valuable insights from vast amounts of data. However, the success of any machine learning model heavily depends on the quality of its training data. One of the most common challenges faced by practitioners is dealing with corrupt or mislabeled data. This issue can significantly impact the performance and reliability of machine learning models. In this technical blog, we will explore the challenges of handling corrupt labels in machine learning and discuss various solutions proposed in the literature.

%% The Impact of Corrupt Labels
Corrupt labels is a widely familiar problem in data science/ML, especially outside academic settings, where there is not as much control over the data curation process. Hence we must be equipped with a few useful tools to help us push our metrics that extra couple percentages 😉. %%

 Corrupt labels, often referred to as label noise, occur when the ground truth labels assigned to data points do not accurately represent the true class of those data points. This can result from human error during the data labeling process or from inherent ambiguity in the data itself. The consequences of corrupt labels can be profound:

1. **Reduced Model Accuracy:** Machine learning models trained on data with corrupt labels are likely to produce inaccurate and unreliable predictions, as they learn from incorrect information.

2. **Increased Model Bias:** Corrupt labels can introduce bias into the training data, leading models to make predictions that align with these biases. This can perpetuate harmful stereotypes or incorrect assumptions.

3. **Undermined Generalization:** Models trained on noisy data may struggle to generalize well to unseen data, as they have learned patterns that do not exist or learned to ignore important features.

4. **Wasted Resources:** Collecting, cleaning, and labeling data is a resource-intensive process. Working with corrupt labels can result in wasted time and effort. 

Challenges in Handling Corrupt Labels

Addressing corrupt labels is a non-trivial task, as it involves several challenges:

1. **Label Noise Types:** There are different types of label noise, such as random errors, systematic errors, and class imbalance. Each type may require a specific approach to mitigate its effects.

2. **Unknown Magnitude:** In many cases, the extent of label corruption is not known, making it difficult to estimate the impact on model performance accurately.

3. **Data Scarcity:** In some domains, collecting additional labeled data for model training may be expensive or infeasible, making label cleaning a critical step.

4. **Robustness to Label Noise:** Building models that are robust to label noise is a complex problem, as it requires finding a balance between fitting the noisy data and generalizing well to clean data.

Solutions to Address Corrupt Labels

Several techniques and strategies have been proposed in the literature to mitigate the impact of corrupt labels. Here are some notable approaches, along with references to distinguished papers:

1. **Data Cleaning:**
    - **Forward and Reverse Iterative Classification (FRIC):** This technique uses an iterative approach to identify and correct mislabeled data points. ([Frenay and Verleysen, 2014](https://link.springer.com/chapter/10.1007/978-3-319-08302-5_32))
    
2. **Robust Loss Functions:**
    - **Symmetric Cross-Entropy (SCE):** SCE loss function is designed to handle noisy labels by focusing on the likelihood of a predicted label given the input, rather than the true label. ([Wang et al., 2019](https://arxiv.org/abs/1908.06112))
    
3. **Ensemble Methods:**
    - **Co-teaching:** In this approach, two neural networks are trained independently and exchange information about which examples are likely to be mislabeled. ([Han et al., 2018](https://arxiv.org/abs/1804.06872))

4. **Active Learning:**
    - **Confidence-based Labeling:** Active learning strategies that query labels for instances where the model is uncertain, with the assumption that noisy instances are more likely to be in these regions. ([Ray and Craven, 2001](https://www.aaai.org/Library/AAAI/2001/aaai01-048.php))




Conclusion

Dealing with corrupt labels in machine learning is a significant challenge, but various techniques and strategies are available to address this issue. The choice of approach depends on the nature of the label noise, available resources, and the specific machine learning task. By implementing these techniques, practitioners can improve the robustness and reliability of their machine learning models when working with noisy data.

References:

1. Frenay, B., & Verleysen, M. (2014). Classification in the presence of label noise: a survey. In [International Conference on Neural Networks](https://link.springer.com/chapter/10.1007/978-3-319-08302-5_32). Springer.

2. Wang, Y., Ma, T., & Bailey, J. (2019). Symmetric cross entropy for robust learning with noisy labels. [arXiv preprint arXiv:1908.06112](https://arxiv.org/abs/1908.06112).

3. Han, X., Zhang, D., Yao, S., Zhang, J., & Ye, Q. (2018). Co-teaching: Robust training of deep neural networks with extremely noisy labels. [arXiv preprint arXiv:1804.06872](https://arxiv.org/abs/1804.06872).

4. Ray, S., & Craven, M. (2001). Labeling and filtering data for supervised classification in text categorization. In [Proceedings of the Seventeenth conference on Uncertainty in artificial intelligence](https://www.aaai.org/Library/AAAI/2001/aaai01-048.php) (pp. 335-343).





#### [Mixup](https://arxiv.org/pdf/1710.09412.pdf)
 Zhang, Hongyi, et al. "mixup: Beyond empirical risk minimization." _arXiv preprint arXiv:1710.09412_ (2017). 
 
 >Large deep neural networks are powerful, but exhibit undesirable behaviors such as memorization and sensitivity to adversarial examples. In this work, we propose mixup, a simple learning principle to alleviate these issues. In essence, mixup trains a neural network on convex combinations of pairs of examples and their labels. By doing so, mixup regularizes the neural network to favor simple linear behavior in-between training examples. Our experiments on the ImageNet-2012, CIFAR-10, CIFAR-100, Google commands and UCI datasets show that mixup improves the generalization of state-of-the-art neural network architectures. We also find that mixup reduces the memorization of corrupt labels, increases the robustness to adversarial examples, and stabilizes the training of generative adversarial networks
 
- Best explanation of empirical risk minimization. simple data aug method, beat SOTA then.

![[Screenshot 2023-10-12 at 4.21.29 AM.png]]

- this is the code for mixup implementation inside the getitem of a dataset class
```
	
	def __getitem__(self, index):
		img_path = self.image_paths[index]
		print(img_path)
		img = np.array(Image.open(img_path).convert("RGB"))
		if self.aug==True:
		img = self.augmentations(img)
		label = self.labels[index]
		img = self.resize_preprocess_fn(img)
		# img=self.pad_preprocess_fn(img)
		img = img / 255.0
  
		# Mixup implementation
	
		if self.alpha > 0:
			lam = np.random.beta(self.alpha, self.alpha)
			index2 = random.randint(0, len(self.image_paths) - 1)
			img2 = np.array(Image.open(self.image_paths[index2]).convert("RGB"))
			img2 = self.augmentations(img2)
			# img2 = self.resize_preprocess_fn(img2)
			img2=self.pad_preprocess_fn(img2)
			img2 = img2 / 255.0
			mixed_img = lam * img + (1 - lam) * img2
			label2 = self.labels[index2]
			label = label * lam + label2 * (1 - lam)
			return mixed_img.transpose(2, 0, 1).astype('float32'), label
		return img.transpose(2, 0, 1).astype('float32'), label

```


![[Screenshot 2023-10-12 at 5.05.40 AM.png]]

Ablation Studies:

> Mixup is a data augmentation method that consists of only two parts: random convex combination of raw inputs, and correspondingly, convex combination of one-hot label encodings. However, there are several design choices to make. For example, on how to augment the inputs, we could have chosen to interpolate the latent representations (i.e. feature maps) of a neural network, and we could have chosen to interpolate only between the nearest neighbors, or only between inputs of the same class. When the inputs to interpolate come from two different classes, we could have chosen to assign a single label to the synthetic input, for example using the label of the input that weights more in the convex combination. To compare mixup with these alternative possibilities, we run a set of ablation study experiments using the PreAct ResNet-18 architecture on the CIFAR-10 dataset...
> ...From the ablation study experiments, we have the following observations. 
> - First, mixup is the best data augmentation method we test, and is significantly better than the second best method (mix input + label smoothing). 
> - Second, the effect of regularization can be seen by comparing the test error with a small weight decay (10−4 ) with a large one (5 × 10−4 ). For example, for ERM a large weight decay works better, whereas for mixup a small weight decay is preferred, confirming its regularization effects. We also see an increasing advantage of large weight decay when interpolating in higher layers of latent representations, indicating decreasing strength of regularization. 
> - Among all the input interpolation methods, mixing random pairs from all classes (AC + RP) has the strongest regularization effect. 
> - Label smoothing and adding Gaussian noise have a relatively small regularization effect. 
> - Finally, we note that the SMOTE algorithm (Chawla et al., 2002) does not lead to a noticeable gain in performance.

#### Cutmix


#### [Phantom of Benchmark Dataset: Resolving Label Ambiguity Problem on Image Recognition in the Wild](https://openaccess.thecvf.com/content/WACV2023W/DNOW/papers/Chung_Phantom_of_Benchmark_Dataset_Resolving_Label_Ambiguity_Problem_on_Image_WACVW_2023_paper.pdf)

