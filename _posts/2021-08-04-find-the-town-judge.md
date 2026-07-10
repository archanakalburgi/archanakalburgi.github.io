---
layout: post
title: "Find the Town Judge"
date: 2021-08-05
description: Walking through the UMPIRE method to identify the judge from trust relationships.
tags: [algorithms, leetcode, graphs]
---
Here is an interesting problem that I enjoyed solving using UMPIRE method.

<p align="center">
  <img width="460" height="300" src="/images/image_tj.jpg">
</p>


# Problem statement

In a town, there are _n_ people labeled from 1 to n. There is a rumor that one of these people is secretly the town judge.

If the town judge exists, then:

- The town judge trusts nobody.
- Everybody (except for the town judge) trusts the town judge.

There is exactly one person that satisfies properties 1 and 2.

You are given an array _trust_ where trust[i] = [ai, bi] representing that the person labeled a<sub>i</sub> trusts the person labeled b<sub>i</sub>. 

Return the label of the town judge if the town judge exists and can be identified, or return -1 otherwise.

For Example:
- For n = 2 and trust = [[1,2]] return 2, because 1 trusts 2, 2 trusts no one

# Understand 

**Happy cases**:

Input: n = 3, trust = [[1,2],[3,2]]\
Output: 2

Input: n = 4, trust = [[1,3],[1,4],[2,3],[2,4],[4,3]]\
Output: 3

Input: n = 4, trust = [[1,2],[2,4],[3,4],[1,4]]\
Output: 4

Input: n = 3, trust = [[1,2],[2,3]]\
Output: -1

Input: n = 3, trust = [[1,3],[2,3],[3,1]]\
Output: -1

Input: n = 2, trust = [[1,2],[2,1]]\
Output: -1

Input: n = 5, trust = [[1,3],[1,2],[1,4],[1,5]]\
Output: -1

# Match 

**Lists**:
```sh
// iterate _trust_ to create a list of list, 
    list[i] = list of people i trusts, 
// find a member common to all the lists (town judge)
// if no common member found: 
    return -1 
```
- complexity:
    - time: O(n), where n = len(_trust_)
        - time to traverse _trust_ array 
    - space: O(m<sup>2</sup>), where m = n
        - for example, if n = 5, trust = [[1,2],[1,3],[1,4],[1,5],[2,1],[2,3],[2,4],[2,5],[3,1],[3,2],[3,4],[3,5],[4,1],[4,2],[4,3],[4,5],[5,1],[5,2],[5,3],[5,4]]
        - after iterating through _trust_ we will have a list = [[2,3,4,5],[1,3,4,5],[1,2,4,5],[1,2,3,5],[1,2,3,4]]
        - space = O(m*m-1) ≈ O(m<sup>2</sup>), where m = n

**Linked List**
- linked list will not optimize the m<sup>2</sup> complexity 

**Stack and Queue**
- stacks and Queue data structure help us in maintaining LIFO or FIFO order
- the problem does not require us to look or maintain LIFO or FIFO order
- so it's better to avoid stack or queue 

**Heap**
- Since we are not required to sort, heap will not be useful to solve this problem 

**Hashmap**
- Oh, yes hashmaps!! 
- we could have n keys whose values can be the people a particular person trusts 
- since every person is unique, the keys are unique 
- also, there is no need to maintain the order of people 
- plus hashmaps have constant time look up, which will make our solution faster 
- so hashmap it is!

# Plan

- build a hashmap, where keys are the people ranging from 1-n and the values are can the appended looking up the _trust_ array
- in other words we are building a graph
- the relation between the nodes is given in the _trust_ array 
- the first property, the town judge trusts nobody, makes town judge a sink node, and
- the second property, everybody (except for the town judge) trusts the town judge, essentially tells that all the nodes in the graph are connected to the sink node
- this is clearly a dependency graph problem 
- steps involved:
    ```sh
    // construct a graph
    // if sink node not found: 
            return -1
    // if sink node present:
        if all nodes are not connected to sink node:
            return -1
    // return sink node
    ```
    
# Implement

```sh
    def findJudge(n: int, trust: List[List[int]]) -> int:
        graph = dict()
        in_degree = dict() # to find sink node
        judge = -1
        
        for i in range(1,n+1): # O(n) 
            graph[i] = set()
            in_degree[i] = 0
            
        for edge in trust: # O(t)
            graph[edge[0]].add(edge[1])
            in_degree[edge[0]] += 1
        
        for key in in_degree.keys(): # O(n)
            if in_degree[key]==0: # only one sink node, only one judge per town
                judge = key
   
        for key in graph.keys():# O(n)
            if judge not in graph[key] and key!=judge: # in order to exclude the judge himself
                return -1
            
        return judge
```

# Review 

Input: n = 2, trust = [[1,2]]
- for this problem, 
    - graph = {1: {2}, 2: set()}
    - in_degree = {1: 1, 2: 0} 
    - judge = 2 (in_degree[2]==0)

Output: 2

# Evaluate 

- time: n + t + n + n ~ O(3n + t) ~ O(n + t)
    - if t << n ~ O(n)
    - where n = n(number of people) and t = len(trust) 
- space: O(n) 
    - graph and in_degree
