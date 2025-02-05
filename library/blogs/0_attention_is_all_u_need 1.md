# Why is Attention is all you need?


In this blog, I am going to implement the paper [Attention is all you need](https://arxiv.org/pdf/1706.03762.pdf). It was a pivotal paper in the field of deep learning and has been used as a backbone for today's most popular SOTA models. But lets take a look back at how we got to this point.

## Language Modelling and Machine Translation

Understanding Language has been a long standing problem in the field of AI, falling under the umbrella of Natural Language Processing (NLP). ALthough models today are almost better than humans on most NLP "benchmarks", this was not the case pre-2018. Among the different problems in NLP, Language translation was one of the most important one. For the longest time Statistical Machine Translation (SMT) models were used, which was based on purely statistical methods. Other methods like Rule-Based Translation (RBT) and N-gram were also used. But they all suffered from the problem of being brittle and not generalizable, required manual feature engineering and were not able to handle long sentences.

But once deep-learning started flourishing, efforts were made to replace the earlier statistical methods with neural networks. 

Understanding Language is tightly linked with the problem of word prediction. What is word prediction anyways? Its the problem of predicting the next word in a sequence, given the previous words. i.e suppose we take a text corpus of several physics books. For a sentence like "light tranvels faster than ____", we want to predict the next word. ie we want to find $P(w|h)$ where $h$ is the history of the sentence and w is the next word. One way to do it would be to count that of the total number of times this phrase occurs in the corpus, and out of those times what was the distribution of the following word. You're likely to find that "sound" has the highest probability. 

In this case, we are correct. But this is a statistical approach that doesn't capture the context of the sentence.
$$
P(sound|\text{light tranvels faster than}) = \frac{C(\text{light tranvels faster than sound})}{C(\text{light tranvels faster than})}
$$

The assumption that the current word only depends on what came before it is called the Markov assumption. Using the chain rule, we can express the probability of an entire sentence as the product of individual conditional probabilities of all the words in the sentence. Generalizing, a statistical model of language can be represented by the conditional probability of the next word given all the previous ones, since
$$
P(w_1^T) = P(w_1)P(w_2|w_1)P(w_3|w_{1:2})...P(w_t|w_{1:t-1}) = \prod_{t=1}^{T} P(w_t | w_1^{t-1})
$$

N-gram models work on the assumption that the probability of the next word is only dependent on the previous $n$ words, thus simplyfying a lot of the calculations ie 
$$
P(w_1^T) = P(w_t|w_{1}^{t-1}) \approx P(w_t|w_{t-n+1}^{t-1}) =  \prod_{t=1}^{T} P(w_t | w_{t-n+1}^{t-1})
$$




Yoshua Bengio etal. in 2003 [1] proposed a neural network based language model that could predict the next word in a sequence. This was the first step towards replacing the statistical methods with neural networks that had distributed representation of words. Now what does that mean? Till now we have treated each word as a discrete entity. But in a language, words are not independent of each other. They have a lot of context and meaning. Some words are more similar to each other than others. N-gram models also miss the context of the entire sentence, focussing on the last $n$ words.   

> "here we push this idea to a large scale, and concentrate on learning a statistical model of the distribution of word sequences, rather than learning the role of words in a sentence"  

Having distributed representation of words [3] means that the words are represented by a dense vector of real numbers, instead of a one-hot encoded vector. The probabilty function of word sequences is then modelled as a function of these word vectors. Both the probabilty function parameters and the representation are learned, which means that the model learns the representation of the words from the data This helps mitigate high dimensionality and sparsity of the data and allows the model to capture the context of the sentence and the relationship between words. The idea of learning distributed representations of words was had been done by  Bellegarda (1997) ealier but they used a statistical n-gram model.

In their work, Bengio etal trained a feed forwrd network to jointly learn both the word vector represenations and the statistical language model. 

Then around 2013-14, landmark papers like word2vec and glove [4,5] showed us better ways to learn word representation vectors. Most of the progress here came from better model architectures via more intuitive biases and a larger data size.

But apart from good word representations, we also need real world use cases like genersting sequences of words. Machine Translation was one of the hottest problems in this field at the time. Statistical Machine Translation (SMT) models were the most popular ones. Going deeper into that is out of scope for this blog, refer to this [blog](https://mardiyyah.medium.com/statistical-machine-translation-a-data-driven-translation-approach-8a5dc15ba057) for more details.

Cho etal. in 2014 [6] proposed a seq2seq model that used RNNs to model phrase pairs as part of a SMT model. Their model consists of two recurrent neural networks (RNN) that act as an encoder and a decoder pair. The encoder maps a variable-length source sequence to a fixed-length vector, and the decoder maps the vector representation back to a variable-length target sequence. The two networks are trained jointly to maximize the conditional probability of the target sequence given a source sequence. 

Let the input sequence be $x_1^T$ and the output sequence be $y_1^T$. 
At each time step $t$, the encoder RNN takes the input $x_t$ and outputs a hidden state $h_t$ using: 

$$
h_t = f(x_t,h_{t-1})
$$

where $f$ is a non-linear function.
The final hidden state $h_T$ is used as the context vector $c$ for the decoder.
The decoder RNN is different as for each hidden state $h_t$ it takes as input the previous hidden state, output as well as the encoder context vector $c$.

$$
h_t = f(y_{t-1}, h_{t-1}, c)
$$

then using this hidden state and the previous output $y_{t-1}$ the decoder outputs the next word $y_t$ using:

$$
y_t = g(h_t, y_{t-1}, c)
$$


<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/rnn_enc_dec.png" alt="rnn_enc_dec" width="300" height="300" />
     <figcaption>Figure 1: RNN encoder decoder model.</figcaption>
</center>  


The model is trained to maximize the conditional probability of the target sequence given the source sequence: 
$$
\max_{\theta} \frac{1}{N} \sum_{t=1}^{N} \log p_\theta(y_t | x_t) 
$$
where $x_t$ is the source sequence, $y_t$ is the target sequence, and $\theta$ represents the model parameters. In this paper the authors actually used their interpretation of gating like LSTMs as the usual activation of tanh didnt yield meaningful results. 

This looks like a solid foundation for end to end sequence modelling, it was just a part of a phrase based SMT system and was trained on phrase pairs of English and French. Papers like [7,8,9] advanced the field of neural machine translation where the entire translation was done by the neural model itself. Sutskever et al., 2014 [7] used LSTMs and achieved near SOTA for machine translation tasks. Their model was a 4 layer LSTM with 384 million parameters. They found that reversing the input sentences helped the model learn the translation better and to mitigate exploding graidents they enforced a hard constraint on the norm of the gradient. When used in a SMT system, the model was able to achieve SOTA results. 

Bahadanu et al. in 2014 [8] identified the context vector as a bottleneck and introduced the concept of **attention** to the seq2seq model.

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/lil-weng-attantion-fig-4.png" alt="seq2seq" width="500" height="300" />
     <figcaption>Figure 2: Seq2Seq model with attention.</figcaption>
 </center>

In the new architecture, the encoder is the same as before, but the decoder is now modified to take the a distinct context vector $c_t$ along with the previous hidden state $h_{t-1}$ and the previous output $y_{t-1}$ to generate the next hidden state $h_t$. The context vector $c_t$ is computed as a weighted sum of the encoder hidden states $h_i$, as opposed to the previous model where the context vector was a fixed length vector. 
$$ 
c_i = \sum_{j=1}^{T_x} \alpha_{ij} h_j 
$$

The context vector $c_i$ is a weighted sum of the encoder hidden states $h_j$. The weight $\alpha_{ij}$ for each hidden state $h_j$ is computed as:

$$
\alpha_{ij} = \frac{exp(e_{ij})}{\sum_{k=1}^{T_x} exp(e_{ik})}
$$

where $e_{ij}$ is an alignment score calculated by:

$$
e_{ij} = a(s_{i-1}, h_j)
$$

The alignment model $a$ scores how well the input at position $j$ matches the output at position $i$. This score is based on the decoder's previous hidden state $s_{i-1}$ and the encoder's hidden state $h_j$. Intuitively, these weights allow the decoder to implement a soft attention mechanism, where the decoder can focus on different parts of the input sequence at different times. The alignment model $a$ is implemented as a feedforward neural network and is trained jointly with the rest of the model. 
As expected, the model with attention outperforms the model without attention especially for longer sentences. (see fig 3) 

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/learning-to-jointly-align-and-translate-fig-2.png" alt="performance comparison of models with and without attention" width="600" height="300" />
     <figcaption>Figure 3: Performance comparison of models with and without attention.[8]</figcaption>
</center>
<br>


## But…

- Sequential computation inhibits parallelization.
- No explicit modeling of long and short range dependencies.
- We want to model hierarchy.
- RNNs (w/ sequence-aligned states) seem wasteful!

## Transformers!

Following Bahdanu's work, several attempts were made to improve on this attention mechanism and counter the various limitations of RNNs at the same time. Following the discussion on the paper we will move on to implement it and understand the different parts of it. 

What is attention really? Intuitively you might say that to be able to "attend" means to be able to choose. As lil weng says 
> In a nutshell, attention in deep learning can be broadly interpreted as a vector of importance weights: in order to predict or infer one element, such as a pixel in an image or a word in a sentence, we estimate using the attention vector how strongly it is correlated with (or “attends to” as you may have read in many papers) other elements and take the sum of their values weighted by the attention vector as the approximation of the target.

One thing it solves is that we can now attend to longer sequences. And these "weights" can be obtained by various methods ranging from simple cosine similarities to neural layer/networks of relevant features.
 
Coming back to transformers, Vasqani et al. in 2017 [10] proposed the transformers architecture which relied solely on attention mechanisms (and other things, but not RNN units). Through this we were able to parallelize the training of the model, since the model was able to process the entire sequence at once. This was a huge step forward in the field of NLP, since along with advancements in hardware, it allowed us to train models on much larger datasets and with much larger models. 


A transformer takes as input a variable length sentence and outputs a variable length sentence, ie its a sequence to sequence model.

Lets take the french sentence "je suis étudiant" as an example. Its english translation is "i am a student".
<center>
     <img src="https://jalammar.github.io/images/t/the_transformer_3.png" alt="transformer" width="600" height="150" />
     <!-- <figcaption>Figure 4: The Transformer model.</figcaption> -->
</center>  
Similar to the seq2seq models of the past, the transformer model is made up of an encoder and a decoder block.
<center>
     <img src="https://jalammar.github.io/images/t/The_transformer_encoders_decoders.png" alt="transformer" width="500" height="250" />
     <!-- <figcaption>Figure 4: The Transformer model.</figcaption> -->
</center>  

Lets start unboxing these blocks to find out how they work. 
Do not be overwhelmed by the following image, but this is what the transformer model looks like internally. 

<center>
     <img src="https://arxiv.org/html/1706.03762v7/extracted/1706.03762v7/Figures/ModalNet-21.png" alt="transformer" width="400" height="500" />
     <figcaption>Figure 4: The Transformer model.</figcaption>
</center>  

But we are going to start with just a string of words.

## Tokenization

$$
\text{Input: } \text{'je suis étudiant'}
$$
The way the model is able to understand a string input is by breaking down the string into multiple "tokens" and mapping each token to a unique integer. For eg if we consider only character tokens, our sentence has the following tokens: 
$$
\text{vocab\_char} = \{\text{'j'}, \text{'e'}, \text{' '}, \text{'s'}, \text{'u'}, \text{'i'}, \text{'é'}, \text{'t'}, \text{'d'}, \text{'a'}, \text{'n'}, \text{'SOS'}, \text{'EOS'}, \text{'PAD'}\}
$$

We also add some special tokens to the vocab like 

- the start of sequence token (<SOS>): denotes the start of the sentence.
- end of sequence token (<EOS>): denotes the end of the sentence.
- the padding token (<PAD>): denotes the padding of the sentence.
- the unknown token (<UNK>): denotes the unknown token.

The total number of tokens make up the vocabulary of the model.  If we do word level tokenozation, we get the follwoing vocab:

$$
\text{vocab\_word} = \{\text{'je'}, \text{'suis'}, \text{'étudiant'}, \text{' '}, \text{'SOS'}, \text{'EOS'}, \text{'PAD'}\}
$$

There is a slight nuance to consider to manage compute when considering the vocabulary size. Character level tokenization would yield a much smaller vocab, but would end up making the sequence length much longer. On the other hand word level tokenization would yield a larger vocab, but would end up making the sequence length much shorter. The tokens in this paper are subword units which are selected through the Byte Pair Encoding (BPE) algorithm. Essentially the most frequent subword units are selected and used as tokens. Tokenization algirithms is a separate field of study and there are many other algorithms like WordPiece, SentencePiece, etc. that are used in the industry. Here we'll just treat tokenizer like a black box function that takes in a string and returns a list of tokens. 

Obviously the model doesnt understand the token strings, hence we map the tokens to a unique integer. For simplicity in this blog lets consider the vocabulary word level of size 30k same as the original paper, which means we have a dictionary of size 30k that maps each token to a unique integer. Since we actually train in batches, we need to pad the sequence to the same length, so that we can batch them together. The maximum sequence length ($L$) is 512 tokens. Longer seqeunces are truncated and shorter ones are padded to get the same sequence length. We'll ignore the special tokens in the diagrams.

Since the first transformer paper the vocab size has grown from 30k to 100k-200k tokens. While sequence length has grown from 512 in the attention paper to millions of tokens in the SOTA models of 2024. 

$$
\text{Input: } [\text{'SOS'}, \text{'je'}, \text{'suis'}, \text{'étudiant'}, \text{'EOS'}]  \\
\text{Padded input: } [\text{'SOS'}, \text{'je'}, \text{'suis'}, \text{'étudiant'}, \text{'EOS'}, \text{'PAD'}, \text{'PAD'} ... ]  \\ 
\text{Tokenized input: } [100, 101, 102, 103, 104]  
$$

## Embedding Layer

After tokenization comes the embedding layer. The embedding layer is a layer with learnable parameters which maps each token to a dense vector of fixed length, which is the embedding dimension, $d_{model}$. In the attention paper, various values of $d_{model}$ were tried, but the most common one was $d_{model}=512$. Hence the embedding layer is a matrix of shape $L \times d_{model}$ that adds a dimension to the input sequence.


$$
\text{Input: } \mathbb{Z}^{L}
$$
$$
\text{Output: } \mathbb{R}^{L \times d_{model}}
$$

Syntactically what matters is that a array of numbers now becomes a matrix of vectors since each token is mapped to a vector. 

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/illustrated-transformers-fig-12312.png" alt="embedding" width="600" height="100" />
     <figcaption>Each word is embedded into a vector of size 512.</figcaption>
</center>  

Semantically embedding layers [1,3] allow us to learn a dense representation of the input sequence. Baseline methods like one-hot encoding will embed each token to a vector of size equal to the vocab size. Apart from making training efficient by reducing the dimensionaly of the embedding layer, it also allows us to learn semantic relationships between words. By making the training objective such that similar words have similar embeddings, we learn an embedding space that is a learned, geometric representation of language where the position of a word vector reflects its meaning and its relationship to other words, based on how those words are used in context. 

You might have noticed the entire input sequence is being passed through the embedding layer. So how does the model know the order of the tokens in the sequence? For that, we then add **positional encodings** to the input embeddings. For now imagine that the positional encodings are a predefined (ie not learned via backprop) matrix of shape $L \times d_{model}$ that contains the positional information of each token in the sequence. So each index is just mapped to a unique vector which is added elementwise to the embedding vector of that token to produce a new matrix of shape $L \times d_{model}$.


## Encoder block

So now we have as input a matrix of shape $L \times d_{model}$ that represents the input sequence. This input tensor now goes through the encoder which consists of 6 layers of "encoder blocks". Each encoder block is made up of two sublayers: a multi-head self-attention mechanism and a position-wise feed-forward neural network. 
The abstraction that is common to all the encoders blocks is that they receive a list of vectors each of the size $L$. In the bottom encoder that would be the word embeddings, but in other encoders, it would be the output of the encoder that’s directly below it. 

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/attention-is-all-u-need-encoder.png" alt="encoder" width="200" height="300" />
     <figcaption>An Encoder block</figcaption>
</center>  

Lets now look at the first sublayer of the encoder block, the multi-headed self attention mechanism.

## Self attention

We saw in [8] what advantage attention gives us over non-attention based models. Applying attention means to be able to choose where to attend to in the input sequence. Self attention is the process of attending to the input sequence itself. We need self attention because without it the tokens wont be able to communicate with each other.   

See the following intuitive explanation of self attention from [12]:   

Say the following sentence is an input sentence we want to translate: ”The animal didn't cross the street because it was too tired”.

What does “it” in this sentence refer to? Is it referring to the street or to the animal? It’s a simple question to a human, but not as simple to an algorithm.

When the model is processing the word “it”, self-attention allows it to associate “it” with “animal”. As the model processes each word (each position in the input sequence), self attention allows it to look at other positions in the input sequence for clues that can help lead to a better encoding for this word.

If you’re familiar with RNNs, think of how maintaining a hidden state allows an RNN to incorporate its representation of previous words/vectors it has processed with the current one it’s processing. Self-attention is the method the Transformer uses to bake the “understanding” of other relevant words into the one we’re currently processing.
<center>
     <img src="https://jalammar.github.io/images/t/transformer_self-attention_visualization.png" alt="transformer" width="350" height="400" />
     <figcaption>Fig: Self attention</figcaption>
</center>  

The two most commonly used attention functions are additive attention and dot-product (multiplicative) attention. Additive attention computes the compatibility function using a feed-forward network with a single hidden layer. While the two are similar in theoretical complexity, dot-product attention is much faster and more space-efficient in practice, since it can be implemented using highly optimized matrix multiplication code.


Lets assume the input tensor $x$
$$ 
x = [{x}_1, {x}_2, {x}_3, ... {x}_{L}] 
$$
where $x_i$ is the $i^{th}$ token's embedding vector of length $d_{model}$. Hence, 
$$ 
x \in \mathbb{R}^{L \times d_{model}} 
$$
$$ 
x_i \in \mathbb{R}^{d_{model}} 
$$

Every input token vector $𝐱_i$ is used in three different ways in the self attention operation:
- It is compared to every other vector to establish the weights for its own output $𝐲_i$
- It is compared to every other vector to establish the weights for the output of the $j^{th}$ vector $𝐲_j$
- It is used as part of the weighted sum to compute each output vector once the weights have been established.

These roles are often called the query, the key and the value (we'll explain where these names come from later). In the basic self-attention we've seen so far, each input vector must play all three roles.
We project the same input vector (denoted as $\mathbb{x_i}$) into 3 different vectors, query $(q_i)$, key $(k_i)$ and value $(v_i)$, by multiplying it with 3 different weight matrices $W_k$, $W_q$ and $W_v$ of shape $d_{model} \times d_{model}$.

$$ 
k_i = W_k \mathbb{x_i} \quad q_i = W_q \mathbb{x_i} \quad v_i = W_v \mathbb{x_i} 
$$

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/illustrated-transformer-fig-21432.png" alt="encoder" width="600" height="350" />
     <figcaption>Fig: token level view of self attention. Each vector represents one token in the input sequence.</figcaption>
</center>  

Note that the projection matrices are applied across the embedding dimension. That is, **each token vector ($x_i \in \mathbb{R}^{d_{model}}$) in an input sequence shares the parameters of the projection matrices** ($W_q, W_k, W_v \in \mathbb{R}^{d_{model} \times d_{model}}$).  

To understand further we will have to switch to the matrix view of the input tensor.

### Matrix view: 

The entire input sequence matrix $X \in \mathbb{R}^{L \times d_{model}}$ when multiplied by the projection matrices results in output matrices of shape $L \times d_{model}$.
$$
Q = XW_q \quad K = XW_k \quad V = XW_v \quad \text{where } Q, K, V \in \mathbb{R}^{L \times d_{model}}
$$

Now we do scaled dot product attention using the $Q$, $K$ and $V$ matrices. First we take the dot product of $Q$ and $K^T$ to get the attention scores $A'$. The dot product is also scaled by the square root of the dimension of the vectors ie $d_{model}$. This is done because the dot product of a vector is directly proportional to the length of the vectors, so by scaling down the dot product we prevent the scores from becoming too large and causing numerical instability.

<!-- Each element of $A'$ is the dot product of the $i^{th}$ query vector and the $j^{th}$ key vector and denotes the attention score between the $i^{th}$ token and the $j^{th}$ token. -->

$$
A' = \frac{QK^T}{\sqrt{d_{model}}} \quad  \text{where } A' \in \mathbb{R}^{L \times L} 
$$

Finally we take the softmax of the attention scores to get the attention weights $A$ that correspond to probabilities.

$$
A = \text{softmax}(\frac{QK^T}{\sqrt{d_{model}}}) \quad  \text{where } A \in \mathbb{R}^{L \times L}
$$

We can now take the dot product of the attention matrix $A$ and the value matrix $V$ to get the output matrix $Y$.

$$
Y = \text{softmax}(\frac{QK^T}{\sqrt{d_{model}}})V
$$

where $Y \in \mathbb{R}^{L \times d_{model}}$ ie same shape as the input.

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/illustrated-transformer-fig-6543.png" alt="encoder" width="300" height="350" />
     <figcaption>Fig: Matrix view of self attention. Every row in the X matrix corresponds to a word in the input sentence.</figcaption>
</center>  

### Token level view:

For $a_{ij}$ the attention score between the $i^{th}$ token and the $j^{th}$ token, it is computed using the scaled dot product of the $i^{th}$ query vector and the $j^{th}$ key vector. **This is where the inter-token communication b/w tokens i and j happens.**

$$
a_{ij} = \frac{q_i \cdot k_j}{\sqrt{d_{model}}}
$$

We finally obtain output vectors $y_i$ by taking the weighted sum of the value vectors $v_j$ using the attention weights $a_{ij}$. Now, each token has now looked at all the other tokens and updated itself.

$$
y_i = \sum_{j=1}^{L} a_{ij} v_j
$$

ie 
$$
y_i = [|q_1k_1|*v1 + |q_1k_2|*v2 + |q_1k_3|*v3 + |q_1k_4|*v4 + .. |q_1k_{L}|*v_{L}]
$$

where $+$ is the elementwise product.

This diagram should now be very easy to understand. (Masking will be covered later in the decoder section)
<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/attention-is-all-u-need-sa.png" alt="encoder" width="200" height="230" />
     <figcaption>Fig: Scaled Dot product attention</figcaption>
</center>  

## Multi-headed Attention (MHA)

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/attention-is-all-u-need-mha.png" alt="encoder" width="200" height="300" />
     <figcaption>Fig: Multi-headed attention</figcaption>
</center>  

Since each token depends on each other in the the scaled-dot product attention layer, it cant be parallelized. To get around this, we use multiple attention heads. Mulitple heads also gives the attention layer multiple “representation subspaces” hence learning multiple types of relationships between tokens and creating a more robust and expressive representation of the input sequence.  
Lets say the number of attention heads is $h$ (in the paper $h=8$). We project input tensor from length $L$ to $L/h$ for each attention head. $h$ such projections are done in parallel. and their result is concatenated to get back the original length $L$. This is done by making the projection matrices $W_q, W_k, W_v$ of shape $d_{model} \times \frac{d_{model}}{h}$. Hence for the $i^{th}$ attention head, we have the following:  

$$
X \in \mathbb{R}^{L \times d_{model}} 
$$
$$ 
W_{qi} \in \mathbb{R}^{d_{model} \times \frac{d_{model}}{h}} 
$$
$$ 
Q_i = XW_{qi} \quad \text{where } Q_i \in \mathbb{R}^{L \times \frac{d_{model}}{h}} 
$$
$$
K_i = XW_{ki} \quad \text{where } K_i \in \mathbb{R}^{L \times \frac{d_{model}}{h}} 
$$
$$
V_i = XW_{vi} \quad \text{where } V_i \in \mathbb{R}^{L \times \frac{d_{model}}{h}} 
$$
$$
A_i = \text{softmax}(\frac{Q_iK_i^T}{\sqrt{d_{model}}}) \quad \text{where } A_i \in \mathbb{R}^{L \times L}
$$
$$ 
Y_i = A_iV_i \quad \text{where } Y_i \in \mathbb{R}^{L \times \frac{d_{model}}{h}}
$$


The output of the attention heads are concatenated and passed through a linear layer to produce the final output.

$$
Y = \text{concat}(Y_1, Y_2, ..., Y_h)W_o \quad \text{where } Y \in \mathbb{R}^{L \times d_{model}}, W_o \in \mathbb{R}^{d_{model} \times d_{model}}
$$

$W_o$ is the output projection matrix. It allows some computation after concatenation of outputs from different attention heads.  

Looking at everything in the self attention block, we have:

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/illustrated-transformer-fig-encoder-full-detailed.png" alt="encoder" width="700" height="400" />
     <figcaption>Fig: Expanded view of the self attention block</figcaption>
</center>  


## Residual Connections

Residual connections are used to mitigate vanishing gradients in deep neural networks. They were first introduced in [Deep Residual Learning for Image Recognition](https://arxiv.org/pdf/1512.03385) and have since been used in almost all deep neural networks.

Mathematically, a residual connection is a shortcut connection that allows the network to bypass some layers and directly influence the output of other layers. This is done by adding the output of a layer to the input of the same layer.

$$
y = x + f(x)
$$

where $x$ is the input to the layer, and $f(x)$ is the output of the layer.  
In transformers, 
$$
\text{MHA}(x) = x + \text{MHA}(x)
$$

## Layer Normalization

In CNNs we employ batch normalization to normalize the activations of each layer across the batch dimension. In batch normalization, for a batch of shape $B, H, W, C$, the mean $\mu$ and standard deviation $\sigma$ are calculated as follows:

$$
\mu = \frac{1}{HWB} \sum_{h=1}^{H} \sum_{w=1}^{W} \sum_{b=1}^{B} x_{b,h,w} \quad \text{and} \quad \sigma = \sqrt{\frac{1}{HWB} \sum_{h=1}^{H} \sum_{w=1}^{W} \sum_{b=1}^{B} (x_{b,h,w} - \mu)^2}
$$

$$ 
y_{b,h,w,c} = \gamma \frac{x_{b,h,w,c} - \mu}{\sigma} + \beta
$$

So during training in CNNs, we take mean and std across the batch dimension (its either cross features or across samples, bathc denotes samples. In both cases we also take mean and std across the spatial dimensions(H,W or seq_len)) then subtract each channel value by the mean and divide by the std. This is done for each channel in the batch. To let the model still be able to model complex patterns, we add learnable parameters $\gamma$ and $\beta$ to the normalized values.
During inference, we use the average mean and std calculated during training to normalize the activations.

Note that the mean and standard deviation are calculated across the $H$, $W$, and $B$ dimensions for each channel $C$.
But this would hinder parallization in transformers. So instead we use layer normalization. In layer normalization, we take mean and std across the feature dimension (ie $H,W,C$ or $L,d_{model}$) then subtract each channel value by the mean and divide by the std.
$$
\mu = \frac{1}{HWC} \sum_{h=1}^{H} \sum_{w=1}^{W} \sum_{c=1}^{C} x_{h,w,c} \quad \text{and} \quad \sigma = \sqrt{\frac{1}{HWC} \sum_{h=1}^{H} \sum_{w=1}^{W} \sum_{c=1}^{C} (x_{h,w,c} - \mu)^2}
$$

Similarly in transformers, we take mean and std across the feature dimension (ie $L,d_{model}$) then subtract each value by the mean and divide by the std.
$$
\mu = \frac{1}{L \cdot d_{model}} \sum_{l=1}^{L}\sum_{d=1}^{d_{model}} x_{l,d} \quad \text{and} \quad \sigma = \sqrt{\frac{1}{L \cdot d_{model}} \sum_{l=1}^{L} \sum_{d=1}^{d_{model}} (x_{l,d} - \mu)^2}
$$

$$
y_{l,d} = \gamma \frac{x_{l,d} - \mu}{\sigma} + \beta
$$

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/batch-norm-layer-norm.png" alt="encoder" width="400" height="200" />
     <figcaption>Fig: Batch norm vs layer norm</figcaption>
</center>  


- [Batch normalization](https://arxiv.org/pdf/1502.03167) and [layer normalization](https://arxiv.org/pdf/1607.06450) both help in stabilizing the training process. This is because without normalization, the activations can become too large or too small, and variance between batches (internal covariate shift) or samples can cause the network to become unstable and difficult to train. Normalization also acts as a regularizer and helps in preventing overfitting. 
- **Learnable Parameters:** Both γ and β are *learnable* parameters. This means that the network can adjust them during training through backpropagation, just like the weights and biases of the layers.
- **Undoing Normalization:** In the extreme case, the network can learn to set γ to the standard deviation of the original activations and β to the mean of the original activations. This would effectively "undo" the normalization and restore the original distribution if that's what's best for the task.  

Here is an an excellent and intuitive explanation from [Pramod Goyal's excellent blog on transformers](https://goyalpramod.github.io/blogs/Transformers_laid_out/#understanding-the-encoder-and-decoder-block)

<center>
     <img src="https://raw.githubusercontent.com/mindadeepam/obsidian-vault/main/media/pramod-transformers-sa-intuition.png" alt="encoder" width="600" height="500" />
     <figcaption>Fig: Self attention exlained, Transformers Laid Out - Pramod Goyal</figcaption>
</center>    

**As demonstrated self-attention is a linear transformation.**

Now we have the final equation for the first half of the encoder block.

$$
\text{MHA}(x) = \text{LN}(\text{MHA}(x) + x)
$$

---

## WIP

- **Point-wise Feed-Forward Network**

- **Dec block**
    
  - recurrence
  - masked self attention
  - cross attention
  - diagrams

- **Positional Encoding**


- **Qs:**
  - why did we remove recurrance?
  - what is attention?
  - one token, sentence and batch visualizations.
  - what is self attention? 
  - why multiplicative attention instead of additive?
  - why kqv? 
  - why multihead?
  - Here we begin to see one key property of the Transformer, which is that the word in each position flows through its own path in the encoder. There are dependencies between these paths in the self-attention layer. The feed-forward layer does not have those dependencies, however, and thus the various paths can be executed in parallel while flowing through the feed-forward layer ?????????/ 
  - code implementation in beginning or end? code does make it ugly fo-sho.

- **[13]**
  - According to Vaswani himself [13], the main advantage of the transformer architecture is that it is parallelizable. hence it makes training faster (attention paper reported 3* faster than similar sized RNN based models, this is because the attention is a simple linear operation. this makes it favourable to train via backprop/SGD).
  - he also says that if diff heads focused locally (based on posituib, like convolution does) instead of globally (globally but in lower dims) and if theere were many heads it would be like a convolution.

## References 
<small>
📕: must read  
📘: good to read  
📓: if time permits  
📙: just noting for use, if ever  
</small>


[1] [A Neural Probabilistic Language Model - Bengio et al, 2003](https://www.jmlr.org/papers/volume3/bengio03a/bengio03a.pdf) 📕  
[2] [Standford Speech and Language Processing 3rd edition, 2024](https://web.stanford.edu/~jurafsky/slp3/3.pdf) 📘  
[3] [Learning Distributed Representations of Concepts - Hinton et al, 1986](https://www.cs.toronto.edu/~hinton/absps/families.pdf) 📕   
[4] [Efficient Estimation of Word Representations in Vector Space - Mikolov et al, 2013](https://arxiv.org/pdf/1301.3781) 📕  
[5] [GloVe: Global Vectors for Word Representation - Pennington et al, 2014](https://nlp.stanford.edu/pubs/glove.pdf) 📘  
[6] [Learning Phrase Representations using RNN Encoder–Decoder for Statistical Machine Translation - Cho et al, 2014a](https://arxiv.org/pdf/1406.1078) 📘  
[7] [Sequence to Sequence Learning with Neural Networks - Sutskever et al, 2014](https://arxiv.org/pdf/1409.3215) 📕  
[8] [Neural Machine Translation by Jointly Learning to Align and Translate - Bahdanau et al, 2014](https://arxiv.org/pdf/1409.0473) 📕  
[9] [On the Properties of Neural Machine Translation: Encoder–Decoder Approaches, Cho et al, 2014b](https://arxiv.org/pdf/1409.1259) 📘   
[10] [Attention Is All You Need, Vaswani et al, 2017](https://arxiv.org/pdf/1706.03762) 📕  
[11] [Attention? Attention! - Lilian Weng, 2018](https://lilianweng.github.io/posts/2018-06-24-attention/) 📘  
[12] [The Illustrated Transformer - Jay Alammar, 2018](https://jalammar.github.io/illustrated-transformer/) 📕   
[13] [Stanford CS224N: NLP with Deep Learning | Winter 2019 | Lecture 14 – Transformers and Self-Attention, Vaswani](https://www.youtube.com/watch?v=5vcj8kSwBCY) 📕  

---
Other resources:

- https://goyalpramod.github.io/blogs/Transformers_laid_out/#understanding-the-encoder-and-decoder-block   
- https://nlp.seas.harvard.edu/annotated-transformer/ code-led explanation
- https://jalammar.github.io/illustrated-transformer/ most famous blog on it ever ig
- https://peterbloem.nl/blog/transformers another OG
- [Attention? Attention! - Lilian Weng, 2018](https://lilianweng.github.io/posts/2018-06-24-attention/) : for a more indepth read on Attention mechanism and its usage elsewhere.
- [The transformer family - lilian weng, 2020](https://lilianweng.github.io/posts/2020-04-07-the-transformer-family/): explains transformers and further advancements on the architecture.
- https://lilianweng.github.io/posts/2023-01-27-the-transformer-family-v2/
- https://mehta-rohan.com/writings/blog_posts/attention.html
- https://www.youtube.com/watch?v=5vcj8kSwBCY: stanford guest lecture of transformers by vaswani himself



---

<details>
<summary>
These are some resoruces I picked on the way, to read later</summary>

[ ] [H. Schwenk. Continuous space language models. Computer Speech and Language, vol. 21, 2007](https://publi.limsi.fr/RS2005/chm/tlp/tlp11/) 📘
[ ] [T. Mikolov, A. Deoras, S. Kombrink, L. Burget, J. Cernock ˇ y. Empirical Evaluation and Com- ´
bination of Advanced Language Modeling Techniques, In: Proceedings of Interspeech, 2011.](https://www.isca-speech.org/archive/interspeech_2011/i11_0812.html) 📓
[ ] Attention, Attention! - Lilian Weng, [blog](https://lilianweng.github.io/posts/2018-06-24-attention/) 📘  
[ ] [Effective Approaches to Attention-based Neural Machine Translation, Luong et al, 2015](https://arxiv.org/pdf/1508.04025) 📘  



- [Parallel Distributed Processing, Volume 1: Explorations in the Microstructure of Cognition: Foundations](https://direct.mit.edu/books/monograph/4424/Parallel-Distributed-Processing-Volume) (book that contained [3] hinton paper.) read the book if time allows, not necessary. 📓
- attention origins: [Generating Sequences With Recurrent Neural Networks, 2014](https://arxiv.org/pdf/1308.0850) -> [Neural Machine Translation by Jointly Learning to Align and Translate, 2014](https://arxiv.org/pdf/1409.0473) -> [Attention is all you need, 2017](https://arxiv.org/pdf/1706.03762.pdf)
</details>

**neural machine translation**

<details><summary>Appendix</summary>

[1] Language model probabilities are always stored and computed in log space as log probabilities. This is because probabilities are (by definition) log probabilities less than or equal to 1, and so the more probabilities we multiply together, the smaller the product becomes. Multiplying enough n-grams together would result in numerical underflow. Adding in log space is equivalent to multiplying in linear space, so we combine log probabilities by adding them. By adding log probabilities instead of multiplying probabilities, we get results that are not as small. We do all computation and storage in log space, and just convert back into probabilities if we need to report probabilities at the end by taking the exp of the logprob:
$$p_1 \times p_2 \times p_3 \times p_4 = \exp(\log p_1 + \log p_2 + \log p_3 + \log p_4)$$




<details>
<summary>[2]</summary>
Q. chatgpt give me a detailed long report of main papers in nlp leading upto attention/gpt/bert, each paper u mention must be a significant milestone in nlp that  advanced the field in some way.

-> Okay, here is a detailed report tracing the significant milestones in NLP leading up to attention mechanisms, GPT, and BERT. This report focuses on key papers that advanced the field, particularly in the context of neural network-based approaches.

##### Early Foundations (Pre-2000s)

Before the neural network revolution, NLP relied heavily on:

*   **Rule-based systems:** Expert-crafted rules for tasks like parsing and translation.
*   **Statistical methods:** Techniques like n-grams, Hidden Markov Models (HMMs), and probabilistic context-free grammars (PCFGs) gained prominence.
*   **Machine learning:** Early ML algorithms like Naive Bayes and Support Vector Machines (SVMs) were applied to NLP tasks, often with hand-engineered features.

These methods, while effective to some extent, struggled with the nuances and complexities of human language.

##### The Rise of Neural Networks in NLP (2000s - 2013)

**1. A Neural Probabilistic Language Model (2003) - Bengio et al.**

*   **Significance:** This landmark paper introduced the concept of learning distributed representations of words (word embeddings) using a feedforward neural network.
*   **Contribution:**
    *   Proposed a neural architecture to learn word embeddings jointly with a language model.
    *   Showed that these learned embeddings captured semantic and syntactic similarities between words.
    *   Demonstrated improved performance over traditional n-gram models, especially in handling data sparsity.
*   **Impact:** This work laid the foundation for using neural networks in NLP and popularized the idea of word embeddings, which became a cornerstone of future advancements.

**2. Collobert and Weston's NLP (Almost) from Scratch (2008, 2011) - Collobert et al.**

*   **Significance:** Demonstrated the effectiveness of a unified neural network architecture for various NLP tasks.
*   **Contribution:**
    *   Proposed a convolutional neural network (CNN) architecture that could be applied to tasks like part-of-speech tagging, chunking, named entity recognition, and semantic role labeling.
    *   Showed that the network could learn meaningful features directly from raw text, reducing the need for hand-engineered features.
    *   Introduced a novel window approach and the use of a "tagger" network.
*   **Impact:** This work further solidified the potential of neural networks for NLP and showcased the power of CNNs in capturing local patterns in text.

**3. Recursive Neural Networks for Structure Prediction (2010-2013) - Socher et al.**

*   **Significance:** Introduced Recursive Neural Networks (RNNs) for processing tree-structured data, enabling better handling of syntactic structures.
*   **Contribution:**
    *   Proposed RNN architectures that could recursively process sentences based on their parse trees.
    *   Demonstrated their effectiveness in tasks like sentiment analysis and paraphrase detection.
    *   Introduced the concept of recursive compositionality in neural networks.
*   **Impact:** This work highlighted the importance of modeling syntactic structure in NLP and paved the way for more sophisticated tree-based neural models.

**4. Word2Vec (2013) - Mikolov et al.**

*   **Significance:** Introduced highly efficient methods for learning high-quality word embeddings.
*   **Contribution:**
    *   Proposed two novel architectures: Continuous Bag-of-Words (CBOW) and Skip-gram.
    *   Showed that these models could learn word embeddings that captured semantic relationships (e.g., "king" - "man" + "woman" ≈ "queen").
    *   Released pre-trained word embeddings that became widely used in the NLP community.
*   **Impact:** Word2Vec significantly advanced the field by providing a fast and effective way to learn high-quality word embeddings, which became a standard component in many NLP systems.

##### The Era of Sequence-to-Sequence Models and Attention (2014-2017)

**5. Sequence to Sequence Learning with Neural Networks (2014) - Sutskever et al.**

*   **Significance:** Introduced the sequence-to-sequence (seq2seq) model for machine translation.
*   **Contribution:**
    *   Proposed an encoder-decoder architecture using two Long Short-Term Memory (LSTM) networks.
    *   The encoder processed the input sequence into a fixed-length vector.
    *   The decoder generated the output sequence from this vector.
*   **Impact:** This work marked a major shift towards end-to-end neural machine translation and demonstrated the power of LSTMs in handling sequential data. However, it struggled with long sequences due to the fixed-length vector bottleneck.

**6. Neural Machine Translation by Jointly Learning to Align and Translate (2014) - Bahdanau et al.**

*   **Significance:** Introduced the attention mechanism to overcome the limitations of the basic seq2seq model.
*   **Contribution:**
    *   Proposed an attention mechanism that allowed the decoder to focus on different parts of the input sequence during translation.
    *   Enabled the model to handle longer sequences more effectively.
    *   Improved translation quality significantly.
*   **Impact:** This was a groundbreaking paper that revolutionized neural machine translation and introduced the concept of attention, which became a fundamental building block of modern NLP architectures.

**7. Effective Approaches to Attention-based Neural Machine Translation (2015) - Luong et al.**

*   **Significance:** Refined and extended the attention mechanism.
*   **Contribution:**
    *   Proposed different attention mechanisms, including "global" and "local" attention.
    *   Introduced different scoring functions for calculating attention weights.
    *   Further improved translation performance.
*   **Impact:** This work solidified the importance of attention in NMT and provided a more comprehensive understanding of different attention variants.

**8. Attention Is All You Need (2017) - Vaswani et al.**

*   **Significance:** Introduced the Transformer architecture, which relied solely on attention mechanisms.
*   **Contribution:**
    *   Proposed a novel architecture that eliminated the need for recurrent or convolutional layers.
    *   Used multi-head self-attention to capture relationships between different parts of the input sequence.
    *   Achieved state-of-the-art results in machine translation and other NLP tasks.
*   **Impact:** This was a revolutionary paper that transformed the field of NLP. The Transformer architecture became the dominant paradigm for many NLP tasks due to its efficiency, parallelizability, and ability to capture long-range dependencies.

##### The Rise of Pre-trained Language Models (2018-Present)

**9. Improving Language Understanding by Generative Pre-Training (GPT) (2018) - Radford et al.**

*   **Significance:** Introduced the Generative Pre-trained Transformer (GPT) model.
*   **Contribution:**
    *   Pre-trained a large Transformer model on a massive text corpus using a language modeling objective.
    *   Fine-tuned the pre-trained model on specific downstream tasks with minimal task-specific modifications.
    *   Achieved impressive results on various NLP benchmarks.
*   **Impact:** GPT demonstrated the power of pre-training and transfer learning in NLP. It showed that a single pre-trained model could be effectively adapted to a wide range of tasks, significantly reducing the need for task-specific architectures and large labeled datasets.

**10. BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding (2018) - Devlin et al.**

*   **Significance:** Introduced the Bidirectional Encoder Representations from Transformers (BERT) model.
*   **Contribution:**
    *   Pre-trained a deep bidirectional Transformer model using two novel pre-training objectives: Masked Language Model (MLM) and Next Sentence Prediction (NSP).
    *   Achieved state-of-the-art results on a wide range of NLP tasks, surpassing previous methods by a significant margin.
*   **Impact:** BERT revolutionized the field of NLP by demonstrating the effectiveness of bidirectional pre-training. It became the dominant approach for many NLP tasks and set a new standard for performance.

##### Conclusion

The journey from early rule-based systems to today's powerful pre-trained language models like GPT and BERT has been marked by a series of significant breakthroughs. The introduction of neural networks, word embeddings, sequence-to-sequence models, and the attention mechanism were crucial steps that paved the way for the Transformer architecture and the era of pre-trained language models. These advancements have dramatically improved the performance of NLP systems and have opened up new possibilities for understanding and generating human language. The field continues to evolve rapidly, with ongoing research exploring new architectures, pre-training objectives, and applications of these powerful models.

</details>

[3]
Cimputational complexity:

Self-Attention: O(length² · dim)  
NN (LSTM): O(length · dim²)  
Convolution: O(length · dim² · kernel_width)    

<details><summary>[4]</summary>
The magic of information passing between tokens in self-attention lies in the attention mechanism itself. It allows each token to "attend" to every other token in the sequence and incorporate information from them into its own representation. Here's a step-by-step breakdown:

1. Queries, Keys, and Values:

   Each token's embedding is projected into three vectors: a query vector, a key vector, and a value vector. These projections are learned during training and are the same for all tokens, as established earlier.
   Think of these as follows:
       Query: Represents what information the token is looking for from other tokens.
       Key: Represents what information the token offers to other tokens.
       Value: Represents the actual information content of the token that will be passed on if deemed relevant.

2. Calculating Attention Scores:

   The dot product is computed between the query vector of one token and the key vector of every other token in the sequence (including itself).
   This dot product results in a score for each pair of tokens, representing the "attention" that the query token should pay to the key token.
   The scores are then scaled down by dividing by the square root of the key dimension (`d_k`) to stabilize training.
   A softmax function is applied to these scaled scores, normalizing them into a probability distribution. These normalized scores are the attention weights.

3. Weighted Sum of Values:

   The attention weights determine how much each token's value vector contributes to the new representation of the query token.
   A weighted sum is calculated, where each token's value vector is multiplied by its corresponding attention weight, and the results are summed up.
   This weighted sum is the new, context-aware representation of the query token.

4. Information Flow:

   Through this process, each token effectively gathers information from all other tokens, weighted by their relevance (as determined by the attention scores).
   Tokens that are deemed highly relevant (high attention weight) will have a greater influence on the new representation of the query token.
   This allows information to flow between tokens, even those that are far apart in the sequence.

Intuition:

Imagine a sentence: "The cat sat on the mat because it was comfortable."

   When processing the word "it," the self-attention mechanism will likely produce high attention scores between "it" and "cat" and "it" and "mat" because these words are semantically related.
   The value vectors of "cat" and "mat" will then be weighted heavily and contribute significantly to the updated representation of "it."
   This allows the model to understand that "it" refers to the cat and the mat, incorporating information from those words into the representation of "it."

Multi-Head Attention:

The "Attention is All You Need" paper takes this a step further with multi-head attention.

   Instead of a single set of `Q`, `K`, `V` projections, multiple sets are used, each representing an "attention head."
   Each head learns to attend to different aspects of the relationships between tokens.
   The outputs of all heads are concatenated and then projected again to produce the final output of the multi-head attention layer.

In essence, self-attention allows each token to dynamically gather information from all other tokens in the sequence, weighted by their relevance. This mechanism enables the Transformer to capture long-range dependencies and build context-aware representations of each token, which is crucial for understanding the meaning of the sequence as a whole.



</details>

</details>


