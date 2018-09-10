---
layout: post
title: "topological sort"
tagline: ""
description: "topological sort in python"
category: python
tags: [python,topological sort]
last_updated: 2018-09-09
---

616. Course Schedule II
There are a total of n courses you have to take, labeled from 0 to n - 1.
Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: [0,1]

Given the total number of courses and a list of prerequisite pairs, return the ordering of courses you should take to finish all courses.

There may be multiple correct orders, you just need to return one of them. If it is impossible to finish all courses, return an empty array.

Example
Given n = 2, prerequisites = [[1,0]]
Return [0,1]

Given n = 4, prerequisites = [1,0],[2,0],[3,1],[3,2]]
Return [0,1,2,3] or [0,2,1,3]

## kahn`s algorithm
```python
class Solution:
    """
    @param: numCourses: a total of n courses
    @param: prerequisites: a list of prerequisite pairs
    @return: true if can finish all courses or false
    """
    def findOrder(self, numCourses, prerequisites):
        # write your code here
        indegree = [0]*numCourses
        graph = {i: [] for i in range(numCourses)}
        for e,s in prerequisites:
            indegree[e] += 1
            graph[s].append(e)
        # Add courses with no prereqs to queue
        q = [i for i in range(numCourses) if indegree[i]==0]
        ans = []
        while q:
            head = q.pop(0)
            ans.append(head)
            # Remove all outgoing edges
            for node in graph[head]:
                indegree[node] -= 1
                if indegree[node] == 0:
                    q.append(node)
        return ans if len(ans) == numCourses else []
```
using `q = collections.deque()` to replace `q = list()` and `q.popleft()` to replace `q.pop(0)`could save the running time, using list `graph = [[] for i in range(numCourses)]` to replace dict `graph = {i: [] for i in range(numCourses)}` also could save the running time

this algorithm could also be used to determine if the topological sorting result is unique, taking problem above for example:
```python
class Solution:
    """
    @param: numCourses: a total of n courses
    @param: prerequisites: a list of prerequisite pairs
    @return: true if can finish all courses or false
    """
    def findOrder(self, numCourses, prerequisites):
        # write your code here
        indegree = [0]*numCourses
        graph = {i: [] for i in range(numCourses)}
        for e,s in prerequisites:
            indegree[e] += 1
            graph[s].append(e)
        # Add courses with no prereqs to queue
        q = [i for i in range(numCourses) if indegree[i]==0]
        ans = []
        not_unique = False
        while q:
            if len(q) > 1:
                not_unique = True
            head = q.pop(0)
            ans.append(head)
            # Remove all outgoing edges
            for node in graph[head]:
                indegree[node] -= 1
                if indegree[node] == 0:
                    q.append(node)
        if len(ans) != numCourses:
            return 'cyclic'
        if not_unique:
            return 'not_unique'
        return 'unique'
```

## DFS


```python
class Solution:
    """
    @param: numCourses: a total of n courses
    @param: prerequisites: a list of prerequisite pairs
    @return: true if can finish all courses or false
    """
    def canFinish(self, numCourses, prerequisites):
        # write your code here
        graph = [[] for _ in xrange(numCourses)]
        visit = [0 for _ in xrange(numCourses)]
        for e, s in prerequisites:
            graph[s].append(e)
        ans = []
        for i in xrange(numCourses):
            if not self.dfs(graph,visit,i,ans):
                return []
        return ans

    def dfs(self,graph,visit,i,ans):
        if visit[i] == 2:
            return True
        if visit[i] == 1:
            return False
        visit[i] = 1
        for j in graph[i]:
            if not self.dfs(graph,visit,j,ans):
                return False
        visit[i] = 2
        ans.insert(0,i)
        return True
```
