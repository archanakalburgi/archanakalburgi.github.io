---
layout: post
title: "Two Number Sum"
date: 2021-08-16
description: A classic array problem that surfaces the time-vs-space complexity tradeoff.
tags: [algorithms, leetcode, arrays]
---

Following is one of the many array questions and is a good example that shows the time and space complexity trade off 
<p align="center">
  <img width="350" height="460" src="/images/two_num_sum.jpg">
</p>

[image source](https://www.pinterest.se/pin/466192998921808715/) 

# Problem statement  

Write a function that takes in a non-empty array of distinct integers and an integer representing a target sum. If any two numbers in the input array sum up to the target sum, the function should return them in an array, in any order. If no two numbers sum up to the target sum, the function should return an empty array.

The two numbers must be two distinct integers in the array, you cannot add a single integer to itself in order to obtain the target sum 

Assume that there will be at most one pair of numbers summing up to the target sum.

# Understand

Input: array = [1,4,2,6], targetSum = 3\
Output: [1,2]

Input: array = [2,3,1,4] targetSum = 5\
Output: Not a valid test case

Input: array = [], targetSum = 10\
Output: Not a valid test case

Input: array = [9,3,2,1], targetSum=0\
Output: Not a valid test case

Input: array = [3], targetSum = 4\
Output: Not a valid test case

# Match and plan 

**List**

- given input is a list, so we have to iterate through the list to arrive at the solution 
- let us look at how does one of the solutions looks like and it's complexity 

    - **Plan**

        ```sh
        // for i in 0-len(array): # t: O(n)
            // for j in i+1-len(array): # t: O(n)
                // if array[i] + array[j] == targetSum
                    return [array[i], array[j]]
        ```
        - Complexity:
            - time: O(n<sup>2</sup>), where n = len(array)
            - space: O(1)

    - we could _in_ operator as follows

    - **Plan**

        ```sh
        // result = []
        // for num in array: # t: O(n)
            // if (targetSum - num) in array and (targetSum - num) != num: # t: O(n) in operation 
                result->add[num, (targetSum-num)]
                return result
        return result 
        ```
        - Complexity:
            - time: O(n<sup>2</sup>), where n=len(array)
            - space: O(1) 

**Linked list**

- complicates the solution 

**Hash-set**

- we can solve this problem using set, where you just have to remember if you have seen the difference before while iterating through the list, if yes return the two numbers if not then add the number to the set
- don't look for the the second number just look for the difference, inspired from the image 
    - **Plan**

        ```sh
        // hashset = set() , result = [] # s: O(n), O(1)
        // for 0-len(array) # t: O(n)
            if targetSum-array[i] in hashset: # t: O(1)
                return array[i], targetSum-array[i]
            else
                hashset(array[i]) # t: O(1)
        ```
        - Complexity:
            - time: O(n) + O(1) + O(1) -> O(n), n=len(array)
            - space: O(n) 
	
**Stack or Queue**

- not required to maintain any order 

**Heaps**

- useful if we were to check for the top k elements

**Technique**

- **Two pointers**

- idea behind using two pointers:
    - if we have a sorted array, we can place a _start_ pointer at the first number of the array _end_ pointer at the last number in the array
    - check if array[start] + array[end] == targetSum
    - if start is incremented by 1 will increment the sum(array[start] + array[end])
    - where as decrementing the end by 1 will reduce the sum(array[start] + array[end])
    - with this information we can obtain the list of two numbers ([array[start], array[end]]) which sum up to the targetSum

    - **Plan**
        ```sh
        // result = []
        // sort the array # t: O(n log(n))
        // using points look for the two numbers whose sum = targetSum
            // return the two numbers
        ```

        - Complexity:
            - time: O(n), where n = len(array)
            - space: O(1) 


# Implement 

- let us implement all the above discussed solutions 

```sh
# this an O(n^2) solution
def two_number_sum(array, targetSum):
    for i in 0-len(array): # t: O(n)
        for j in i+1-len(array): # t: O(n)
            if array[i] + array[j] == targetSum
                return [array[i], array[j]]
    return []
```
- time: O(n<sup>2</sup>), where n = len(array)
- space: O(1)

```sh
# O(n^2) solution 
def two_number_sum(array, targetSum):
    result = []
	for num in array:
		if (targetSum - num) in array and (targetSum - num) != num:
			result.extend([num, (targetSum-num)])
			return result
	return result 
```
- time: O(n<sup>2</sup>) 
- space: O(1) 

```sh
# O(n) solution
def two_number_sum(array, targetSum):
    array = sorted(array)
    start = 0
    end = len(array) - 1
    while start < end:
        current_sum = array[start] + array[end]
        if current_sum == targetSum:
            return [array[start], array[end]]
        elif current_sum < targetSum:
            start += 1
        elif current_sum > targetSum:
            right -= 1
    return []
```
- time: O(n), where n = len(array)
- space: O(1)

```sh
# O(n) solution using hashset
def twoNumberSum(array, targetSum):
	temp = set()
	for i in range(len(array)):
		if targetSum-(array[i]) in temp:
			return array[i], targetSum-array[i]
		else:
			temp.add(array[i]) 
    return []
```
- time: O(n)
- space: O(n)

# Evaluate 

- this problem has solution of varied time and space complexity 
- before implementation we have decide on space and time trade off
