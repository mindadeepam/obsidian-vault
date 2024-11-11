#cnns #convolution

Why CNNs?
-> Regular Neural Nets don’t scale well to full images.


# CS231n course notes

## Conv layer

> check out more about [[Convolutions]] here. But you'd be surpirsed to learn that CNNs actually use correlation and not convoltuion operation. see more [here](https://archive.li/zrPkM). its just a small techincal difference in case of deep-neural networks since the filters are learned.

- filter interaction with image (shapes) explained by chatgpt [here](https://chatgpt.com/share/7f96de6f-c64e-4d80-b390-f02d61f4ea7c)
- **Local Connectivity.** When dealing with high-dimensional inputs such as images, as we saw above it is impractical to connect neurons to all neurons in the previous volume. Instead, we will connect each neuron to only a local region of the input volume. The spatial extent of this connectivity is a hyperparameter called the **receptive field** of the neuron (equivalently this is the filter size). The extent of the connectivity along the depth axis is always equal to the depth of the input volume. It is important to emphasize again this asymmetry in how we treat the spatial dimensions (width and height) and the depth dimension: ***The connections are local in 2D space (along width and height), but always full along the entire depth of the input volume. ie if $C_{in}=12$, then a size 3 filter will be of shape [3*3*12]***

  _Example 1_. For example, suppose that the input volume has size [32x32x3], (e.g. an RGB CIFAR-10 image). If the receptive field (or the filter size) is 5x5, then each neuron in the Conv Layer will have weights to a [5x5x3] region in the input volume, for a total of 5*5*3 = 75 weights (and +1 bias parameter). Notice that the extent of the connectivity along the depth axis must be 3, since this is the depth of the input volume.

  _Example 2_. Suppose an input volume had size [16x16x20]. Then using an example receptive field size of 3x3, every neuron in the Conv Layer would now have a total of 3*3*20 = 180 connections to the input volume. Notice that, again, the connectivity is local in 2D space (e.g. 3x3), but full along the input depth (20).

![[Screenshot 2024-07-27 at 8.19.20 PM.png]]

- **_Use of zero-padding_**: In the example above on left, note that the input dimension was 5 and the output dimension was equal: also 5. This worked out so because our receptive fields were 3 and we used zero padding of 1. If there was no zero-padding used, then the output volume would have had spatial dimension of only 3, because that is how many neurons would have “fit” across the original input. In general, setting zero padding to be P=(F−1)/2𝑃=(𝐹−1)/2 when the stride is S=1𝑆=1 ensures that the input volume and output volume will have the same size spatially. It is very common to use zero-padding in this way and we will discuss the full reasons when we talk more about ConvNet architectures.
- **_Constraints on strides_.**: cant use any hyperparams, output $H$ and $W$ must be integers

**Summary**. To summarize, the Conv Layer:

- Accepts a volume of size 𝑊1×𝐻1×𝐷1
- Requires four hyperparameters:

  - Number of filters 𝐾,
  - their spatial extent 𝐹,
  - the stride 𝑆,
  - the amount of zero padding 𝑃.

- Produces a volume of size 𝑊2×𝐻2×𝐷2 where:
  - 𝑊2=(𝑊1−𝐹+2𝑃)/𝑆+1
  - 𝐻2=(𝐻1−𝐹+2𝑃)/𝑆+1 (i.e. width and height are computed equally by symmetry)
  - 𝐷2=𝐾
- With parameter sharing, it introduces 𝐹⋅𝐹⋅𝐷1 weights per filter, for a total of (𝐹⋅𝐹⋅𝐷1)⋅𝐾 weights and 𝐾 biases.
- In the output volume, the 𝑑-th depth slice (of size 𝑊2×𝐻2) is the result of performing a valid convolution of the 𝑑-th filter over the input volume with a stride of 𝑆, and then offset by 𝑑-th bias.
- **Backpropagation.** The backward pass for a convolution operation (for both the data and the weights) is also a convolution (but with spatially-flipped filters). This is easy to derive in the 1-dimensional case with a toy example (not expanded on for now).
- **Dilated convolutions.** A recent development (e.g. see [paper by Fisher Yu and Vladlen Koltun](https://arxiv.org/abs/1511.07122)) is to introduce one more hyperparameter to the CONV layer called the _dilation_. So far we’ve only discussed CONV filters that are contiguous. This can be very useful in some settings to use in conjunction with 0-dilated filters because it allows you to merge spatial information across the inputs much more agressively with fewer layers.

## Pooling Layer

It is common to periodically insert a Pooling layer in-between successive Conv layers in a ConvNet architecture. Its function is to progressively reduce the spatial size of the representation to ***reduce the amount of parameters and computation*** in the network, and hence to also ***control overfitting***.

More generally, the pooling layer:

- Accepts a volume of size 𝑊1×𝐻1×𝐷1
- Requires two hyperparameters:
  - their spatial extent F,
  - the stride S,
- Produces a volume of size W2×H2×D2 where:
  - W2=(W1−F)/S+1
  - H2=(H1−F)/S+1
  - D2=D1
- Introduces zero parameters since it computes a fixed function of the input
- For Pooling layers, it is not common to pad the input using zero-padding.

**Getting rid of pooling**. Many people dislike the pooling operation and think that we can get away without it. For example, [Striving for Simplicity: The All Convolutional Net](http://arxiv.org/abs/1412.6806) proposes to discard the pooling layer in favor of architecture that only consists of repeated CONV layers. To reduce the size of the representation they suggest using larger stride in CONV layer once in a while. Discarding pooling layers has also been found to be important in training good generative models, such as variational autoencoders (VAEs) or generative adversarial networks (GANs). It seems likely that future architectures will feature very few to no pooling layers.\

## Architecture patters

#### Layer Patterns

The most common form of a ConvNet architecture stacks a few CONV-RELU layers, follows them with POOL layers, and repeats this pattern until the image has been merged spatially to a small size. At some point, it is common to transition to fully-connected layers. The last fully-connected layer holds the output, such as the class scores. In other words, the most common ConvNet architecture follows the pattern:

    `INPUT -> [[CONV -> RELU]*N -> POOL?]*M -> [FC -> RELU]*K -> FC`

_Prefer a stack of small filter CONV to one large receptive field CONV layer_. Suppose that you stack three 3x3 CONV layers on top of each other (with non-linearities in between, of course). In this arrangement, each neuron on the first CONV layer has a 3x3 view of the input volume. A neuron on the second CONV layer has a 3x3 view of the first CONV layer, and hence by extension a 5x5 view of the input volume. Similarly, a neuron on the third CONV layer has a 3x3 view of the 2nd CONV layer, and hence a 7x7 view of the input volume. Suppose that instead of these three layers of 3x3 CONV, we only wanted to use a single CONV layer with 7x7 receptive fields. These neurons would have a receptive field size of the input volume that is identical in spatial extent (7x7), but with several disadvantages. First, the neurons would be computing a linear function over the input, while the three stacks of CONV layers contain non-linearities that make their features more expressive. Second, if we suppose that all the volumes have C𝐶 channels, then it can be seen that the single 7x7 CONV layer would contain $𝐶×(7×7×𝐶)=49𝐶^2$ parameters, while the three 3x3 CONV layers would only contain $3×(𝐶×(3×3×𝐶))=27𝐶^2$ parameters. Intuitively, stacking CONV layers with tiny filters as opposed to having one CONV layer with big filters allows us to express more powerful features of the input, and with fewer parameters. As a practical disadvantage, we might need more memory to hold all the intermediate CONV layer results if we plan to do backpropagation.

**Recent departures.** It should be noted that the conventional paradigm of a linear list of layers has recently been challenged, in Google’s Inception architectures and also in current (state of the art) Residual Networks from Microsoft Research Asia. Both of these (see details below in case studies section) feature more intricate and different connectivity structures.

#### Layer Sizing Patterns

Until now we’ve omitted mentions of common hyperparameters used in each of the layers in a ConvNet. We will first state the common rules of thumb for sizing the architectures and then follow the rules with a discussion of the notation:

The **input layer** (that contains the image) should be divisible by 2 many times. Common numbers include 32 (e.g. CIFAR-10), 64, 96 (e.g. STL-10), or 224 (e.g. common ImageNet ConvNets), 384, and 512.

The **conv layers** should be using small filters (e.g. 3x3 or at most 5x5), using a stride of S=1𝑆=1, and crucially, padding the input volume with zeros in such way that the conv layer does not alter the spatial dimensions of the input. That is, when F=3𝐹=3, then using P=1𝑃=1 will retain the original size of the input. When F=5𝐹=5, P=2𝑃=2. For a general F𝐹, it can be seen that P=(F−1)/2𝑃=(𝐹−1)/2 preserves the input size. If you must use bigger filter sizes (such as 7x7 or so), it is only common to see this on the very first conv layer that is looking at the input image.

The **pool layers** are in charge of downsampling the spatial dimensions of the input. The most common setting is to use max-pooling with 2x2 receptive fields (i.e. F=2𝐹=2), and with a stride of 2 (i.e. S=2𝑆=2). Note that this discards exactly 75% of the activations in an input volume (due to downsampling by 2 in both width and height). Another slightly less common setting is to use 3x3 receptive fields with a stride of 2, but this makes “fitting” more complicated (e.g., a 32x32x3 layer would require zero padding to be used with a max-pooling layer with 3x3 receptive field and stride 2). It is very uncommon to see receptive field sizes for max pooling that are larger than 3 because the pooling is then too lossy and aggressive. This usually leads to worse performance.

_Reducing sizing headaches._ The scheme presented above is pleasing because all the CONV layers preserve the spatial size of their input, while the POOL layers alone are in charge of down-sampling the volumes spatially. In an alternative scheme where we use strides greater than 1 or don’t zero-pad the input in CONV layers, we would have to very carefully keep track of the input volumes throughout the CNN architecture and make sure that all strides and filters “work out”, and that the ConvNet architecture is nicely and symmetrically wired.

_Why use stride of 1 in CONV?_ Smaller strides work better in practice. Additionally, as already mentioned stride 1 allows us to leave all spatial down-sampling to the POOL layers, with the CONV layers only transforming the input volume depth-wise.

_Why use padding?_ In addition to the aforementioned benefit of keeping the spatial sizes constant after CONV, doing this actually improves performance. If the CONV layers were to not zero-pad the inputs and only perform valid convolutions, then the size of the volumes would reduce by a small amount after each CONV, and the information at the borders would be “washed away” too quickly.

_Compromising based on memory constraints._ In some cases (especially early in the ConvNet architectures), the amount of memory can build up very quickly with the rules of thumb presented above. For example, filtering a 224x224x3 image with three 3x3 CONV layers with 64 filters each and padding 1 would create three activation volumes of size [224x224x64]. This amounts to a total of about 10 million activations, or 72MB of memory (per image, for both activations and gradients). Since GPUs are often bottlenecked by memory, it may be necessary to compromise. In practice, people prefer to make the compromise at only the first CONV layer of the network. For example, one compromise might be to use a first CONV layer with filter sizes of 7x7 and stride of 2 (as seen in a ZF net). As another example, an AlexNet uses filter sizes of 11x11 and stride of 4.

**VGGNet in detail**. Lets break down the [VGGNet](http://www.robots.ox.ac.uk/~vgg/research/very_deep/) in more detail as a case study. The whole VGGNet is composed of CONV layers that perform 3x3 convolutions with stride 1 and pad 1, and of POOL layers that perform 2x2 max pooling with stride 2 (and no padding). We can write out the size of the representation at each step of the processing and keep track of both the representation size and the total number of weights:

```
INPUT: [224x224x3]        memory:  224*224*3=150K   weights: 0
CONV3-64: [224x224x64]  memory:  224*224*64=3.2M   weights: (3*3*3)*64 = 1,728
CONV3-64: [224x224x64]  memory:  224*224*64=3.2M   weights: (3*3*64)*64 = 36,864
POOL2: [112x112x64]  memory:  112*112*64=800K   weights: 0
CONV3-128: [112x112x128]  memory:  112*112*128=1.6M   weights: (3*3*64)*128 = 73,728
CONV3-128: [112x112x128]  memory:  112*112*128=1.6M   weights: (3*3*128)*128 = 147,456
POOL2: [56x56x128]  memory:  56*56*128=400K   weights: 0
CONV3-256: [56x56x256]  memory:  56*56*256=800K   weights: (3*3*128)*256 = 294,912
CONV3-256: [56x56x256]  memory:  56*56*256=800K   weights: (3*3*256)*256 = 589,824
CONV3-256: [56x56x256]  memory:  56*56*256=800K   weights: (3*3*256)*256 = 589,824
POOL2: [28x28x256]  memory:  28*28*256=200K   weights: 0
CONV3-512: [28x28x512]  memory:  28*28*512=400K   weights: (3*3*256)*512 = 1,179,648
CONV3-512: [28x28x512]  memory:  28*28*512=400K   weights: (3*3*512)*512 = 2,359,296
CONV3-512: [28x28x512]  memory:  28*28*512=400K   weights: (3*3*512)*512 = 2,359,296
POOL2: [14x14x512]  memory:  14*14*512=100K   weights: 0
CONV3-512: [14x14x512]  memory:  14*14*512=100K   weights: (3*3*512)*512 = 2,359,296
CONV3-512: [14x14x512]  memory:  14*14*512=100K   weights: (3*3*512)*512 = 2,359,296
CONV3-512: [14x14x512]  memory:  14*14*512=100K   weights: (3*3*512)*512 = 2,359,296
POOL2: [7x7x512]  memory:  7*7*512=25K  weights: 0
FC: [1x1x4096]  memory:  4096  weights: 7*7*512*4096 = 102,760,448
FC: [1x1x4096]  memory:  4096  weights: 4096*4096 = 16,777,216
FC: [1x1x1000]  memory:  1000 weights: 4096*1000 = 4,096,000

TOTAL memory: 24M * 4 bytes ~= 93MB / image (only forward! ~*2 for bwd)
TOTAL params: 138M parameters
```

As is common with Convolutional Networks, notice that most of the memory (and also compute time) is used in the early CONV layers, and that most of the parameters are in the last FC layers. In this particular case, the first FC layer contains 100M weights, out of a total of 140M.

# Visulaizing convnets

https://cs231n.github.io/understanding-cnn/


# CNN Architectures

[reference cs231n slides](https://cs231n.stanford.edu/slides/2024/lecture_6_part_1.pdf)

## Alexnet 2012
[paper](https://papers.nips.cc/paper_files/paper/2012/file/c399862d3b9d6b76c8436e924a68c45b-Paper.pdf)

Details/Retrospectives: 
- first use of ReLU 
- used LRN layers (not common anymore) 
- heavy data augmentation 
- dropout 0.5 
- batch size 128 
- SGD Momentum 0.9 
- Learning rate 1e-2, reduced by 10 manually when val accuracy plateaus 
- L2 weight decay 5e-4 
- 7 CNN ensemble: 18.2% -> 15.4%
Alexnet main points were using large filters.

## VGG 2015
[paper](https://arxiv.org/pdf/1409.1556)

![[Screenshot 2024-08-03 at 3.18.22 PM.png|Alexnet vs VGG|400x400]]

- 8 layers (AlexNet) -> 16 - 19 layers (VGG16Net) 
- Only 3x3 CONV stride 1, pad 1 and 2x2 MAX POOL stride 2 
- max pool used for downsampling
- 11.7% top 5 error in ILSVRC’13 (ZFNet) -> 7.3% top 5 error in ILSVRC’14
- ILSVRC’14 2nd in classification, 1st in localization 
- Similar training procedure as Krizhevsky 2012 
- No Local Response Normalisation (LRN) 
- Use VGG16 or VGG19 (VGG19 only slightly better, more memory) 
- Use ensembles for best results 
- FC7 features generalize well to other tasks

  
VGG explored deeper convnets. by making filters smaller. This has several advantages. 
- smaller filters stacjed over multiple layers have **same receptive field** as large filters but **smaller number of parameters** + **more non linearity** so they can learn more complex features.
  ![[Screenshot 2024-08-03 at 2.55.48 PM.png |3 layers of 3x3conv(stride=1) have lower params 3\*($3^2C^2$) than a single 7*7 filter $(7^2C^2)$ but the same receptive field|400x300]]
- Sicne less paramters: total memory in forward pass is reduced! Total memory in forward pass is sum of input volumes before and after each layer
  
  ![[Screenshot 2024-08-03 at 3.06.16 PM.png|notice how conv layers have max memory but FC layers have max parameters|500x300]]
  
  - CNNs kept getting deeper and wider following VGGs
  ![[Screenshot 2024-08-03 at 3.13.30 PM.png|look at how cnns got deeper every year|500x300]]

## Inception 2014

[paper](https://arxiv.org/pdf/1409.4842.pdf)

•	Introduced in ILSVRC 2014, with top-5 error rate of 6.7%
•	Key innovation: Inception module, which applies multiple filter sizes (1x1, 3x3, 5x5) and max pooling in parallel, followed by concatenation of the results
•	Used 22 layers but optimized for computational efficiency
•	Replaced large FC layers with global average pooling to reduce parameters
•	Significant reduction in parameters compared to AlexNet and VGG (only 5M parameters vs 60M+ for VGG)
•	Heavy use of 1x1 convolutions for dimensionality reduction before computationally expensive layers
•	Utilized RMSProp optimizer (learning rate 0.045, decay 0.9, epsilon 1.0)
•	Batch size 128; L2 weight decay of 4e-5
•	Used auxiliary classifiers for intermediate outputs (deep supervision) during training
![[Screenshot 2024-08-14 at 8.05.21 PM.png| 600x220]]

![[Screenshot 2024-08-14 at 8.05.36 PM.png|600x300]]
![[inception-arch.png|inception architecture|800x250]]


## Resnets 2015
[paper](https://arxiv.org/pdf/1512.03385)
[great blog explaing resnets](https://towardsdatascience.com/understanding-and-visualizing-resnets-442284831be8)

Full ResNet architecture: 
- **Stack residual blocks** 
- Every residual block has two 3x3 conv layers 
- Periodically, double # of filters and downsample spatially using stride 2 (/2 in each dimension) 
- Additional conv layer at the beginning (stem) 
- No FC layers at the end (only FC 1000 to output classes) 
- (In theory, you can train a ResNet with input image of variable sizes)
- **Batch Normalization** after every CONV layer. see [[Batch normalization]] for more detail on batchnorm.
- **convolutions with stride 2 used for downsampling instead of pooling**


![[Screenshot 2024-08-14 at 8.29.27 PM.png| 350x700]]


Training ResNet in practice: 
- Batch Normalization after every CONV layer 
- Xavier initialization from He et al. 
- SGD + Momentum (0.9) 
- Learning rate: 0.1, divided by 10 when validation error plateaus 
- Mini-batch size 256 
- Weight decay of 1e-5 
- No dropout used

![[Screenshot 2024-08-03 at 4.00.12 PM.png]]


![[Screenshot 2024-08-03 at 3.59.45 PM.png| without residual connections, deeper layers perform worse!!]]


## MobileNet (V1 and V2)

### MobileNetV1

[MobileNetV1 paper](https://arxiv.org/pdf/1704.04861)
- Focused on lightweight models for mobile and edge devices
- Introduced **depthwise separable convolutions**: Factorize standard convolutions into a depthwise convolution (applies a single filter per channel) and a pointwise (1x1) convolution, reducing computation and parameters significantlyTwo hyperparameters for controlling trade-offs between latency and accuracy. 
  
  In this we turn the filter/kernel from  $$(D_k, D_k, M) \ -> (D_k,D_k,1) + (1,1,M)$$

  Computation goes from
  $$D_f.D_f.M . D_k . D_k . N\ -> D_f.D_f.N.D_k.D_k.1 + D_f.D_f.M.1.1.N$$
   where
   $D_k$: filter size
  $D_f$: output resolution
  $N$: input channels
  $M$:output channels
  
  This betters computation by a factor of 
   $$-> (D_f.D_f.N.D_k.D_k.1 + D_f.D_f.M.1.1.N)\ /\ (D_f.D_f.M . D_k . D_k . N)$$
  $$-> 1/M + 1/{D_K^2}$$
  ![[Screenshot 2024-08-15 at 4.32.46 PM.png|300x200]]
  
- It also defines 2 paramters to build smaller models for more efficiency, width multiplier and resolution multiplier:
  
	- Width multiplier (α): Reduces the number of channels
	- Resolution multiplier (ρ): Reduces the spatial resolution of the input


- Used ReLU6 activation for better stability at lower precision (e.g., 8-bit)
- Achieved impressive performance with only 4.2M parameters and a similar error rate to much larger networks
- Architecure:
  ![[Screenshot 2024-08-15 at 4.33.17 PM.png|250x400]]
### MobileNetV2

[MobileNetV2 paper](https://arxiv.org/pdf/1801.04381)
•	Introduced/added **inverted residuals** and **linear bottlenecks**. 
(residuals are generally around `wide->narrow->wide` blocks while inverted residuals are around
`narrow->wide->narrow` blocks)
•	Bottleneck structure: 1x1 expansion, followed by depthwise separable convolution, then a 1x1 projection to a smaller dimension
•	Skip connections between the bottleneck layers to preserve information
•	Removed non-linear activations in the final layers (linear bottlenecks) to reduce information loss
•	Significantly improved accuracy and efficiency compared to V1, with 3.4M parameters and better classification performance


# References

- sebastian raschka course material [here](https://sebastianraschka.com/blog/2021/dl-course.html#l13-introduction-to-convolutional-neural-networks)
- CS231N CNN article [here](https://cs231n.github.io/convolutional-networks/#case). Great intuition about the shapes, local connectivity, spatial arrangement, and loads of other stuff
- arc folder where i maintain things related to CNNs [link](https://arc.net/folder/C52714A4-4BBF-4DBA-926D-098F2F07314D)
- CNN architectures cs231n slides [link](https://cs231n.stanford.edu/slides/2024/lecture_6_part_1.pdf)
- deplearning.ai CNN course [link](https://www.coursera.org/learn/convolutional-neural-networks/home/week/2) 




# Other tings

- **histogram of gradients** for facial detection: This might seem like a random thing to do, but there’s a really good reason for replacing the pixels with gradients. If we analyze pixels directly, really dark images and really light images of the same person will have totally different pixel values. But by only considering the _direction_ that brightness changes, both really dark images and really bright images will end up with the same exact representation. That makes the problem a lot easier to solve!
- What are **depthwise and pointwise convolutions**: explained by chatgpt [here](https://chatgpt.com/share/d7254a8d-167e-4d07-b9e3-9a6d7be2ee06)
- git diff style features of early cnn models : https://chatgpt.com/share/d00f71ca-da88-45f5-96f4-efc60008334b 



# Archive
%% 
## Introduction to CNNs

### Terminology

- depth - num of channels / $C_o$
- filters / kernels / feature detectors - $K$
- activation / feature map - is the result of conv operation by one filter on input image/tensor.
- receptive field - kernel sized portion of input image that the conv layer acts on

---

While feedforward networks assume independent features, and sequential networks assume sequential relationship, CNNs assume local feature relation.

![image](https://drive.google.com/uc?export=view&id=1mbGw7eCgQJlJjU9i7mM0cPHB62u2MmHK)

## Intuition on the Conv layer

[link](https://cs231n.github.io/convolutional-networks/#case)
The CONV layer’s parameters consist of a set of learnable filters. Every filter is small spatially (along width and height), but extends through the full depth of the input volume. For example, a typical filter on a first layer of a ConvNet might have size 5x5x3 (i.e. 5 pixels width and height, and 3 because images have depth 3, the color channels). During the forward pass, we slide (more precisely, convolve) each filter across the width and height of the input volume and compute dot products between the entries of the filter and the input at any position. As we slide the filter over the width and height of the input volume we will produce a 2-dimensional activation map that gives the responses of that filter at every spatial position. Intuitively, the network will learn filters that activate when they see some type of visual feature such as an edge of some orientation or a blotch of some color on the first layer, or eventually entire honeycomb or wheel-like patterns on higher layers of the network. Now, we will have an entire set of filters in each CONV layer (e.g. 12 filters), and each of them will produce a separate 2-dimensional activation map. We will stack these activation maps along the depth dimension and produce the output volume.

## Main concepts of CNN

[link](https://youtu.be/7fWOE-z8YgY?t=825)

1. sparse connectivity: a single patch in feature map is connected to only a small patch of image (in MLPs there is dense/full connection).
2. parameter sharing: the same kernel/filter slides across the image. ie different neurons in each activation map is calculated using the same filter. In MLPs each neuron in the output space is calculated using different weight values.  this makes it efficient for computation.
3. many layers: combining extracted local patterns to global patterns.
4. convolution is translation invariant

## All about the shapes

If stride=1, pad=0, etc all are default values,

$$
H_o = (H_i - K + 2P)/S + 1
$$

$$
H_{o} = H_{i} - K + 1
$$

For the 1st hidden layer, size decreases from (32,32) to (28,28). hence kernel is (5,5). Since channels go from 1 to 6, num_of_filters/depth = 6.

- These filters are learnable parameters (There is one bias for each output channel. Each bias is added to every element in that output channel)

$$
W = [C_o, C_i,  K_h, K_w]
$$

$$
bias = [C_o]
$$

- **Pooling layers**
  - are not learnable, they just downsampling operation along the spatial dimensions $(H, W)$ by avg/max/min pooling. There is information lost due to pooling.
  - The use of pooling operation helps to extract a combination of features, which are invariant to translational shifts and small distortions.
  - Reduction in the size of feature-map to invariant feature set not only regulates the complexity of the network but also helps in increasing the generalization by reducing overfitting.

Each Conv2d layer is defined by $[C_i, C_o,  K_h, K_w]$,
ie there are $C_{out}$ filters and each is of size -

$$
one\ kernel = [C_i, K_h, K_w]
$$

Since each kernels' depth =

$C_i$

, each filter produces one channel in the resulting matrix irrespective of num of input channels.

Hence if

$$
Input = [C_i, H_i, W_i] = [3,32,32],
$$

$$
Conv2d layer = [C_i, C_o, K_h, K_w] = [3, 16, 3, 3],
$$

 then there are

$$
n=16\ kernels
$$

 each of size

$$
kernel = [C_i, K_h, K_w] = [3, 3, 3]
$$

 $H_o = 32-3+1 = 29$ and same with $W_o$. So,

$$
Output = [C_o, H_o, W_o] = [16, 29, 29]
$$

see [this](https://pytorch.org/docs/stable/generated/torch.nn.Conv2d.html) pytorch documentation on Conv2D for more nuanced understanding on the shapes.

## What a CNN can see

Use different interpretation methods that help visualize which parts if the image the model is focussing on when inferencing. good debugging tool.
Some of the methods have been explained in [this](https://thegradient.pub/a-visual-history-of-interpretation-for-image-recognition/) article.
[This](https://pypi.org/project/grad-cam/) is a library that provides most of these methods.
 %%