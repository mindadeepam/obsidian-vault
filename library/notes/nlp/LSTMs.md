
Althoigh rnns are great we saw some issues related to grdient flow in RNNs especially over long sequences. Lets revisit them once again:

# Vanilla RNN Gradient Flow

An RNN block takes in input ($x_t$) and previous hidden representation ($h_{t-1}$) and learns a transformation, which is then passed through ($\tanh$) to produce the hidden representation  ($h_t$) for the next time step and output ($y_t$) as shown in the equation below.

$$ h_t = \tanh(W_{hh} h_{t-1} + W_{xh} x_t) $$

For the backpropagation, let’s examine how the output at the very last timestep affects the weights at the very first time step. The partial derivative of $h_t$ with respect to $h_{t-1}$ is written as:

$$ \frac{\partial h_t}{\partial h_{t-1}} = \tanh'(W_{hh} h_{t-1} + W_{xh} x_t) W_{hh} $$

We update the weights \( $W_{hh}$ \) by getting the derivative of the loss at the very last time step \( $L_t$ \) with respect to \( $W_{hh}$ \):

$$ \frac{\partial L_t}{\partial W_{hh}} = \frac{\partial L_t}{\partial h_t} \frac{\partial h_t}{\partial h_{t-1}} \ldots \frac{\partial h_1}{\partial W_{hh}} = \frac{\partial L_t}{\partial h_t} \left( \prod_{t=2}^{T} \frac{\partial h_t}{\partial h_{t-1}} \right) \frac{\partial h_1}{\partial W_{hh}} $$

$$ = \frac{\partial L_t}{\partial h_t} \left( \prod_{t=2}^{T} \tanh'(W_{hh} h_{t-1} + W_{xh} x_t) W_{hh}^{T-1} \right) \frac{\partial h_1}{\partial W_{hh}} $$

**Vanishing Gradient:**
We see that ( $\tanh'(W_{hh} h_{t-1} + W_{xh} x_t)$ ) will almost always be less than 1 because ( $\tanh$ ) is always between -1 and 1. Thus, as ( $t$ ) gets larger (i.e., longer timesteps), the gradient $\left( \frac{\partial L_t}{\partial W} \right)$ will decrease in value and get close to zero. This will lead to the vanishing gradient problem, where gradients at future time steps rarely impact gradients at the very first time step. This is problematic when we model a long sequence of inputs because the updates will be extremely slow.

**Removing Non-linearity ( $\tanh$ ):**
If we remove non-linearity ($\tanh$) to solve the vanishing gradient problem, then we will be left with:

$$ \frac{\partial L_t}{\partial W} = \frac{\partial L_t}{\partial h_t} \left( \prod_{t=2}^{T} W_{hh}^{T-1} \right) \frac{\partial h_1}{\partial W} $$

- **Exploding Gradients:**
  If the largest singular value of \( $W_{hh}$ \) is greater than 1, then the gradients will blow up, and the model will get very large gradients coming back from future time steps. Exploding gradient often leads to getting gradients that are NaNs.

- **Vanishing Gradients:**
  If the largest singular value of \( $W_{hh}$ \) is smaller than 1, then we will have a vanishing gradient problem, as mentioned above, which will significantly slow down learning. In practice, we can treat the exploding gradient problem through gradient clipping, which is clipping large gradient values to a maximum threshold. However, since the vanishing gradient problem still exists in cases where the largest singular value of the \( $W_{hh}$ \) matrix is less than one, LSTM was designed to avoid this problem.


# LSTMs

LSTMs were introduced ([paper](https://blog.xpgreat.com/file/lstm.pdf)) to overcome vanishing gradient problem and allow us to model longer-range dependencies in sequences. To do so, they introduce one main concept: using gates to allow gradients to flow along a path, potentially uninterrupted. It also has mechanisms that enable it to add to or remove from that gradient flow path.

The following is the precise formulation for LSTM. On step \( t \), there is a hidden state \( h_t \) and a cell state (c_t\). Both \( h_t \) and \( c_t \) are vectors of size \( n \). One distinction of LSTM from Vanilla RNN is that LSTM has this additional \( c_t \) cell state, and intuitively it can be thought of as \( c_t \) storing long-term information. LSTM can read, erase, and write information to and from this \( c_t \) cell. The way LSTM alters \( c_t \) cell is through three special gates: \( i, f, o \) which correspond to "input", "forget", and "output" gates. The values of these gates vary from closed (0) to open (1). All \( i, f, o \) gates are vectors of size \( n \).

At every timestep, we have an input vector \( x_t \), previous hidden state \( h_{t-1} \), previous cell state \( c_{t-1} \), and LSTM computes the next hidden state \( h_t \), and next cell state \( c_t \) at timestep \( t \) as follows:

$$ f_t = \sigma(W_{hf} h_{t-1} + W_{xf} x_t) $$
$$ i_t = \sigma(W_{hi} h_{t-1} + W_{xi} x_t) $$
$$ o_t = \sigma(W_{ho} h_{t-1} + W_{xo} x_t) $$
$$ g_t = \tanh(W_{hg} h_{t-1} + W_{xg} x_t) $$

![[lstm_mformula_1.png|300]]******

$$ c_t = f_t \odot c_{t-1} + i_t \odot g_t $$
$$h_t = o_t \odot \tanh(c_t)$$

![[lstm_mformula_2.png|300]]

where \( $\odot$ \) is an element-wise Hadamard product. \( g_t \) in the above formulas is an intermediary calculation cache that’s later used with the \( o \) gate in the above formulas.

Since all \( f, i, o \) gate vector values range from 0 to 1, because they were squashed by the sigmoid function \( \sigma \), when multiplied element-wise, we can see that:

- **Forget Gate:** Forget gate \( f_t \) at time step \( t \) controls how much information needs to be “removed” from the previous cell state \( c_{t-1} \). This forget gate learns to erase hidden representations from the previous time steps, which is why LSTM will have two hidden representations \( h_t \) and cell state \( c_t \). This \( c_t \) will get propagated over time and learn whether to forget the previous cell state or not.
- **Input Gate:** Input gate \( i_t \) at time step \( t \) controls how much information needs to be “added” to the next cell state \( c_t \) from the previous hidden state \( h_{t-1} \) and input \( x_t \). Instead of \( \tanh \), the “input” gate \( i \) has a sigmoid function, which converts inputs to values between zero and one. This serves as a switch, where values are either almost always zero or almost always one. This “input” gate decides whether to take the RNN output that is produced by the “gate” gate \( g \) and multiplies the output with the input gate \( i \).
- **Output Gate:** Output gate \( o_t \) at time step \( t \) controls how much information needs to be “shown” as output in the current hidden state \( h_t \).

The key idea of LSTM is the cell state, the horizontal line running through between recurrent timesteps. You can imagine the cell state to be some kind of highway of information passing through straight down the entire chain, with only some minor linear interactions. With the formulation above, it’s easy for information to just flow along this highway. Thus, even when there are a bunch of LSTMs stacked together, we can get an uninterrupted gradient flow where the gradients flow back through cell states instead of hidden states \( h \) without vanishing in every time step.

This greatly fixes the gradient vanishing/exploding problem we have outlined above. Figure 5 also shows that the gradient contains a vector of activations of the “forget” gate. This allows better control of gradient values by using suitable parameter updates of the “forget” gate.

## why this works

![[Screenshot 2024-07-29 at 12.20.26 AM.png]]

The gates are the key. particularly the ability to maintain a constant information flow thst we can either forget or **add** to based on inputs and hidden states allows for stable grad flow. the additive part eespecially helps and is analogous to residual/skip connections in resnets. this is bcuz backprop distributes gradients equally(and identically) across an addition node in the computation graph. so smaller values dont die down, as in multiplication nodes (forget gates kill it though, so they are in practise set with positive bias to allow gradients to flow initially during trainig and then nn can learn to shuut it, f=1 is dont forget, f=0 is forget)

# references
- karpathy cs231n rnns video, timstamped on lstm part-> [link](https://www.youtube.com/watch?v=yCC09vCHzF8) (explains a lot of intuitions)