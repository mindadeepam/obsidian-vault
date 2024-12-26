
> [!Note]
> 1. always set random seed in experiments, especially comaprative ones #tip
> 2. one very common way to , especially when dealing with large *data* is to have it consume lower memory.



## bullshit -> ~~Vectorize your functions:


1.5-3 times speed up when using vectorized functions directly on `df['column']` as apposed to `df['column].apply(func)`~~

```
# result.head()
"""
shuffle off makes vectorized 2500->13.5 secs, slow 2500->18 secs


vectorised: 
    slope=0.0041 
    68mins for 10lakhs lines
    
    10000 - 46
    7500 - 34
    5000 - 20
    2500 - 17
    1000 - 9
    
    50000 - expected-205 seconds, actual-194 seconds
    
slow_preprocess_data: 
    slope - 0.012
    200 mins for 10lakh lines
    
    5000-65
    2500-40
    1000-17
    
    5000 with preprocess outside and feault arguments: 75 seconds
    5000 with preprocess outside and no arguments(global vars): 76 seconds
    
    5000: after restaritng smhpow 45s, 50, 
    2500: 25s
afterrandom seed wasnt set 36,35
    
"""
calc_time_vectorized = lambda x: (0.0041111*x)/60
calc_time_slow = lambda x: (0.012*x)/60
```


## np.vectorize is a SCAM!! wtf

- https://archive.li/pOgyv / https://towardsdatascience.com/dont-assume-numpy-vectorize-is-faster-dd7e455dba2
- official docs! ![[Screenshot 2024-07-04 at 12.36.44 AM.png]]

## use efficient datatypes 

- (pyarraw backend helps a lot in strings)
- pd.read_<any_type>: https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_json.html 
	![[Screenshot 2024-07-04 at 2.18.11 AM.png]]


## To truly free memory, use memory intensive jobs in sub_process/worker process.
#memory #memorymanagement #RAMoverflow

[reference]

There is one basic drawback that you should be aware of: The CPython interpreter actually [can actually barely free memory and return it to the OS](https://realpython.com/python-memory-management/). For most workloads, you can assume that memory is not freed during the lifetime of the interpreter's process. However, the interpreter can re-use the memory internally. So looking at the memory consumption of the CPython process from the operating system's perspective really does not help at all. **A rather common work-around is to run memory intensive jobs in a sub-process / worker process (via [multiprocessing](https://docs.python.org/3/library/multiprocessing.html) for instance) and "only" return the result to the main process. Once the worker dies, the memory is actually freed.**

Second, using `sys.getsizeof` on `ndarray`s can be impressively misleading. Use the `ndarray.nbytes` property instead and be aware that this may also be misleading when dealing with [views](https://scipy-cookbook.readthedocs.io/items/ViewsVsCopies.html).

Besides, I am not entirely sure why you "pickle" numpy arrays. There are better tools for this job. Just to name two: [h5py](https://www.h5py.org/) (a classic, based on [HDF5](https://en.wikipedia.org/wiki/Hierarchical_Data_Format)) and [zarr](https://zarr.readthedocs.io/en/stable/). Both libraries allow you to work with `ndarray`-like objects directly on disk (and compression) - essentially eliminating the pickling step. Besides, zarr also allows you to [create _compressed_ `ndarray`-compatible data structures in memory](https://zarr.readthedocs.io/en/stable/tutorial.html#compressors). Must `ufunc`s from numpy, scipy & friends will happily accept them as input parameters.


## for strings use regex cuz that becomes parelllelizable
