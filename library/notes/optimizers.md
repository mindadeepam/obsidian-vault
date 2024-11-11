#optim 

https://chatgpt.com/share/66e348e6-0154-800c-9d64-d5feb8db9d32

Let's walk through each optimization algorithm:

### 1. **Stochastic Gradient Descent (SGD)**

**Equation:**
\[
\theta_{t+1} = \theta_t - \eta \nabla_{\theta_t} J(\theta_t)
\]
Where:
- \(\theta_t\) are the model parameters at step \(t\),
- \(\eta\) is the learning rate,
- \(\nabla_{\theta_t} J(\theta_t)\) is the gradient of the loss function \(J(\theta)\) with respect to \(\theta_t\).

**Contribution:**
SGD updates the model parameters by taking steps proportional to the negative gradient of the loss function with respect to the parameters. It's computationally efficient for large datasets since it uses a random subset (batch) of data for each update. However, it suffers from high variance in updates, leading to noisy and sometimes inefficient convergence.

---

### 2. **SGD with Momentum**

**Equation:**
\[
v_{t+1} = \beta v_t + (1 - \beta) \nabla_{\theta_t} J(\theta_t)
\]
\[
\theta_{t+1} = \theta_t - \eta v_{t+1}
\]
Where:
- \(v_t\) is the velocity (accumulated gradient),
- \(\beta\) is the momentum term (e.g., 0.9).

**Contribution:**
Momentum builds upon SGD by accelerating the gradient descent in relevant directions and smoothing updates. This helps dampen the oscillations caused by noisy updates, leading to faster convergence, especially in deep valleys of the loss landscape. The key idea is that it accumulates past gradients to increase the update step in the correct direction.

---

### 3. **RMSprop (Root Mean Square Propagation)**

**Equation:**
\[
s_{t+1} = \beta s_t + (1 - \beta) (\nabla_{\theta_t} J(\theta_t))^2
\]
\[
\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{s_{t+1} + \epsilon}} \nabla_{\theta_t} J(\theta_t)
\]
Where:
- \(s_t\) is a running average of squared gradients,
- \(\epsilon\) is a small constant to prevent division by zero.

**Contribution:**
RMSprop divides the learning rate by the square root of the running average of squared gradients. This results in adaptive learning rates, where frequently updated parameters get smaller learning rates. This reduces the problem of overshooting the minima, especially in scenarios with steep gradients. RMSprop helps stabilize convergence by scaling learning rates based on the gradient's magnitude.

---

### 4. **Adam (Adaptive Moment Estimation)**

**Equation:**
\[
m_{t+1} = \beta_1 m_t + (1 - \beta_1) \nabla_{\theta_t} J(\theta_t)
\]
\[
v_{t+1} = \beta_2 v_t + (1 - \beta_2) (\nabla_{\theta_t} J(\theta_t))^2
\]
\[
\hat{m}_{t+1} = \frac{m_{t+1}}{1 - \beta_1^t}, \quad \hat{v}_{t+1} = \frac{v_{t+1}}{1 - \beta_2^t}
\]
\[
\theta_{t+1} = \theta_t - \frac{\eta}{\sqrt{\hat{v}_{t+1}} + \epsilon} \hat{m}_{t+1}
\]
Where:
- \(m_t\) and \(v_t\) are estimates of the first and second moments of the gradient, respectively.

**Contribution:**
Adam combines momentum and RMSprop by incorporating both a running average of the gradient (momentum) and a running average of squared gradients (adaptive learning rate). This balances the benefits of momentum and RMSprop, making it well-suited for noisy datasets and sparse gradients. The moment corrections (\(\hat{m}_{t+1}\), \(\hat{v}_{t+1}\)) account for initialization bias, improving performance in the early training steps. Adam is widely used because of its robust performance across different types of neural networks.

---

### Summary of Improvements:
- **SGD**: Simple, but suffers from noisy updates.
- **SGD with Momentum**: Adds momentum to smooth updates and speed up convergence.
- **RMSprop**: Introduces adaptive learning rates, stabilizing training.
- **Adam**: Combines momentum and adaptive learning rates, making it versatile and effective for most tasks.


> 1st moment can be loosely thought of as direction. it is mean/average but in a losse sense it alleviates the issue of noisy updates. 
> 2nd moment is variance (maybe loosely? see chat above). variacne is mean of $(X-\mu)^2$ and 2nd moment is mean of $X^2$. we devide by 2nd moment