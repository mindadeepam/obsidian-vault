#rnn #lstm #practical-tips


# Backpropagation in RNNs

![[Screenshot 2024-07-28 at 9.46.57 PM.png]]

> tldr: we only need to update the W, hence only need ${\partial l}/{\partial W_{hh}}$ at every timestep but that depends on ${\partial l}/{\partial dh_{t-1}}$ from previous timestep. So $|W|$ being small causes vanishing gradients. 

backpropagation trhough time (bptt) happens via the hidden states in vanilla rnns. see equations. intuitively, $h_t$ is a funtion of $h_{t-1}$ so ..

## Issues:

If you observe, to compute the gradient wrt the previous hidden state, which is the downstream gradient, the upstream gradient flows through the tanh non-linearity and gets multiplied by the weight matrix. Now, since this downstream gradient flows back across time steps, it means the computation happens over and over again at every time step. There are a couple of problems with this:

1. Since we’re multiplying over and over again by the weight matrix, the gradient will be scaled up or down depending on the largest singular value of the matrix: if the singular value is greater than 1, we’ll face an _exploding gradient_ problem, and if it’s less than 1, we’ll face a _vanishing gradient_ problem.
2. Now, the gradient passes through the tanh non-linearity which has saturating regions at the extremes. It means the gradient will essentially become zero if it has a high or low value once it passes through the non-linearity — so the gradient cannot propagate effectively across long sequences and it leads to ineffective optimization.

### 1. vanishing gradients remedies

#### a. Using LSTMs sure helps. But why?

1. **Gating Mechanism**:
   LSTMs introduce a gating mechanism that allows for better control of information flow. The key component is the cell state, which acts as a highway for information to flow through the network with minimal alteration.

2. Additive Updates:
   Unlike standard RNNs which completely overwrite their hidden state at each time step, LSTMs use additive updates. This means that gradients can flow backwards through the cell state without being multiplied by the recurrent weight matrix at every time step.

3. **Constant Error Carousel (CEC)**:
   The cell state in LSTMs acts as a CEC, allowing gradients to flow backwards through time with minimal decay. This is achieved through the forget gate, which can be set close to 1 to maintain information for long periods.

4. Gradient Flow Analysis:
   Mathematically, we can analyze the gradient flow in LSTMs. Let's consider the cell state update equation:

   $c[t] = f[t] * c[t-1] + i[t] * g[t]$

   Where f[t] is the forget gate, i[t] is the input gate, and g[t] is the candidate cell state.

   The gradient of the loss L with respect to c[t-1] is:

   $∂L/∂c[t-1] = ∂L/∂c[t] * f[t]$

   This multiplicative factor f[t] is learned and can be close to 1, allowing gradients to flow backwards without vanishing.

5. Adaptive Learning:
   LSTMs can learn when to forget old information and when to update with new information, effectively adapting the time scale of remembered information based on the task at hand.


#### b. Residual connections:

- Implement skip connections or residual blocks (as in ResNet architecture):
  residual connections. this will ensure thst if in longer sequences, gradients coming from non-lienarity are ->0, then there is another path for gradients to flow.

- Batch Normalization:
   Normalize the inputs to each layer, which can stabilize the gradient flow. (read up on this)

### 2. exploding gradients remedies


#### a. Gradient Clipping

While LSTMs help with vanishing gradients, exploding gradients can still occur in deep networks or with certain initialization schemes. Gradient clipping is indeed an effective technique to address this issue. Let's delve deeper:

1. Gradient Clipping Mechanics:
   Gradient clipping involves scaling down the gradient when its norm exceeds a threshold. The most common form is:

   if $||g|| > c:$
      $g = (c / ||g||) * g$

   Where g is the gradient, ||g|| is its L2 norm, and c is the clipping threshold.

2. Theoretical Justification:
   Gradient clipping can be viewed as adaptively adjusting the learning rate. When gradients are large, it effectively reduces the learning rate, preventing overshooting in the optimization landscape.

3. Impact on Optimization:
   By preventing extremely large updates, gradient clipping helps maintain stability in the optimization process. This is particularly important in the early stages of training when gradients can be volatile.

4. Clipping Strategies:
   - Global Norm Clipping: Clip the global norm of the gradient across all parameters.
   - Per-Parameter Clipping: Clip gradients for each parameter individually.
   - Layer-wise Clipping: Apply different clipping thresholds for different layers.

5. Choosing the Clipping Threshold:
   The choice of clipping threshold c is crucial. Too small a value can hinder learning, while too large a value may not prevent explosions effectively. Typical values range from 1 to 10, but this is often task-dependent.

6. Limitations:
   While effective, gradient clipping is a heuristic approach. It doesn't address the root cause of gradient explosion, which often lies in the network architecture or initialization.
   
7. Advanced Considerations:
   - In very deep networks, even clipped gradients may vanish as they propagate backwards. This has led to research into techniques like gradient norm preservation.
   - Some recent work explores learnable clipping thresholds that adapt during training.



### 3. How to detect/examine gradients of a model during training?
#debug #gradients #profiling


```python

from torch.utils.tensorboard import SummaryWriter
writer = SummaryWriter('./runs/')

for name, param in model.named_parameters():
	if param.grad is not None:
		writer.add_histogram(f'gradients/{name}', param.grad, i)

writer.close()
```

#### Q. Why do we see a histogram and how to interpret a histogram of gradients?

Histograms are used because they give us a comprehensive view of the distribution of gradient values across all elements of a parameter (e.g., all weights in a layer) over time. This is more informative than just looking at summary statistics like mean or standard deviation.

#### Q. How to interpret Gradient Histograms?

**Key Elements of the Histogram**
1. **X-axis**: Represents the values of the gradients.
2. **Y-axis**: Represents the frequency or count of gradients falling into each bin.
3. **Color**: Represents time progression (usually from cool to warm colors).

**Interpretation Guidelines**
1. **Central Tendency**: 
   - Look at where the bulk of the distribution is centered.
   - A concentration around zero might indicate slow learning or saturation.
2. **Spread**: 
   - Wide spread suggests diverse gradient values, potentially indicating active learning.
   - Very narrow spread might suggest the layer isn't learning much.
3. **Shape**: 
   - Gaussian (bell-shaped) distributions are common and often desirable.
   - Highly skewed or multimodal distributions might indicate issues.
4. **Extreme Values**: 
   - Look for long tails or outliers, which might indicate exploding gradients.
5. **Changes Over Time**:
   - Gradients should generally become smaller and more concentrated as training progresses.
   - Sudden large changes might indicate instability or learning rate issues.
6. **Comparison Across Layers**:
   - Earlier layers often have smaller gradients than later layers.
   - Drastically different distributions between layers might suggest problems.

**Common Patterns and Their Meanings**
1. **Vanishing Gradients**: Histogram becomes very narrow and centered at zero.
2. **Exploding Gradients**: Histogram shows extreme values or becomes very wide.
3. **Dead Neurons**: Large spike at exactly zero.
4. **Healthy Training**: Gradients start wider and gradually narrow and center closer to zero.




# Papers/References

- "Long Short-Term Memory" by Hochreiter and Schmidhuber (1997) - This paper introduced LSTMs, which addressed the vanishing gradient problem by introducing memory cells and gates. [link](https://blog.xpgreat.com/file/lstm.pdf)
- An excellent article on Medium: [Backpropagation in RNN](https://towardsdatascience.com/backpropagation-in-rnn-explained-bdf853b4e1c2)