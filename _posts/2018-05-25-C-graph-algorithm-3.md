---
layout: post
title: "图论算法3-Djikstra算法（C语言实现）"
tagline: ""
description: "图论算法3-Djikstra算法（C语言实现）"
category: algorithm
tags: [C,graph,algorithm]
last_updated: 2018-05-25
---

在上一篇文章[图论算法2-DFS与BFS及其应用（C语言实现）]({% post_url 2018-04-26-C-graph-algorithm-2 %})的最后有提到BFS能够用来寻找两个给定顶点间边的数量最小的路径，然而现实中我们更常遇到的是加权图的单源最短路径问题，针对这个问题有一个经典而且高效的算法——Djikstra算法（图论算法系列文章的代码实现：[adjacency_list.c]({{site.url}}/assets/adjacency_list.c)）。

## dijkstra算法
dijkstra算法是图论最短路径算法中非常经典的算法，在具体代码实现中的线索串联如下：
+ 参数初始化（函数输入：图结构体和起点位置；函数输出：每个顶点的前驱顶点数组和起点到各个顶点最短路径数值的数组）
+ 对起点的初始化
+ 遍历除起点外的每个顶点，每次找出一个顶点的最短路径 => 最外层循环
+ 遍历所有顶点，在未获取到最短路径的顶点中（flag==0），寻找离起点最近的顶点k（即长度数组的最小值，注意提前初始化min为极大值） => 内层循环1，实际作用是找到最短路径的下一个顶点k
+ 标记顶点k的flag为1，即已获取到从起始点至顶点k的最短路径
+ 获取到顶点k的最短路径之后，长度数组（即每个顶点到起始点的距离）和前驱顶点可能发生了变化，通过遍历所有顶点，判断当前顶点j到顶点k的距离加上内层循环1求得的顶点k到起点的最短距离是否小于目前顶点j到起点的距离，小于则更新 => 内层循环2
+ 最外层循环结束后，该算法的实质部分已经结束了，此时可以打印出起点到任一顶点的最短路径。

具体的代码实现（邻接表实现）：
```C++
/*
 * Dijkstra最短路径。
 * 即，统计图(G)中"顶点vs"到其它各个顶点的最短路径。
 *
 * 参数说明：
 *        G -- 图
 *       vs -- 起始顶点(start vertex)。即计算"顶点vs"到其它顶点的最短路径。
 *     prev -- 前驱顶点数组。即，prev[i]的值是"顶点vs"到"顶点i"的最短路径所经历的全部顶点中，位于"顶点i"之前的那个顶点。
 *     dist -- 长度数组。即，dist[i]是"顶点vs"到"顶点i"的最短路径的长度。
 */
void dijkstra(LGraph G, int vs, int prev[], int dist[])
{
    int i,j,k;
    int min;
    int tmp;
    int flag[MAX];      // flag[i]=1表示"顶点vs"到"顶点i"的最短路径已成功获取。
    
    // 初始化
    for (i = 0; i < G.vexnum; i++)
    {
        flag[i] = 0;                    // 顶点i的最短路径还没获取到。
        prev[i] = 0;                    // 顶点i的前驱顶点为0。
        dist[i] = get_weight(G, vs, i);  // 顶点i的最短路径为"顶点vs"到"顶点i"的权。
    }

    // 对"顶点vs"自身进行初始化
    flag[vs] = 1;
    dist[vs] = 0;

    // 遍历G.vexnum-1次；每次找出一个顶点的最短路径。
    for (i = 1; i < G.vexnum; i++)
    {
        // 寻找当前最小的路径；
        // 即，在未获取最短路径的顶点中，找到离vs最近的顶点(k)。
        min = INF;
        for (j = 0; j < G.vexnum; j++)
        {
            if (flag[j]==0 && dist[j]<min)
            {
                min = dist[j];
                k = j;
            }
        }
        // 标记"顶点k"为已经获取到最短路径
        flag[k] = 1;

        // 修正当前最短路径和前驱顶点
        // 即，当已经"顶点k的最短路径"之后，更新"未获取最短路径的顶点的最短路径和前驱顶点"。
        for (j = 0; j < G.vexnum; j++)
        {
            tmp = get_weight(G, k, j);
            tmp = (tmp==INF ? INF : (min + tmp)); // 防止溢出
            if (flag[j] == 0 && (tmp  < dist[j]) )
            {
                dist[j] = tmp;
                prev[j] = k;
            }
        }
    }

    // 打印dijkstra最短路径的结果
    printf("dijkstra(%c): \n", G.vexs[vs].data);
    for (i = 0; i < G.vexnum; i++)
        printf("  shortest(%c, %c)=%d\n", G.vexs[vs].data, G.vexs[i].data, dist[i]);
}
```