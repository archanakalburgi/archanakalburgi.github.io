---
layout: post
title: "Walls and Gates — From OGM to OM"
date: 2021-09-13
description: A follow-up walkthrough simplifying the original walls-and-gates solution.
tags: [algorithms, leetcode, graphs, bfs]
---

<p align="center">
  <img width="500" height="300" src="/images/ogm_to_om.jpg">
</p>

[image source](https://www.azenhut.com/2018/02/28/dogs-in-meditation/) 

In my previous [blog post](https://archanakalburgi.github.io/2021/09/11/walls-gates.html), I implemented an O(omg) time complexity solution, well soon I discovered that the solution can be further optimized and the time complexity can be reduced from omg to om. OMG right!?

What made the previous implementation's time complexity omg?

```sh
def wallsAndGates(rooms):
    for row in range(len(rooms)):
        for column in range(len(rooms[0])):
            if rooms[row][column] == 0:
                bfs(row, column, rooms)
```

BFS is called on every gate in the matrix
```sh
if rooms[row][column] == 0:
    bfs(row, column, rooms)
```
Even though the time complexity of BFS is O(om), calling it on every gate in the grid make the overall implementation an O(omg) solution\
Instead of starting BFS on every gate, starting BFS from the first gate and updating the values of cells with the shortest distance will reduce the time complexity\
This is how modified the previous implementation 

```sh
def valid_neighbors(r,c,row,col):
    valid = []
    neighbors = [(r,c-1),(r,c+1),(r-1,c),(r+1,c)]
    for (nr,nc) in neighbors:
        if 0 >= nr > row and 0 >= nc > col:
            valid.append((nr,nc))
    return valid

def bfs(row,col,rooms):
    queue = [(row, col, 0)]
    visited = set((row,col))
    while queue:
        r,c,dist = queue.pop(0)
        for (nr, nc) in valid_neighbors(r,c,rooms): 
            if (nr,nc) not in visited:
                if rooms[nr][nc] > dist:
                    rooms[nr][nc] = dist+1
                    queue.append((nr,nc,dist+1))
                    visited.add((nr,nc))

def wallsAndGates(rooms):
    ro = len(rooms)
    column = len(rooms[0])
 
    for row in range(ro):
        for col in range(column):
            if rooms[row][col] == 0:
                bfs(row,col,rooms)
```

- with this implementation, we are checking the cell once and update the value only if the current distance is smaller than the value in the cell.
- what I found very delightful is that we do not have to worry about the 0 and -1 value in the cell, which represents a gate and a wall because 0 or -1 cannot be greater than the distance 
- the following condition in bfs() function will take care of the above

```sh
if rooms[nr][nc] > dist
```

Hope you find this post interesting!! 