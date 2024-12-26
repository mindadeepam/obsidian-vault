
#binary-search #dsa

*binary search reduces comlpexity from O(N) to O(logN)*

Q. You are given an array with unique elements of stalls[], which denote the position of a **stall**. You are also given an integer **k** which denotes the number of aggressive cows. Your task is to assign **stalls** to **k** cows such that the **minimum distance** between any two of them is the **maximum** possible.

--> 
Typical signs of binary search: min ka max karo, max ka min karo...

we define a range of answers and perform binary search on that. For eg here the min dist possible is 0 and max is max(arr)-min(arr). So we can perform binary search on that range and everytime **check**. If check succeeds, we go higher(s=mid+1) or lower(e=mid-1) depending on whether we are maxing or mining. 

the **check** function returns True if we can satisy the condistions with that *distance* (in this case)
And usually, check involves binning k things with a max or min size per bin or something.. so just **greedily stsrt allocating** and return False if not possible and True otherwise.

```python
def check_possible(arr, k, max_dist):

    prev = arr[0]
    cows = 1

    for i in range(len(arr)):
        if arr[i]-prev>=max_dist:
            cows+=1
            prev = arr[i]

        # if cows are finished, return True
        if cows>=k:
            return True 

    return False


def minimum_cow_distance(arr, k):

    arr = sorted(arr)
    s = 0
    e = max(arr)
	
    result = -1
    while s<=e:
        mid = int((s+e)/2)
        # print(f"mid {mid}")    

        if check_possible(arr, k, mid):
            # print(mid)
            result = mid 
            s = mid+1
        else:
            e = mid-1
        
    return result
    pass 

stalls = [1, 2, 4, 8, 9]
k = 3

print(minimum_cow_distance(stalls, k))
```



## Over continuous values:
when we are searching over a continuous range there are a few changes, eg in [minimize-max-distance-to-gas-station](https://www.geeksforgeeks.org/problems/minimize-max-distance-to-gas-station/1):
  
  ```python
	while e - s > 1e-6:  # 1. Use small epsilon for floating point comparison
		mid = (s + e) / 2
		if check_possible(arr, k, mid):
			e = mid      # 2. no +- 1 because we are not iterating over ints now 
		else:
			s = mid

