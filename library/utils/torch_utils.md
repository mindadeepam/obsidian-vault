# torch utils

### 1. view vs reshape vs clone

In PyTorch, the `view` method creates a new view of the original tensor without copying the data in memory. Reshaping with `reshape` may or may not create a view, depending on whether the operation is contiguous or not. On the other hand, `clone` always creates a deep copy of the original tensor, copying the data.

### 2. Save and Load model weights

- **Save:**
    
    `torch.save(model.state_dict(), PATH)`
    
- **Load:**
    
    `model = TheModelClass(*args, **kwargs)`
    
    `model.load_state_dict(torch.load(PATH, map_location=<optional>))`
    
    `model.eval()`
    
- note:
    
    *tldr - `.to()` > `map_location`* 
    
    **The `map_location` changes the device of the Tensors in the state dict that is returned. But when you `load_state_dict()`, then these values are loaded (and only values) into the model. But that does not change the model’s device! you will need to move the model itself with `.to()` if you want to have it on a different device
     

### 3. cross entropy in torch
#cross-entropy #loss
***class weights*** -> When training a classification problem with `C` classes, it is beneficial to use the optional argument `weight` if provided. This argument should be a 1D Tensor assigning weight to each of the classes. This becomes particularly useful when dealing with an unbalanced training set.

***shape of input/x*** -> The `input` is expected to contain the unnormalized logits for each class, which do not need to be positive or sum to 1 in general. For unbatched input, the size of `input` should be `(C)`. For minibatched input, it should be `(minibatch, C)`, or in the case of higher-dimensional inputs, such as per-pixel cross entropy loss for 2D images, `(minibatch, C, d1, d2, ..., dK)` with `K ≥ 1`.


### 4. summary of models / some useful functions for debugging

#summary #model

torchinfo is the best library
```
pip install torchinfo
```

![[Screenshot 2024-08-03 at 7.41.49 PM.png|300x200]]
> - Total mult-adds is the total flops in Giga ie billion.
> - lower half shows memory consuption of GPU and its components. 



```python
# some functions
def get_model_params(model):
    return sum(p.numel() for p in model.parameters())/1e6

def get_trainable_params(model):
    return sum(p.numel() for p in model.parameters() if p.requires_grad)/1e6

def model_summary(model):
    print(f"Total params: {get_model_params(model)} million")
    print(f"Trainable params: {get_trainable_params(model)} million")

def print_memory_stats():
    print(f"max memory allocated: {torch.cuda.max_memory_allocated(device)/1e9:.2f}GB")
    print(f"max memory reserved: {torch.cuda.max_memory_reserved(device)/1e9:.2f}GB")

```

```python
gpu_stats = torch.cuda.get_device_properties(0)
start_gpu_memory = round(torch.cuda.max_memory_reserved() / 1024 / 1024 / 1024, 3)
max_memory = round(gpu_stats.total_memory / 1024 / 1024 / 1024, 3)
print(f"GPU = {gpu_stats.name}. Max memory = {max_memory} GB.")
print(f"{start_gpu_memory} GB of memory reserved.")
```


## Other Notes

- sample logits using torch.multinomial
	`next_token = torch.multinomial(probas, 1)     # probabilities, num of items to sample.``

- dim = -1, means doing it across the last dimension, (usually across columns or along the row)
- 


