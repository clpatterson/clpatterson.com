---
title: "Sorting an Array by Absolute Value"
date: 2020-09-17T13:51:25-04:00
draft: False
description: "Implementing selection sort or using a built-in function."
---

I tried out [Pramp](https://www.pramp.com/) this morning and encountered an interesting question. The question can be solved with one line of code or by implementing a sorting algorithm. I implemented a selection sorting algorithm with the help of my partner and was delighted to learn about the one-liner approach afterwards. So in this post I'm going to review the problem and explain both approaches. Below is the problem prompt:

### The Question
**Absolute Value Sort:**

Given an array of integers `arr`, write a function `absSort(arr)` that
sorts the array according to the absolute values of the numbers in `arr`.
If two numbers have the same absolute value, sort them according to sign,
where the negative numbers come before the positive numbers.

```python
# Example
input: arr = [2, -7, -2, -2, 0]
output: [0, -2, -2, 2, -7]

# Constraints:
[time limit] 5000ms
[input] array.integer arr
0 <= arr.length <= 10
[output] array.integer
```

### Implementing Selection Sort

Selection sort is not the most efficient sorting algorithm (O(n^2) vs. O(n log n) for merge sort), but it is relatively straight forward and easy to implement. In selection sort you break the array into two subarrays, sorted and unsorted. The sorted subarray is created by the first pass through the array. You begin the pass at the 0th index, storing that number and its index as the current_minimum, then compare it to every other number in the array, overwriting the current_minimum if you find a smaller number. When all numbers have been compared, you take the smallest number (stored in current_minimum) and swap that number and the number at the start of the pass (in this case the 0th index). Then you repeat the process. Over time the sorted subarray grows and overtakes the unsorted subarray, resulting in a final sorted array.

As you can see below, this algorithm is implemented with a nested for loop. The first for loop tracks the start of our current pass and the second for loop carries out the search for the next smallest number (using absolute value and a number's sign).

```python
def absSort(arr):
  
  for i in range(len(arr)): # Starting index of current pass
    cur_min = (arr[i],i) # reset min at start of each pass
    
    for idx in range(i, len(arr)): # For each pass, compare each remaining elements
      if abs(arr[idx]) < abs(cur_min[0]):
          cur_min = (arr[idx], idx)
      
      elif abs(arr[idx]) == abs(cur_min[0]):
        if arr[idx] < 0: # check sign, negative goes first
          cur_min = (arr[idx], idx)
      
      else:
        continue
    
    # Perform swap after each pass
    prev = arr[i]
    arr[i] = cur_min[0]
    arr[cur_min[1]] = prev
  
  return arr
```

### Using `sort()` or `sorted()` and Lambda Functions
Both `sort()` and `sorted()` take an optional keyword argument, "key", that takes
a function that determines sorting criteria. However, the two have some important
differences. `sort()` is a list method that changes a list in-place. `sorted()` is a
built-in function that takes an iterable as an argument and sorts its elements 
out-of-place. This means with `sort()` the original order of the elements in the
list is non-recoverable, while `sorted()` does not alter the original iterable. Also
keep in mind `sorted()` is less space efficient because it creates a copy of the input
iterable. 

The Absolute Value Sort problem does restrict us from changing the input array,
so I'll use sort(). The challenge is then to figure out how to write the key 
function that will allow us to sort by absolute value, but place negative 
numbers first if two numbers have the same absolute value. We can accomplish this
by writing a lambda function that will produce a tuple to use as the key. The first value
of the tuple is the absolute value of the element and the second value is the element.
If the absolute value (the first element of the key) is the same, then the sorting algorithm
will default to comparing the second element of the key, the original value of the element.
This will cause the negative number to always be placed first. 

Both sorting functions use the [Timsort](https://www.wikiwand.com/en/Timsort) algorithm
developed by early Python Core Developer Tim Peters, which has a time complexity of 
O(n log n). This is faster than our O(n^2) solution using Selection Sort. 

```python
def absSort(arr):
  arr.sort(key=lambda x: (abs(x),x))
  return arr 
# not technically a one-liner with sort(), but space complexity is O(1)
```

Remember that `arr.sort(...)` will return None, not the sorted list.
That is why we return arr, not the statement above, which makes this function a "two-liner".

```python
def absSort(arr):
   return sorted(arr, key=lambda x: (abs(x),x)) 
# space complexity is O(n)
```




### Additional Resources:
* [RealPython: How to Use Sorted() and Sort() in Python](https://realpython.com/python-sort/)
* [RealPython: How to Use Python Lambda Functions](https://realpython.com/python-lambda/)

