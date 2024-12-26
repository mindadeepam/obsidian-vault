#activation-functions #relu #sigmoid

## why is relu better than sigmoid/tanh?

- Tanh, sigmoid require computing exponentials. Relu is.. Well ... Linear. It is a LOT faster to compute. It also solves a problem that sigmoid and tanh have that at large values grad(tanh(x)) is effectively 0 meaning there is no gradient flow to the network, same for sigmoid (which is just a shifted tanh). [link](https://www.reddit.com/r/learnmachinelearning/comments/ua6n6s/comment/i5w6jfg/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

- A big difference between relu and sigmoid is that when looked at from the perspective of gradients, it's essentially an on/off switch;
  
  "Is this node activated? Then gradients pass through via linear algebra. If not? Then no alteration of weights propagating back through this line"
  
  If you couple this with weight decay, then you can end up reinforcing activations if they have a positive effect, but otherwise diminishing weights the throughout the network, activated or not.
  
  And that's fine, because an over-parameterised network with a regulariser is probably what you want anyway, so the fact that sticking in an l1 or l2 regulariser helps deal with the potential dying relu or exploding weights problem (by dealing with large negative biases and large magnitude weights respectively), while also helping you with your generalisation is pretty handy.
  
  Instead of a gradual loss of sensitivity on a very deep network, you get very different training sensitivities on each update, but a full ability to propagate the signal backwards through whatever sub-network activates, however deep it is. [link](https://www.reddit.com/r/learnmachinelearning/comments/ua6n6s/comment/i5w8nc8/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)