---
title: "Island Hopping: BFS & DFS for Counting Islands"
date: 2020-09-22T20:30:32-04:00
draft: true
description: Learning Breadth-first Search & Depth-first Search
---

I really like the [Number of Islands](https://leetcode.com/problems/number-of-islands/) problem on leetcode. I imagine my algorithm traversing a clear blue lagoon and racing over palm strewn islands, which is a more enjoyable visual than reversing words in a string. The problem was fairly challenging for me, but I had fun figuring it out. And it made me more excited about learning breadth-first search (BFS) and depth-first search (DFS). Below I'll discuss my approach to solving this problem and explain BFS and DFS along the way.

### The Problem
(introduce problem here)

### First attempt at BFS
Notes:
* Tried to break up functionality to make it easier to read
* This solution runs for most tests, but gets time exceeded error for 20 x 20 matrix
* Why is my solution so slow? How can I improve it?
    * I eliminated the class attribute for grid (no need to copy that)...still times out
    * 
```python
import collections

Coordinate = collections.namedtuple('Coordinates', ['x','y'])

class Solution:
    
    def __init__(self):
        self.grid = None
    
    def nbrs(self, cur):
        # north south east west nbrs
        nbr_coords = [Coordinate(cur.x - 1, cur.y), 
                      Coordinate(cur.x + 1, cur.y), 
                      Coordinate(cur.x, cur.y + 1),
                      Coordinate(cur.x, cur.y - 1)]
        # filter out invalid nbrs
        nbrs = [nbr for nbr in nbr_coords 
                if (0<= nbr.x < len(self.grid)) and (0<= nbr.y < len(self.grid[0]))]
        return [nbr for nbr in nbrs if self.grid[nbr.x][nbr.y] == "1"]

    def bfs(self, start):
        queue = collections.deque()
        queue.appendleft(start)
        while len(queue) > 0:
            cur = queue.pop()
            nbrs = self.nbrs(cur)
            if len(nbrs) > 0:
                queue.extendleft(nbrs)
            self.grid[cur.x][cur.y] = "0" # update grid to reflect exploration
        return     
    
    def numIslands(self, grid: List[List[str]]) -> int:
        self.grid = grid
        island_count = 0
        for x, row in enumerate(self.grid):
            for y, value in enumerate(row):
                if value == "0":
                    continue
                else:
                    start = Coordinate(x,y)
                    self.bfs(start) # traverses whole island and converts land to water
                    island_count +=1
        return island_count
```

### BFS
```python
import collections

class Solution:
    
    def __init__(self):
        self.nbr_coords = [(-1,0),(1,0),(0,1),(0,-1)]
        self.x_bnd = None
        self.y_bnd = None
    
    def nbrs(self, grid, cur):
        nbrs = []
        for nbr in self.nbr_coords:
            nbr = (cur[0]+nbr[0],cur[1]+nbr[1])
            if 0 <= nbr[0] < self.x_bnd and 0 <= nbr[1] < self.y_bnd:
                if grid[nbr[1]][nbr[0]] == "1":
                    nbrs.append(nbr)
        return nbrs

    def bfs(self, grid, start):
        queue = collections.deque()
        queue.appendleft(start)
        grid[start[1]][start[0]] = "0"
        while queue:
            cur = queue.pop()
            nbrs = self.nbrs(grid, cur)
            for nbr in nbrs:
                queue.appendleft(nbr)
                grid[nbr[1]][nbr[0]] = "0"
        return
    
    def numIslands(self, grid: List[List[str]]) -> int:
        self.y_bnd = len(grid)
        self.x_bnd = len(grid[0])
        island_count = 0
        for y in range(self.y_bnd):
            for x in range(self.x_bnd):
                if grid[y][x] == "0":
                    continue
                else:
                    start = (x,y)
                    self.bfs(grid, start) # traverses whole island and converts land to water
                    island_count +=1
        return island_count
```

## DFS
```python
import collections

class Solution:
    
    def __init__(self):
        self.nbr_coords = [(-1,0),(1,0),(0,1),(0,-1)]
        self.x_bnd = None
        self.y_bnd = None
    
    def nbrs(self, grid, cur):
        nbrs = []
        for nbr in self.nbr_coords:
            nbr = (cur[0]+nbr[0],cur[1]+nbr[1])
            if 0 <= nbr[0] < self.x_bnd and 0 <= nbr[1] < self.y_bnd:
                if grid[nbr[1]][nbr[0]] == "1":
                    nbrs.append(nbr)
        return nbrs

    def dfs(self, grid, node):
        nbrs = self.nbrs(grid, node)
        
        if not nbrs: # base case: node has no nbrs that are land 
            return
        
        else:
            for nbr in nbrs: # recursive case: node has nbrs with land
                grid[nbr[1]][nbr[0]] = "0" # mark as traversed
                self.dfs(grid, nbr)
        
    
    def numIslands(self, grid: List[List[str]]) -> int:
        self.y_bnd = len(grid)
        self.x_bnd = len(grid[0])
        island_count = 0
        for y in range(self.y_bnd):
            for x in range(self.x_bnd):
                if grid[y][x] == "0":
                    continue
                else:
                    start = (x,y)
                    self.dfs(grid, start) # traverses whole island and converts land to water
                    island_count +=1
        return island_count
```