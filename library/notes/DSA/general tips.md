#cp #DSA 

- Most concepts used are basic and dont need you to go out of your way. 
	- for example you are iterating over 2 arrays and are getting a TLE. current T=N^2. can we use hashmaps to improve read speed?
	- another example, is the [tree removal queestion](https://www.codechef.com/problems/TREEREMOVAL?tab=statement). whenver a question comes see which datastructures are obviously used and what properties/algorithms relted to thst structure and help you 

- A #tree is a specific type of graph that satisfies the following constraints:
	1. Acyclic: A tree does not contain any cycles. This means there is no path in the tree that starts and ends at the same vertex and involves at least one edge.
	2. Connected: A tree is connected, meaning there is a path between any two vertices in the graph.
	3. N-1 Edges: A tree with  N  vertices has exactly  N-1  edges. This is a direct consequence of the acyclic and connected properties.
	4. Unique Path: There is exactly one path between any two vertices in a tree. This ensures the absence of cycles and redundancy in connections.

- When there are too many comabinations aggregating from an answer in an allowed total set, think about complementing!!

- nearest smallest/largets in a certain direction for an array screams **stacks** aka lists in python.

- https://www.codechef.com/problems/NUMHUNT?tab=statement

- find if rsum$==$lsum for each i in arr:
  ```python
  class Solution:
    def canSplit(self, arr):
        #code here
        n = len(arr)
    
        sum_r = sum(arr)
        sum_l = 0
    
        for i in range(n):
            sum_r -= arr[i]
            sum_l += arr[i]
	        
	        ## this code takes ~O(n) time itself, because 
	        ## copying a slice of arr and summing takes O(n)
            # if sum(arr[:i])==sum(arr[i:]):
                # return True
            if sum_r==sum_l:
                return True
        return False
	```

- xor can show similarity of numbers cuz `a^a=0`. it can show odd even for 2nd num if we know 1st's odd-eve bcuz `odd^even=odd else even`.
### trees (binary)

**each node has 2 children**.
- inorder-traversal, pre and psotorder tranversal


**bst: each node is greter than all nodes on its left child and smaller than its parent and right node.**
  - delete: inorder predecessor


## concepts:
- https://chatgpt.com/share/92cb0390-f7b9-4107-b530-65250f8a653d: 
  A good read, discusses 
	  - graph basics 
	  - python data structures 
	  - time complexities of some algorithms.


