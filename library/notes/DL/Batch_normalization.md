[paper](https://arxiv.org/abs/1502.03167)

Batch Normalization (BatchNorm) is a technique used to stabilize and accelerate the training of deep neural networks by normalizing the inputs of each layer. Here’s a detailed breakdown:

### Motivation
1. **Internal Covariate Shift**: During training, the distribution of inputs to a layer changes as the parameters of previous layers change. This phenomenon, known as internal covariate shift, can slow down training because the network has to adapt to these changing distributions.
2. **Gradient Vanishing/Exploding**: Normalizing activations can help mitigate problems of vanishing or exploding gradients, making training more stable.

### Typical Sequence in a CNN Block:

**Batch Normalization (BatchNorm)** is typically applied after the convolutional layer but before the non-linear activation function (e.g., ReLU). The typical sequence is as follows:

1. **Convolution Layer**: Performs the convolution operation on the input data.
2. **Batch Normalization Layer**: Normalizes the activations from the convolutional layer across a mini-batch.
3. **Activation Function**: Applies a non-linear activation function (like ReLU).

The order of operations in a typical convolutional block might look like this:
$$\text{Output} = \text{Activation}(\text{BatchNorm}(\text{Conv}(x)))$$

This sequence ensures that the activations are properly normalized before applying the non-linearity, leading to more effective training and faster convergence.

### Detailed Explanation

- **Why apply BatchNorm after the convolution?**
  The outputs of the convolution layer are normalized to have a mean of zero and a standard deviation of one across the mini-batch. This normalization stabilizes the training process by reducing internal covariate shifts, making gradients more predictable and preventing the vanishing/exploding gradient problem.

- **Why apply BatchNorm before the activation function?**
  BatchNorm is applied before the activation function because normalizing the inputs to the non-linearity often leads to better training dynamics. If BatchNorm were applied after the non-linearity, the normalized values would be less effective, especially for activations like ReLU that zero out negative inputs.
  
### Formula
However, forcing all the pre-activations to be zero and unit standard deviation across all batches can be too restrictive. It may be the case that the fluctuant distributions are necessary for the network to learn certain classes better.

To address this, batch normalization introduces two parameters: a scaling factor `gamma` (γ) and an offset `beta` (β)

For a mini-batch of size \( m \), and each feature \( x_i \) **ie each channel** in the batch, BatchNorm transforms the input as follows:

1.	Compute Mean and Variance for Each Channel:
For each channel  c , compute the mean and variance across all spatial locations and batch dimensions:
$$ \mu_c = \frac{1}{B \times H \times W} \sum_{b=1}^{B} \sum_{h=1}^{H} \sum_{w=1}^{W} x_{b,c,h,w} $$
$$ \sigma_c^2 = \frac{1}{B \times H \times W} \sum_{b=1}^{B} \sum_{h=1}^{H} \sum_{w=1}^{W} (x_{b,c,h,w} - \mu_c)^2 $$
2.	Normalize:
For each channel  c , normalize each feature map  x_{b,c,h,w}  using the computed mean and variance:
$$ \hat{x}{b,c,h,w} = \frac{x{b,c,h,w} - \mu_c}{\sqrt{\sigma_c^2 + \epsilon}} $$
where $\epsilon$ is a small constant to avoid division by zero.
   
3.	Scale and Shift:
Apply learnable parameters  \gamma_c  and  \beta_c  for each channel:
$$ y_{b,c,h,w} = \gamma_c \hat{x}_{b,c,h,w} + \beta_c $$

where $\gamma$ and $\beta$ are learnable parameters that allow the network to maintain its capacity to represent complex functions.

However, at test time (inference time), we may not necessarily have a batch to compute the batch mean and variance. To overcome this limitation, the model works by maintaining a [moving average](https://mathworld.wolfram.com/MovingAverage.html) of the mean and variance at training time, called the moving mean and moving variance. These values are accumulated across batches at training time and used as mean and variance at inference time.
### Applications
- **Convolutional Neural Networks (CNNs)**: BatchNorm is widely used in CNNs to stabilize training and improve convergence.
- **Deep Feedforward Networks**: It helps in speeding up training and improving performance by reducing internal covariate shift.
- **Recurrent Neural Networks (RNNs)**: Variants of BatchNorm are used in RNNs to handle sequence data effectively.

### Benefits
- **Accelerates Training**: By reducing the need for careful initialization and allowing higher learning rates.
- **Improves Convergence**: More stable gradients and faster convergence.
- **Regularization Effect**: It introduces a slight noise into the learning process, which can act as a form of regularization, reducing overfitting.

### Variants and Extensions
- **Layer Normalization**: Normalizes across features rather than across the batch, useful for certain architectures like RNNs.
- **Instance Normalization**: Normalizes per feature map, often used in style transfer tasks.

Overall, BatchNorm has become a standard technique in training deep neural networks due to its effectiveness in speeding up training and improving generalization.

### Limitations of Batch Normalization

Two limitations of batch normalization can arise:

- In batch normalization, we use the _batch statistics_: the mean and standard deviation corresponding to the current mini-batch. However, when the batch size is small, the sample mean and sample standard deviation are not representative enough of the actual distribution and the network cannot learn anything meaningful.
- As batch normalization depends on batch statistics for normalization, it is less suited for sequence models. This is because, in sequence models, we may have sequences of potentially different lengths and smaller batch sizes corresponding to longer sequences.

 layer normalization is another technique that can be used for sequence models. For convolutional neural networks (ConvNets), batch normalization is still recommended for faster training.

References
- https://www.pinecone.io/learn/batch-layer-normalization/
- [paper](https://arxiv.org/abs/1502.03167)