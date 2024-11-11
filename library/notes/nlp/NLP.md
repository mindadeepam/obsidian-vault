# Papers

1. **Long Short-Term Memory (LSTMs):** "Long Short-Term Memory" by Hochreiter and Schmidhuber (1997) - This paper introduced LSTMs, which addressed the vanishing gradient problem by introducing memory cells and gates. [link](https://blog.xpgreat.com/file/lstm.pdf)
2. **Gated Recurrent Units (GRUs):**
    - "Learning Phrase Representations using RNN Encoder-Decoder for Statistical Machine Translation" by Cho et al. (2014) - This paper introduced GRUs, a simpler alternative to LSTMs that also addressed the vanishing gradient problem.
3. **Attention Mechanisms:**
    - "Neural Machine Translation by Jointly Learning to Align and Translate" by Bahdanau et al. (2014) - This paper introduced the concept of attention mechanisms, which allow the model to focus on specific parts of the input when generating output.
4. **Sequence-to-Sequence Models:**
    - "Sequence to Sequence Learning with Neural Networks" by Sutskever et al. (2014) - This paper introduced sequence-to-sequence models, which use an encoder-decoder architecture to generate output sequences.
5. **Attention-based Sequence-to-Sequence Models:**
    - "Effective Approaches to Attention-based Neural Machine Translation" by Luong et al. (2015) - This paper explored various attention mechanisms for sequence-to-sequence models, including global and local attention.
6. **Transformer:**
    - "Attention Is All You Need" by Vaswani et al. (2017) - This paper introduced the Transformer model, which relies entirely on self-attention mechanisms to process input sequences in parallel.



## instruct gpt
In order to speed up comparison collection, we present labelers with anywhere between K = 4 and K = 9 responses to rank. This produces K 2  comparisons for each prompt shown to a labeler. Since comparisons are very correlated within each labeling task, we found that if we simply shuffle the comparisons into one dataset, a single pass over the dataset caused the reward model to overfit.5 Instead, we train on all K 2  comparisons from each prompt as a single batch element. This is much more computationally efficient because it only requires a single forward pass of the RM for each completion (rather than K 2  forward passes for K completions) and, because it no longer overfits, it achieves much improved validation accuracy and log loss. Specifically, the loss function for the reward model is: loss (θ) = − 1 K 2 E(x,yw,yl)∼D [log (σ (rθ (x, yw) − rθ (x, yl)))] (1) where rθ(x, y) is the scalar output of the reward model for prompt x and completion y with parameters θ, yw is the preferred completion out of the pair of yw and yl , and D is the dataset of human comparisons. 5That is, if each of the possible K 2  comparisons is treated as a separate data point, then each completion will potentially be used for K − 1 separate gradient updates. The model tends to overfit after a single epoch, so repeating data within an epoch also causes it to overfit.

interesting idea to reduce #overfitting, use similar data in a single batch..


%% prefix tuning## 
[paper](https://arxiv.org/pdf/2101.00190)

- only 0.1% params are trained and at the time was better than adapters etc while using lower resources.
- idea: present finetuning fixed prompts as continuous tokens which is appended to input. and only the embeddings of these get updated during finetuning.
#### [chatgpt explains](https://chatgpt.com/share/cd099295-3b8b-4906-b8b2-cb4be95905d4)
Yes, exactly! In prefix tuning, only the embeddings of the prefix vectors are updated during the fine-tuning process.


Breakdown:

	•	Prefix Vectors: These are learnable parameters that serve as the “prefix.” They are typically initialized as random embeddings.
	•	Embedding Update: During fine-tuning, the training process optimizes these embeddings to minimize the loss on the specific task. The optimization process is similar to how embeddings are typically trained in neural networks, using gradient descent or similar techniques.
	•	Model’s Pre-trained Weights: The rest of the model, including its layers and weights, remains frozen and unchanged. The model continues to use these frozen weights to process the combined sequence of the prefix and input text.

How It Works:

	1.	Initialization: The prefix is represented by a set of randomly initialized embeddings. Let’s say you have a prefix of length 10, and the model’s hidden size is 1024. You would have 10 vectors, each with 1024 dimensions.
	2.	Fine-tuning: During training on a specific task, only these 10 vectors (the prefix embeddings) are updated based on the gradients computed from the task-specific loss function (e.g., cross-entropy loss for classification).
	3.	Inference: Once the prefix embeddings are trained, they are fixed, and during inference, the trained prefix is prepended to the input text. The model processes the combined sequence using its pre-trained weights and the learned prefix embeddings to generate the output.

Summary:

Only the prefix embeddings are optimized and updated during fine-tuning, while the rest of the model remains unchanged. This allows for efficient task adaptation with minimal changes to the large pre-trained model. %%