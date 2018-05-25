---
layout: post
title: "图论算法1-基本图结构的实现（C语言实现）"
tagline: ""
description: "图论算法1-基本图结构的实现（C语言实现）"
category: algorithm
tags: [C,graph,algorithm]
last_updated: 2018-02-04
---

图论及相关算法是算法世界里面重要的组成部分，相比排序算法，图论算法与现实世界靠的更近，应用也相当广泛。图论相关算法的内容比较多，这是第一篇，主要讲述基础图概念及其基本数据结构的实现(图论算法系列文章的代码实现：[adjacency_list.c]({{site.url}}/assets/adjacency_list.c))。

## 图的代码实现
一般来说，表示一个图有两种方法，邻接矩阵和邻接链表。由于在大部分的实际应用中，图是稀疏的，因此，邻接链表是图的标准表示方法。另外，个人认为邻接矩阵的内存占用和可扩展性是一个比较大的问题。
### 邻接矩阵
邻接矩阵的结构体定义和图的创建的代码实现如下
```C++
#define MAX         100                 // 矩阵最大容量
#define INF         (~(0x1<<31))        // 最大值(即0X7FFFFFFF)
#define LENGTH(a)   (sizeof(a)/sizeof(a[0]))

// 邻接矩阵
typedef struct _graph
{
    char vexs[MAX];       // 顶点集合
    int vexnum;           // 顶点数
    int edgnum;           // 边数
    int matrix[MAX][MAX]; // 邻接矩阵
}Graph, *PGraph;

// 创建图(用已提供的矩阵)
Graph* create_example_graph()
{
    char vexs[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
    int matrix[][9] = {
             /*A*//*B*//*C*//*D*//*E*//*F*//*G*/
      /*A*/ {   0,  12, INF, INF, INF,  16,  14},
      /*B*/ {  12,   0,  10, INF, INF,   7, INF},
      /*C*/ { INF,  10,   0,   3,   5,   6, INF},
      /*D*/ { INF, INF,   3,   0,   4, INF, INF},
      /*E*/ { INF, INF,   5,   4,   0,   2,   8},
      /*F*/ {  16,   7,   6, INF,   2,   0,   9},
      /*G*/ {  14, INF, INF, INF,   8,   9,   0}};
    int vlen = LENGTH(vexs);
    int i, j;
    Graph* pG;
    
    // 输入"顶点数"和"边数"
    if ((pG=(Graph*)malloc(sizeof(Graph))) == NULL )
        return NULL;
    memset(pG, 0, sizeof(Graph));

    // 初始化"顶点数"
    pG->vexnum = vlen;
    // 初始化"顶点"
    for (i = 0; i < pG->vexnum; i++)
        pG->vexs[i] = vexs[i];

    // 初始化"边"
    for (i = 0; i < pG->vexnum; i++)
        for (j = 0; j < pG->vexnum; j++)
            pG->matrix[i][j] = matrix[i][j];

    // 统计边的数目
    for (i = 0; i < pG->vexnum; i++)
        for (j = 0; j < pG->vexnum; j++)
            if (i!=j && pG->matrix[i][j]!=INF)
                pG->edgnum++;
    pG->edgnum /= 2;

    return pG;
}
```
### 邻接链表
邻接链表（简称邻接表）的实现如下，需要注意的是，图的输入核心是边（edge）的信息，由于两点确定一条线段，因此边的信息由两个顶点和边的权定义（事实上，这里的权应该是可变的，或者说是可以有多个权值来定义一条边，典型的应用就是交通最短路径搜索时一般有两个选项：最短距离和最短时间）。因此开头先定义边的结构体，主要有三个成员：终点、权值、指向下一条边的结构体的指针（起点在表头定义），当然可以根据情况加该边的文字信息，例如在交通应用中，边代表的“某某路”。然后定义邻接表中每个顶点的开始部分，很简单，就是顶点的信息（一般为字符型，人能看懂的信息，如顶点代号“A”之类的）以及指向第一条边的指针。最后是图的整体定义，常规成员：顶点数目、边的数目、各顶点的邻接表数组。

创建图的代码实现如下，需要注意的是，这里的图是无向图，等价于双向图，因此创建的时候初始化了node1和node2，如果是单向图的话初始化node1就行（前提是输入中第一个顶点定义为起点，第二个顶点定义为终点）。

```C++
// 邻接表中表对应的链表的顶点
typedef struct _ENode
{
    int ivex;                   // 该边的终点的位置
    int weight;                 // 该边的权
    struct _ENode *next_edge;   // 指向下一条边的指针
}ENode, *PENode;

// 邻接表中表的顶点
typedef struct _VNode
{
    char data;              // 顶点信息
    ENode *first_edge;      // 指向第一条依附该顶点的边
}VNode;

// 邻接表
typedef struct _LGraph
{
    int vexnum;             // 图的顶点的数目
    int edgnum;             // 图的边的数目
    VNode vexs[MAX];
}LGraph;

// 边的结构体
typedef struct _edata
{
    char start; // 边的起点
    char end;   // 边的终点
    int weight; // 边的权重
}EData;

// 顶点
static char  gVexs[] = {'A', 'B', 'C', 'D', 'E', 'F', 'G'};
// 边
static EData gEdges[] = {
  // 起点 终点 权
    {'A', 'B', 12}, 
    {'A', 'F', 16}, 
    {'A', 'G', 14}, 
    {'B', 'C', 10}, 
    {'B', 'F',  7}, 
    {'C', 'D',  3}, 
    {'C', 'E',  5}, 
    {'C', 'F',  6}, 
    {'D', 'E',  4}, 
    {'E', 'F',  2}, 
    {'E', 'G',  8}, 
    {'F', 'G',  9}, 
};

/*
 * 返回ch在matrix矩阵中的位置
 */
static int get_position(LGraph G, char ch)
{
    int i;
    for(i=0; i<G.vexnum; i++)
        if(G.vexs[i].data==ch)
            return i;
    return -1;
}

/*
 * 将node链接到list的末尾
 */
static void link_last(ENode *list, ENode *node)
{
    ENode *p = list;

    while(p->next_edge)
        p = p->next_edge;
    p->next_edge = node;
}

/*
 * 创建邻接表对应的图(用已提供的数据)
 */
LGraph* create_example_lgraph()
{
    char c1, c2;
    int vlen = LENGTH(gVexs);
    int elen = LENGTH(gEdges);
    int i, p1, p2;
    int weight;
    ENode *node1, *node2;
    LGraph* pG;

    if ((pG=(LGraph*)malloc(sizeof(LGraph))) == NULL )
        return NULL;
    memset(pG, 0, sizeof(LGraph));

    // 初始化"顶点数"和"边数"
    pG->vexnum = vlen;
    pG->edgnum = elen;
    // 初始化"邻接表"的顶点
    for(i=0; i<pG->vexnum; i++)
    {
        pG->vexs[i].data = gVexs[i];
        pG->vexs[i].first_edge = NULL;
    }

    // 初始化"邻接表"的边
    for(i=0; i<pG->edgnum; i++)
    {
        // 读取边的起始顶点,结束顶点,权
        c1 = gEdges[i].start;
        c2 = gEdges[i].end;
        weight = gEdges[i].weight;

        p1 = get_position(*pG, c1);
        p2 = get_position(*pG, c2);

        // 初始化node1
        node1 = (ENode*)malloc(sizeof(ENode));
        node1->ivex = p2;
        node1->weight = weight;
        node1->next_edge = NULL;
        // 将node1链接到"p1所在链表的末尾"
        if(pG->vexs[p1].first_edge == NULL)
            pG->vexs[p1].first_edge = node1;
        else
            link_last(pG->vexs[p1].first_edge, node1);
        // 初始化node2
        node2 = (ENode*)malloc(sizeof(ENode));
        node2->ivex = p1;
        node2->weight = weight;
        // 将node2链接到"p2所在链表的末尾"
        if(pG->vexs[p2].first_edge == NULL)
            pG->vexs[p2].first_edge = node2;
        else
            link_last(pG->vexs[p2].first_edge, node2);
    }
    return pG;
}
```




