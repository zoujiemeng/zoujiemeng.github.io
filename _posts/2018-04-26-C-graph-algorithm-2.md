---
layout: post
title: "图论算法2-DFS与BFS及其应用（C语言实现）"
tagline: ""
description: "图论算法2-DFS与BFS及其应用（C语言实现）"
category: algorithm
tags: [C,graph,algorithm]
last_updated: 2018-04-26
---

拿到一个图之后，尤其是复杂图，如何做一次遍历弄清图的结构就是一个首先要解决的问题。深度优先搜索（depth-first search, DFS）和广度优先搜索（breadth-first search, BFS）是遍历图的经典算法，其在研究图的基础性质中（图的连通性、图是否存在环）也有重要应用（注：图结构用邻接表实现，详见[图论算法1-基本图结构的实现（C语言实现）]({% post_url 2018-02-01-C-graph-algorithm-1 %})，图论算法系列文章的代码实现：[adjacency_list.c]({{site.url}}/assets/adjacency_list.c)）。

## DFS
深度优先搜索是图的基础算法之一，其主要应用包括：求图的连通块、判断图的连通性、图是否存在环。由于深度优先搜索的编程比较简单（递归），在能用DFS解决的问题中一般不用BFS。
### 算法实现
DFS的实现思路如下：
+ 初始化参数：顶点访问标记数组的初始化（函数里面第一步一定是初始化）
+ 最外层循环：遍历各顶点
+ 循环体：如果当前顶点i未被访问，则将该顶点i传入递归函数DFS
+ 递归函数：访问所有与i邻接的顶点，若其没有被访问，则继续递归调用，若其以及被访问，这访问下一个与i邻接的顶点。根据各顶点被访问的先后顺序可以构建出深度优先查找森林。具体代码实现如下
```C++
/*
 * 深度优先搜索遍历图的递归实现
 */
static void DFS(LGraph G, int i, int *visited)
{
    ENode *node;

    visited[i] = 1;
    printf("%c ", G.vexs[i].data);
    node = G.vexs[i].first_edge;
    while (node != NULL)
    {
        if (!visited[node->ivex])
            DFS(G, node->ivex, visited);
        node = node->next_edge;
    }
}

/*
 * 深度优先搜索遍历图
 */
void DFSTraverse(LGraph G)
{
    int i;
    int visited[MAX];       // 顶点访问标记

    // 初始化所有顶点都没有被访问
    for (i = 0; i < G.vexnum; i++)
        visited[i] = 0;

    printf("DFS: ");
    for (i = 0; i < G.vexnum; i++)
    {
        if (!visited[i])
            DFS(G, i, visited);
    }
    printf("\n");
}
```
DFS实际的效率是非常高的，对于邻接链表实现的结构，其时间复杂度为$\Theta(|V|+|E|)$，其中|V|和|E|分别为图的顶点和边的数量。因此DFS实际上是以线性时间为代价解决了图的遍历问题。

### 判断图是否具有连通性
如果在一个无向图中从每一个顶点到每个其他顶点都存在一条路径，则称该无向图是连通的(connected)。利用DFS能够很直观的判断图是否具有连通性：只需将第一个顶点作为输出参数给入DFS递归（理论上任意一点作为起点均可），结束之后判断是否所有顶点都已被访问，如果有未访问的顶点，则该图是不连通的。对于有向图，如果仍然有这个性质，则成为强连通的(strongly connected)。如果一个有向图不是强连通的，但是其基础图（边上去掉方向形成的图）是连通的，则该有向图成为弱连通的(weakly connected)。
```C++
bool judge_connectivity_using_DFS(LGraph G)
{
	int i;
	int visited[MAX];       // 顶点访问标记

	// 初始化所有顶点都没有被访问
	for (i = 0; i < G.vexnum; i++)
		visited[i] = 0;
	printf("DFS of vertex[0]: ");
	// 从顶点0开始访问
	DFS(G, 0, visited);
	printf("\n");
	//
	for (i = 0; i < G.vexnum; i++)
	{
		if (visited[i] == 0)
		{
			return false;
		}
	}
	return true;
}
```
实现的思路很简单，在DFS算法的外层函数中将外层循环遍历所有顶点改成初始顶点，随后判断是否所有顶点都被访问过即可。事实上，DFS还能实现图是否存在环的检测以及拓扑排序。不过需要将DFS进行小的改造，核心思路是将各顶点在DFS中可能存在的状态分为3种（0，1，2），白色代表未访问，灰色代表已访问，黑色代表该顶点的所有邻接点都已访问过。如此改造之后，在环的检测中，如果发现某一顶点有一个指向灰色顶点的边，那么可以判断该图存在环。另外，按照顶点标记为黑色的先后顺序，可以做拓扑排序，即，顶点越先被标记为黑色，其在拓扑序列中越靠后。

## BFS
顾名思义，广度优先搜索体现了一种全面谨慎的思想，它按照一种同心圆的方式，首先访问和初始顶点邻接的所有顶点，然后再访问与初始顶点的邻接顶点邻接的所有顶点，一层一层往外遍历，直到访问完所有与初始顶点在同一个连通分量中的所有顶点，若还存在未访问顶点，则需选择其中一个继续访问（可根据此性质判断图是否具有连通性）。
### 代码实现
实现思路串联：
+ 声明并初始化相关变量（核心算法用队列实现，队列用数组实现，用head和rear两个变量指明目前队列的队头和队尾位置，visit数组用来指示顶点是否被访问，初始化为0）
+ 外层循环，遍历所有顶点，如果该顶点未访问，则将该顶点visit至为1，且将其入队。
+ 队列处理，当队列不为空（即head！=rear）时，将队头的顶点j出队，同时边结构体指向j的第一条边，随后顶点j邻接的顶点全部入队，visit置为1，再一一走出队流程
+ 外层循环结束，算法结束
```C++
/*
 * 广度优先搜索（类似于树的层次遍历）
 */
void BFS(LGraph G)
{
    int head = 0;
    int rear = 0;
    int queue[MAX];     // 辅组队列
    int visited[MAX];   // 顶点访问标记
    int i, j, k;
    ENode *node;

    for (i = 0; i < G.vexnum; i++)
        visited[i] = 0;

    printf("BFS: ");
    for (i = 0; i < G.vexnum; i++)
    {
        if (!visited[i])
        {
            visited[i] = 1;
            printf("%c ", G.vexs[i].data);
            queue[rear++] = i;  // 入队列
        }
        while (head != rear) 
        {
            j = queue[head++];  // 出队列
            node = G.vexs[j].first_edge;
            while (node != NULL)
            {
                k = node->ivex;
                if (!visited[k])
                {
                    visited[k] = 1;
                    printf("%c ", G.vexs[k].data);
                    queue[rear++] = k;
                }
                node = node->next_edge;
            }
        }
    }
    printf("\n");
}
```
可以看出，对于邻接链表实现的BFS算法，其复杂度为$\Theta(|V|+|E|)$，与DFS的效率相同。BFS的另一个重要应用是可以用来寻找两个给定顶点间边的数量最小的路径。