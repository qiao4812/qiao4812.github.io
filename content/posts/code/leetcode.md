---
title: "Leetcode 力扣算法题解"
date: 2023-04-06T11:53:37+08:00
draft: false
tags: ["算法"]
categories: ["算法"]
---

# Leetcode 力扣算法题解

[867. 转置矩阵](https://leetcode.cn/problems/transpose-matrix/)

```python
class Solution:
    def transpose(self, matrix: List[List[int]]) -> List[List[int]]:
        return [[matrix[i][j]for i in range(len(matrix))] for j in range(len(matrix[0]))]
```
