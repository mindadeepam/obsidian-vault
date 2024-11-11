#vision
# What is the convolution operation?

The convolution operation is a mathematical process that combines two functions to produce a third function, showing how one function modifies or affects the shape of the other. In general terms, convolution is used to apply a filter to a signal or data. Here's a more detailed explanation:

### Convolution in One Dimension

For two continuous functions $f(t)$ and $g(t)$, the convolution $(f * g)(t)$ is defined as:

$$[ (f * g)(t) = \int_{-\infty}^{\infty} f(\tau) g(t - \tau) \, d\tau ]$$

In discrete terms, for sequences $f[n]$ and $g[n]$, the convolution $(f * g)[n]$ is defined as:

$$[ (f * g)[n] = \sum_{m=-\infty}^{\infty} f[m] g[n - m] ]$$

### Convolution in Two Dimensions

In two dimensions, convolution is similarly defined for functions $f(x, y)$ and $g(x, y)$:

$$[ (f * g)(x, y) = \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} f(\xi, \eta) g(x - \xi, y - \eta) \, d\xi \, d\eta ]$$

For discrete 2D signals (such as images), the convolution of two functions $f$ and $g$ is given by:

$$[ (f * g)[i, j] = \sum_m \sum_n f[m, n] g[i - m, j - n] ]$$

### Steps in Convolution

1. **Flip the Kernel**: The kernel (or filter) $g$ is flipped both horizontally and vertically. why?
   [[#Q Why is the convolutional filter flipped?]]
3. **Slide the Kernel**: The flipped kernel is then slid (convolved) over the input function $f$.
4. **Element-wise Multiplication**: At each position, element-wise multiplications are performed between the kernel and the overlapping portion of the input function.
5. **Summation**: The results of these multiplications are summed to get a single value, which is placed in the output function.

### Example in One Dimension

Consider $f[n] = \{1, 2, 3\}$ and $g[n] = \{0, 1, 0.5\}$:

$$[ (f * g)[0] = (1 \cdot 0) + (2 \cdot 1) + (3 \cdot 0.5) = 0 + 2 + 1.5 = 3.5 ]$$

### Applications

- **Signal Processing**: Convolution is used to filter signals, remove noise, and detect features.
- **Image Processing**: It is used for blurring, sharpening, edge detection, and other transformations.
- **Engineering and Physics**: Convolution describes the output of a system based on its input and impulse response.

# Convolution in Neural networks. aka Conv layer
--- 

Convolutional layers are the core building blocks of Convolutional Neural Networks (CNNs). These layers apply convolutional filters (kernels) to input data, allowing the network to learn spatial hierarchies in images or other structured data. Below is a detailed explanation with key concepts, formulas, and considerations.

### 1. **Key Elements of a Convolutional Layer**

- **Input feature map**: The input to a convolutional layer, typically a 3D tensor of shape $H_{in} \times W_{in} \times D_{in}$, where:
  - $H_{in}$: Height of the input.
  - $W_{in}$: Width of the input.
  - $D_{in}$: Depth (number of channels) of the input (e.g., 3 for RGB images).

- **Filters (Kernels)**: A convolutional layer contains several 3D filters of size $K \times K \times D_{in}$, where:
  - $K \times K$: Spatial size of the filter (height and width).
  - $D_{in}$: Number of channels (same as input depth).
  
  Each filter is responsible for detecting a specific feature in the input.

- **Stride (S)**: This specifies how the filter moves over the input. A stride of $S = 1$ means the filter moves one pixel at a time, while $S = 2$ skips one pixel between each position.

- **Padding (P)**: Zero-padding is often applied to the input feature map to preserve the spatial dimensions after convolution. Common values are:
  - $P = 0$: No padding, causing the output size to shrink.
  - $P = K//2$: This padding keeps the output size the same as the input size when stride is $S = 1$.

- **Number of Filters (Output Channels)**: The number of filters determines the depth of the output feature map. If we apply $D_{out}$ filters, the output feature map will have a depth of $D_{out}$.

### 2. **Mathematics of the Convolution Operation**

For a single 2D convolution, a filter of size $K \times K$ is applied to a patch of the input feature map by performing element-wise multiplication between the filter weights and the input patch, followed by summing the products to get a single output value.

Let’s consider the following:
- $x$ is the input feature map of size $H_{in} \times W_{in} \times D_{in}$.
- $w$ is the convolution kernel (filter) of size $K \times K \times D_{in}$.
- $b$ is the bias term.

The operation at one location for one output pixel can be expressed as:

$$
y(i, j, n) = \sum_{c=0}^{D_{in}-1} \sum_{h=0}^{K-1} \sum_{w=0}^{K-1} x(i+h, j+w, c) \cdot w(h, w, c, n) + b(n)
$$

Where:
- $i, j$ are the spatial locations in the output.
- $c$ indexes over input channels.
- $h, w$ are the height and width of the filter.
- $n$ is the output channel index.

This operation is performed across all spatial locations and channels.

### 3. **Output Shape Calculation**

The size of the output feature map depends on the input size, filter size, stride, and padding. The formula to calculate the output height $H_{out}$ and width $W_{out}$ is:

$$
H_{out} = \left\lfloor \frac{H_{in} - K + 2P}{S} \right\rfloor + 1
$$
$$
W_{out} = \left\lfloor \frac{W_{in} - K + 2P}{S} \right\rfloor + 1
$$

Where:
- $H_{in}, W_{in}$: Input height and width.
- $K$: Filter size (height and width).
- $P$: Padding size.
- $S$: Stride.

The depth of the output feature map is equal to the number of filters $D_{out}$.

### Example:
- **Input size**: $32 \times 32 \times 3$
- **Filter size**: $3 \times 3$
- **Stride**: 1
- **Padding**: 1
- **Number of filters**: 64

Using the output shape formula:
$$
H_{out} = \left\lfloor \frac{32 - 3 + 2 \times 1}{1} \right\rfloor + 1 = 32
$$
$$
W_{out} = \left\lfloor \frac{32 - 3 + 2 \times 1}{1} \right\rfloor + 1 = 32
$$

So, the output feature map will be $32 \times 32 \times 64$.

### 4. **Memory and Computational Costs**

#### Memory:
- **Input memory**: $H_{in} \times W_{in} \times D_{in}$
- **Weights memory**: $K \times K \times D_{in} \times D_{out}$
- **Output memory**: $H_{out} \times W_{out} \times D_{out}$

#### Computational cost (Multiplications/Additions):
For each position in the output feature map, the number of multiplications for a filter is $K \times K \times D_{in}$ and additions is $= (K \times K \times D_{in}) - 1$, and this is repeated for each spatial location and filter. The total number of operations (multiplications and additions) is:

**For the entire output feature map**:
   The total number of operations for the entire output feature map depends on the number of output pixels and the number of output channels.

   - $\text{Total multiplications} = H_{out} \times W_{out} \times D_{out} \times K \times K \times D_{in}$
   - $\text{Total additions} = H_{out} \times W_{out} \times D_{out} \times (K \times K \times D_{in} - 1)$


> Note: number of Mult and adds ops are roughly the same.

$$
\text{Total operations is of the order of} \ -> H_{out} \times W_{out} \times D_{out} \times K \times K \times D_{in}
$$

### Example Calculation:

- Input size: $32 \times 32 \times 3$
- Filter size: $3 \times 3$
- Stride: $1$
- Padding: $1$
- Number of filters: 64

**Memory:**
- Input memory: $32 \times 32 \times 3 = 3,072$
- Weights memory: $3 \times 3 \times 3 \times 64 = 1,728$
- Output memory: $32 \times 32 \times 64 = 65,536$

**Operations:**
- For each output pixel: $3 \times 3 \times 3 = 27$ multiplications.
- Total multiplications: $(32 \times 32 \times 64 \times 27) = 1,769,472$ flops
- Total operations: $2*\text{multiplications} = 2 \times 1,769,472$ $\approx 3.4 million$ flops.

### 5. **Advanced Considerations**

#### Receptive Field:
The receptive field represents the region in the input space that influences a particular output pixel. As the network goes deeper, the receptive field grows, allowing the network to capture larger-scale patterns.

#### Non-linearity:
After each convolution, a non-linear activation function (like ReLU) is applied element-wise to the output feature map, introducing non-linearity into the model.

#### Batch Normalization and Dropout:

Batch normalization normalizes the activations across a mini-batch to stabilize training, while dropout randomly zeroes out neurons during training to prevent overfitting.

The order of operations in a typical convolutional block might look like this:
$$\text{Output} = \text{Activation}(\text{BatchNorm}(\text{Conv}(x)))$$

see [[Batch normalization]] for more details on batchnorm.
### Summary

Convolutional layers are essential for detecting spatial features in structured data such as images. The output size depends on parameters like kernel size, stride, and padding, and the memory and computational costs scale with the depth and spatial dimensions of the input and output feature maps. Understanding the mechanics of convolutions helps in designing efficient neural network architectures for a variety of tasks.



# Why is the convolutional filter flipped?
****
[link](https://stackoverflow.com/questions/45152473/why-is-the-convolutional-filter-flipped-in-convolutional-neural-networks), [link-actual](https://dsp.stackexchange.com/questions/5992/flipping-the-impulse-response-in-convolution/6355#6355)

The underlying reason for transposing a convolutional filter is the definition of the convolution operation - which is a result of signal processing. When performing the convolution, you want the kernel to be flipped with respect to the axis along which you're performing the convolution because if you don't, you end up computing a correlation of a signal with itself. It's a bit easier to understand if you think about applying a 1D convolution to a time series in which the function in question changes very sharply - you don't want your convolution to be skewed by, or correlated with, your signal.

**[This answer](https://dsp.stackexchange.com/questions/5992/flipping-the-impulse-response-in-convolution/6355#6355) from the digital signal processing stack exchange site gives an excellent explanation that walks through the mathematics of why convolutional filters are defined to go in the reverse direction of the signal.**

[This page](http://www.songho.ca/dsp/convolution/convolution2d_example.html) walks through a detailed example where the flip is done. This is a particular type of filter used for edge detection called a Sobel filter. It doesn't explain why the flip is done, but is nice because it gives you a worked-out example in 2D.

I mentioned that it is a bit easier to understand the _why_ (as in, why is convolution defined this way) in the 1D case (the answer from the DSP SE site is really a great explanation); but this convention does apply to 2D and 3D as well (the Conv2DDNN anad Conv3DDNN layers both have the `flip_filter` option). Ultimately, however, because the convolutional filter weights are not something that the human programs, but rather are "learned" by the network, it is entirely arbitrary - _unless you are loading weights from another network_, in which case you must be consistent with the definition of convolution in that network. If convolution was defined correctly (i.e., according to convention), the filter will be flipped. If it was defined incorrectly (in the more "naive" and "lazy" way), it will not.

The broader field that convolutions are a part of is "linear systems theory" so searching for this term might turn up more about this, albeit outside the context of neural networks.

Note that the convolution/correlation distinction is also mentioned in the docstrings of the [corrmm.py](https://github.com/Lasagne/Lasagne/blob/master/lasagne/layers/corrmm.py#L131) class in lasagne:

> flip_filters : bool (default: False) Whether to flip the filters and perform a convolution, or not to flip them and perform a correlation. Flipping adds a bit of overhead, so it is disabled by default. In most cases this does not make a difference anyway because the filters are learnt. However, `flip_filters` should be set to `True` if weights are loaded into it that were learnt using a regular :class:`lasagne.layers.Conv2DLayer`, for example.


# Cross-Correlation vs Convolution

![[Screenshot 2024-07-27 at 9.02.55 PM.png]]
# Backprop of convolution
#todo


# 1x1 convolutions
#todo

- basically only changes the channel dimension and leaves resolution intact.
- are used to reduce dimensionality in filter space (ie as feature selectors) in inception networks. [[CNNs#Inception 2014]]
- are also used as both expansion and reduction projectors/bottlenecks in mobilenets. see [[CNNs#MobileNetV1]]
- changes a H,W,D_{in} to H, W, D_{out}




# Transpose convolutions

- [andrewng lecture](https://www.coursera.org/learn/convolutional-neural-networks/lecture/kyoqR/transpose-convolutions)
- [gfg article](https://www.geeksforgeeks.org/what-is-transposed-convolutional-layer/)
#### Introduction

Transpose convolutions (also referred to as **deconvolutions** or **up-convolutions**) are a key operation in many neural networks, particularly in architectures such as autoencoders and Generative Adversarial Networks (GANs), where upscaling (e.g., increasing spatial dimensions of the input tensor) is required. While standard convolutions reduce the spatial dimensions of the input, transpose convolutions reverse this process.

Despite the name "deconvolution," transpose convolutions do not "invert" a convolution, but rather apply a transformation that enlarges the input while maintaining trainable parameters, akin to a convolution process with reversed spatial effects.

#### Basic Concept

Transpose convolutions work by reversing the downsampling operation that occurs in standard convolutions. For example, if a regular convolution takes a large image and reduces it to a smaller feature map, a transpose convolution will take a smaller feature map and enlarge it to a larger one. The idea is to add zeros between the input feature map's pixels, creating a larger map, and then apply a convolution filter to fill in the gaps and compute the new pixel values.

This operation can be visualized as convolving a sparse matrix (with zeros inserted) to recover higher spatial dimensions.

#### Notation and Formula

The formula to calculate the **output size** of the transpose convolution is derived from the relationship between these parameters and the input size. The formula for the output height and width is as follows:

#### Output Size Formula

Let:
- $H_{out}$, $W_{out}$: Output height, width
- $H_{in}$, $W_{in}$: Input height, width
- $K$: Kernel size
- $S$: Stride
- $P$: Padding
- $P_{out}$: Output padding
- $D$: Dilation (usually $D = 1$ in most practical cases)

The formulas for output height and width are given by:

$$
H_{out} = (H_{in} - 1) \times S - 2 \times P + D \times (K - 1) + P_{out} + 1
$$
$$
W_{out} = (W_{in} - 1) \times S - 2 \times P + D \times (K - 1) + P_{out} + 1
$$

- **Stride $S$**: Controls how much the input is upsampled. Larger strides result in larger output dimensions.
- **Padding $P$**: Padding influences the spatial size of the output. Zero padding maintains dimensional consistency, while additional padding can shrink the output.
- **Kernel size $K$**: Determines how much context is involved in generating each pixel in the output.

#### Example: Calculation of Output Dimensions

Suppose we have a feature map with dimensions $16 \times 16$, and we apply a transpose convolution with the following hyperparameters:

- $K = 3$, $S = 2$, $P = 1$, $P_{out} = 1$, $D = 1$

The output size is calculated as follows:

$$
H_{out} = (16 - 1) \times 2 - 2 \times 1 + 1 \times (3 - 1) + 1 + 1 = 32
$$
$$
W_{out} = (16 - 1) \times 2 - 2 \times 1 + 1 \times (3 - 1) + 1 + 1 = 32
$$

So, the output dimensions are $32 \times 32$.

See below images too, 

![[Transposed-Convolutional-1.png|Transposed Convolutional  Stride = 1|250x150]]

When values overlap (whenever S<K-1), we just add the values which accumulate in the same output cell.

![[Transposed-Convolutional-2.png|Transposed Convolutional  Stride = 2|350x200]]

#### Padding and Stride Considerations

- **Stride**: The stride in transpose convolutions works inversely to that of regular convolutions. For instance, a stride of 2 will effectively double the spatial dimensions of the input feature map.
  
- **Padding**: Padding works similarly as in regular convolutions, but with more nuanced control due to the presence of output padding. The padding value compensates for boundary effects and determines how much the output size deviates from its theoretical full expansion.

- **Output Padding**: Output padding is applied to adjust the size of the output when necessary, usually when exact alignment of the output dimensions is critical.

#### Applications

1. **Image Generation**: In GANs, transpose convolutions are used to generate high-resolution images from low-dimensional noise vectors.
2. **Segmentation**: In semantic segmentation networks like U-Net, transpose convolutions are used to reconstruct pixel-level predictions by upsampling intermediate feature maps to match the original input resolution.
3. **Autoencoders**: In autoencoders, transpose convolutions are used in the decoder portion to upscale the latent representation back to the original image dimensions.

#### Common Architectures Using Transpose Convolutions

- **DCGAN** (Deep Convolutional GAN): Uses transpose convolutions to upsample latent vectors into full images.
- **U-Net**: Utilizes transpose convolutions in the upsampling path for dense image segmentation.
- **Fully Convolutional Networks (FCNs)**: Use transpose convolutions to map features to full-resolution outputs for tasks like segmentation.

#### Conclusion

Transpose convolutions are essential for networks that require upsampling operations, enabling them to reverse the spatial shrinking of convolutions while maintaining trainability. By carefully adjusting parameters like kernel size, stride, padding, and output padding, developers can precisely control the output spatial dimensions.

The output size formula ensures that developers can predict and control the upscaling behavior of these layers, which is crucial for alignment in complex architectures.



# Depthwise separable convolutions

Introduced in Mobilenet-v1 paper. see [[CNNs#MobileNetV1|this]] section in the notes for more on mobilenets.

Make convolutions more efficient by factorizing standard convolutions into a depthwise convolution (applies a single filter per channel) and a pointwise (1x1) convolution, reducing computation and parameters significantlyTwo hyperparameters for controlling trade-offs between latency and accuracy. 
  
  In this we turn the filter/kernel from  $$(D_k, D_k, M) \ -> (D_k,D_k,1) + (1,1,M)$$

  Computation (see [[#Calculating Memory and Operations for a Convolution Operation|this section]])  goes from
  $$D_f.D_f.M . D_k . D_k . N\ -> D_f.D_f.N.D_k.D_k.1 + D_f.D_f.M.1.1.N$$
   where
   $D_k$: filter size
  $D_f$: output resolution
  $N$: input channels
  $M$:output channels
  
  This betters computation by a factor of 
   $$-> (D_f.D_f.N.D_k.D_k.1 + D_f.D_f.M.1.1.N)\ /\ (D_f.D_f.M . D_k . D_k . N)$$
  $$-> 1/M + 1/{D_K^2}$$

![[Screenshot 2024-08-15 at 5.10.47 PM.png|depthwise separable convolutions|300x400]]