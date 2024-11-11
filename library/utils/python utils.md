
### pip install a github repository

`git+https://github.com/Farmart-Tech/grain-instance-segmentation.git@dev#egg=grain_instance_segmentation`

`pip install git+https://github.com/FarMart-Tech/grain-instance-segmentation.git`


### traceback

`traceback.print_exc()` vs `traceback.format_exc()`: 1st one prints, 2nd only stores string


### multithreading
check this out [[Multiprocessing vs Multithreading vs Asyncio]]


### heapq module

The `heapq` module in Python provides a set of functions for working with heaps, specifically min-heaps. A min-heap is a binary heap where the smallest element is always at the root. The `heapq` module operates on standard Python lists and transforms them into heaps.

Here's a summary of the primary functions provided by the `heapq` module:

1. **`heapq.heapify(iterable)`**:
   - **Description**: Converts a list into a heap in-place.
   - **Complexity**: O(n), where n is the number of elements in the list.

   ```python
   import heapq
   data = [5, 7, 9, 1, 3]
   heapq.heapify(data)
   ```

2. **`heapq.heappush(heap, item)`**:
   - **Description**: Adds an element to the heap, maintaining the heap property.
   - **Complexity**: O(log n), where n is the number of elements in the heap.

   ```python
   import heapq
   heap = [1, 3, 5]
   heapq.heappush(heap, 2)
   ```

3. **`heapq.heappop(heap)`**:
   - **Description**: Removes and returns the smallest element from the heap, maintaining the heap property.
   - **Complexity**: O(log n), where n is the number of elements in the heap.

   ```python
   import heapq
   heap = [1, 2, 3]
   smallest = heapq.heappop(heap)
   ```

4. **`heapq.heappushpop(heap, item)`**:
   - **Description**: Pushes a new element on the heap and then pops and returns the smallest element from the heap. This is more efficient than doing a `heappush` followed by a `heappop`.
   - **Complexity**: O(log n), where n is the number of elements in the heap.

   ```python
   import heapq
   heap = [1, 3, 5]
   result = heapq.heappushpop(heap, 2)
   ```

5. **`heapq.heapreplace(heap, item)`**:
   - **Description**: Pops and returns the smallest element from the heap and then pushes a new element onto the heap. This is more efficient than doing a `heappop` followed by a `heappush`.
   - **Complexity**: O(log n), where n is the number of elements in the heap.

   ```python
   import heapq
   heap = [1, 3, 5]
   result = heapq.heapreplace(heap, 2)
   ```

6. **`heapq.nlargest(n, iterable, key=None)`**:
   - **Description**: Returns the `n` largest elements from the dataset, in descending order. The `key` argument is optional and specifies a function of one argument used to extract a comparison key from each element.
   - **Complexity**: O(n log k), where `n` is the number of elements in the iterable and `k` is the number of largest elements to retrieve.

   ```python
   import heapq
   data = [1, 5, 3, 8, 7]
   largest = heapq.nlargest(3, data)
   ```

7. **`heapq.nsmallest(n, iterable, key=None)`**:
   - **Description**: Returns the `n` smallest elements from the dataset, in ascending order. The `key` argument is optional and specifies a function of one argument used to extract a comparison key from each element.
   - **Complexity**: O(n log k), where `n` is the number of elements in the iterable and `k` is the number of smallest elements to retrieve.

   ```python
   import heapq
   data = [1, 5, 3, 8, 7]
   smallest = heapq.nsmallest(3, data)
   ```

These functions provide an efficient way to maintain and manipulate heaps for various use cases in Python.



### @property tags

https://chatgpt.com/share/e76ddb9c-f095-4eea-9838-0f18fdda955a



### 