---
layout: post
title: "Three Number Sum"
date: 2021-08-17
description: Another array problem demonstrating tradeoffs between brute force and pointer-based solutions.
tags: [algorithms, leetcode, arrays]
---

Another example that shows the time and space complexity trade off

<p align="center">
  <img width="400" height="300" src="/images/three_num_sum.jpg">
</p>

[image source](https://www.datamation.com/?s=geek+%26+poke)

# Problem statement 

Write a function that takes in a non-empty array of distinct integers and an integer representing a target sum. The function should find all the triplets in the array that sum up to the target sum and return a two-dimensional array  of all these triplets. The numbers in each triplet should be ordered in ascending order, and the triplets themselves should be ordered in ascending order with respect to the numbers they hold.

If no three numbers sum up to the target sum, the  function should return an empty array

# Understand

Input: array = [12,3,1,2,-6,5,-8,6], target = 0\
Output: [[-8,2,6],[-8,3,5],[-8,1,5]] 

Input: array = [4,1,3], target = 8\
Output: [[1,3,4]]

Input: [1,4,2,3,1], target = 6\
Output: [[1,1,4],[1,2,3]] 

Input: array = [4,1,3], target = 10\
Output: []

Input: [1,2], target = 5\
Output: []

# Match and Plan

**List**
- the first solution that comes to mind is iterating the list thrice to get the desired three numbers 
- we could do so in the following way

    - **Plan**
        ```sh
        // result = []
        // for i -> 0-len(array):
            // for i -> i-len(array):
                //for k -> j-len(array):
                    if array[i]+array[j]+array[k] == targetSum
                        result -> sort[array[i], array[j],array[k]] 
        // return sort(result)
        ```
        - complexity
            - time: O(n<sup>3</sup>) + O(3 log(3)) + O(r log(r)) -> O(n<sup>3</sup>), where n=len(array), r=len(result)
            - space: O(r), where r=len(result)

**Linked list**
- linked will complicate the solution by increasing the time and space complexity 

**Hashmap**

- while we are iterating the array we will check if targetSum - (array[i]+array[j]) is in the hashmap
- targetSum - (array[i]+array[j]) is essentially the third number of the triplet 
- if present in hashmap, remember the sorted order of array[i], array[j] and targetSum-(array[i]+array[j]
- if not present then map array[j] to array[i], targetSum - (array[i] + array[j])
- we are essentially trying to sum two number of the array and looking if the third number through process 

    - **Plan**
        ```sh
        // result = []
        // for i -> 0-len(array)
            // hashmap = {}
            // for j -> i-len(array)
                if targetSum - (array[i] + array[j]) in hashmap
                    result -> sort([array[i],array[j],targetSum-(array[i]+array[j]))
                else:
                    hashmap[array[j]] -> array[i],targetSum - (array[i] + array[j])
        // return sort(result) 
        ```	
        - complexity
            - time: O(n<sup>2</sup>) + O(3 log(3)) + O(r log(r)) -> O(n<sup>2</sup>), where n=len(array), r=len(result)
            - space: O(r), where r=len(result)

- making use of hashmap we have managed to reduce the time complexity from O(n<sup>3</sup>) to O(n<sup>2</sup>) 
- the above is implemented using hashmap

**Sets**

- there is not need to store hashmap[array[j]]:array[i],targetSum - (array[i] + array[j]), since we are only interested in knowing the third number of the triplet
- to avoid storing the mapping we can use set which will give us a constant time look up while not consuming extra space 
by just storing the numbers we have iterated through  

_Exploring the techniques involved in traversing the array helped me arrive at the following plan_

**Technique**

**Three pointer approach:**

- idea behind using three pointers:
    - if we have a sorted array, we can place a _start_ pointer on the number to the right of the current number and _end_ pointer at the last number in the array
    - check if current number + array[start] + array[end] == targetSum
    - if start is incremented by 1 will increment the sum(current number + array[start] + array[end])
    - where as decrementing the end by 1 will reduce the sum(current number + array[start] + array[end])
    - with this information we can obtain the list of triplets ([current number, array[start], array[end]]) which sum up to the targetSum

    - **Plan**
        ```sh
        // sort given array
        // result = []
        // for: i -> 0-len(array)-2 
                start = i+1
                end = len(array)-1
                while start<end:
                    current_sum = array[i]+array[start]+array[end]
                    if current_sum == targetSum
                        result.append([array[i],array[start],array[end]])
                        move pointers close to each other by increasing start by 1 and decreasing end by 1
                    if current_sum < targetSum
                        move start pointer by increasing start by 1
                    if current_sum > targetSum:
                        move end pointer by decreasing end by 1
        // return result
        ```
        - complexity
            - time: O(n<sup>2</sup>), where n=len(array)
            - space: O(3n) -> O(n), where n=len(array)

# Implementation and Evaluate 

- let's implement all the above discussed solutions 

```sh
def threeNumberSum(array, targetSum):
	result = []
    for i in range(len(array)):
		for j in range(i+1,len(array)):
			for k in range(j+1,len(array)):
				if array[i] + array[j] + array[k] == targetSum:
					result.append(sorted([array[i],array[j],array[k]]))
    return sorted(result)
```
- complexity
    - time: O(n<sup>3</sup>) + O(3 log(3)) + O(r log(r)) -> O(n<sup>3</sup>), where n=len(array), r=len(result)
    - space: O(3n) -> O(n), where n=len(array)

```sh
def threeNumberSum(array, targetSum):
	result = []
	for i in range(len(array)):
		temp = dict()
		for j in range(i+1,len(array)):
			if targetSum - (array[i] + array[j]) in temp:
				result.append(sorted((array[i],array[j],targetSum - (array[i] + array[j]))))
			else:
				temp[array[j]] = array[i],targetSum - (array[i] + array[j])
	return sorted(result)
```
- complexity
    - time: O(n<sup>2</sup>) + O(3 log(3)) + O(r log(r)) -> O(n<sup>2</sup>), where n=len(array), r=len(result)
    - space: O(n+2) + O(n) -> O(n), where n=len(array)

```sh
def threeNumberSum(array, targetSum):
	result = []
	for i in range(len(array)):
		temp = set() # s: O(n)
		for j in range(i+1,len(array)):
			if targetSum-(array[i]+array[j]) in temp:
				result.append(sorted([array[i],array[j],targetSum-(array[i]+array[j])]))
			else:
				temp.add(array[j]) 
    return sorted(result)
```
- complexity 
    - time: O(n<sup>2</sup>) + O(3 log(3)) + O(r log(r)) -> O(n<sup>2</sup>), where n=len(array), r=len(result)
    - space: O(n) + O(3n) -> O(n), where n=len(array)

```sh
def threeNumberSum(array, targetSum):
	# three pointer approach
    array.sort()
	result = []
	for i in range(len(array)-2):
		start = i+1
		end = len(array)-1
		while start<end:
			current_sum = array[i]+array[start]+array[end]
			if current_sum == targetSum:
				result.append([array[i],array[start],array[end]])
				start += 1
				end -= 1
			elif current_sum < targetSum:
				start += 1
			elif current_sum > targetSum:
				end -= 1
	return result
```
- complexity
    - time: O(n<sup>2</sup>), where n=len(array)
    - space: O(3n) -> O(n), where n=len(array)

