---
title: '从八皇后问题到回溯算法'
date: 2020-08-31 10:00:00
tags: [算法,leetcode]
categories: algorithm
published: true
hideInList: true
feature: 
isTop: false
---

> 回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就 “回溯” 返回，尝试别的路径。回溯法是一种选优搜索法，按选优条件向前搜索，以达到目标。但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法，而满足回溯条件的某个状态的点称为 “回溯点”。许多复杂的，规模较大的问题都可以使用回溯法，有“通用解题方法”的美称。

回溯算法的基本思想是：从一条路往前走，能进则进，不能进则退回来，换一条路再试。

**解决一个回溯问题，实际上就是一个决策树的遍历过程**。

1、路径：也就是已经做出的选择。

2、选择列表：也就是你当前可以做的选择。

3、结束条件：也就是到达决策树底层，无法再做选择的条件。

回溯算法最经典的问题是八皇后问题

> 问题表述为：在8×8格的[国际象棋](https://baike.baidu.com/item/国际象棋/80888)上摆放8个[皇后](https://baike.baidu.com/item/皇后/15860305)，使其不能互相攻击，即任意两个皇后都不能处于同一行、同一列或同一斜线上，问有多少种摆法。

<!-- more -->

```java
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return
    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

核心就是 for 循环里面的递归，在递归调用之前「做选择」，在递归调用之后「撤销选择」

```java
class Solution {
    public List<List<String>> solveNQueens(int n) {
        boolean[][] help = new boolean[3][n*2];
        int times = 0;
        List<List<String>> res = new ArrayList<>();
        int[] solutionnum = new int[n];
        return backtrack(help,times,n,solutionnum,res);
    }

    private List<List<String>> backtrack(boolean[][] help, int times, int n, int[] solutionnum, List<List<String>> res) {
       if(times == n){   //if 满足结束条件:
            List<String> solution = new ArrayList<>();
            for (int i = 0; i < n; i++) {
                solution.add( String.join("", Collections.nCopies(solutionnum[i],"."))
                + "Q" + String.join("", Collections.nCopies(n-solutionnum[i]-1,".")));
            }
            res.add(solution);  //将满足的方法添加到结果中
        }
        for (int i = 0; i < n; i++) {  //for 选择 in 选择列表:
            if (!(help[0][i] || help[1][n+i-times] || help[2][i+times])){
                help[0][i] = help[1][n+i-times] = help[2][i+times] = true;  //做选择
                solutionnum[times] = i;
                backtrack(help,times+1,n,solutionnum,res);
                help[0][i] = help[1][n+i-times] = help[2][i+times] = false;  //撤销选择
            }
        }
        return res;
    }
}
```

回溯算法的思想并不复杂，但是我觉得重要的是如何处理做选择和撤销选择这样一个流程。

易错点新手很容易会因为浅拷贝导致结果出错；

- 浅拷贝可能出现的第一个地点是在将`满足的一种方法添加到结果中`，这里的`结果`就很有很可能是引用类型，如果直接添加就会导致输出的结果错误。

- 另外一个容易出错的部分是在做选择和撤销选择的处理上：有一种通用选择是每次选择都`深拷贝`一次路径，这样虽然不用考虑之后的撤销选择的问题，但是会导致程序需要的内存过大。最好的方法还是做选择和撤销选择。

  

