# Module 11: Graphs and Traversals

> **教材章节**：Ch 13 · **对应周次**：Week 11
> **核心问题**：如何表示图？如何系统性访问所有顶点？DFS / BFS 各能解什么问题？

---

## 0. 知识地图

```
                  Graphs
                     │
        ┌────────────┼────────────┐
     表示法        遍历             应用
        │            │               │
   邻接表/矩阵   ┌────┴────┐     拓扑排序
                BFS        DFS    SCC (强连通)
                层序       深度   双连通分量
                最短路(无权) 回溯   有向环检测
                                   割点/桥
```

---

## 1. 思考脉络

1. 图 = 顶点集 V + 边集 E
2. 两种表示：**邻接表**（稀疏，O(V+E) 空间）/ **邻接矩阵**（稠密，O(V²)）
3. 两种遍历：
   - **BFS** 层序扩展 → 无权最短路
   - **DFS** 深度优先 → 时间戳用途广（拓扑、SCC、割点）
4. 所有图算法都基于这两种遍历之一

---

## 2. 核心知识点

### 2.0 先记全称和直觉

| 缩写 | 英文全称 | 中文 | 核心直觉 | 常用数据结构 | 最常见用途 |
|---|---|---|---|---|---|
| BFS | Breadth-First Search | 广度优先搜索 | 先看近的，一层一层向外扩散 | Queue | 无权图最短路径、层序遍历、任务按距离扩展 |
| DFS | Depth-First Search | 深度优先搜索 | 一条路先走到底，再回头换路 | Stack / 递归调用栈 | 回溯、环检测、拓扑排序、连通分量 |

一句话记忆：**BFS 像水波纹，先扩一圈再扩下一圈；DFS 像走迷宫，一条路走到底再回头。**

### 2.1 表示法对比

| 表示 | 空间 | 邻居枚举 | 查边 (u,v) |
|---|---|---|---|
| 邻接表 | O(V + E) | O(deg(v)) | O(deg(u)) |
| 邻接矩阵 | O(V²) | O(V) | **O(1)** |

稀疏图用邻接表，稠密图 / 频繁查边用矩阵。

### 2.2 BFS（广度优先搜索）

```
BFS(s):
  queue = [s]; visited[s] = true
  while queue 非空:
    u = queue.pop_front()
    for v in adj[u]:
      if not visited[v]:
        visited[v] = true
        dist[v] = dist[u] + 1
        queue.push_back(v)
```

- **O(V + E)**
- 求**无权最短路**、层序、二分图判定

### 2.3 DFS（深度优先搜索）

```
DFS(u):
  visited[u] = true
  time_in[u] = ++clock
  for v in adj[u]:
    if not visited[v]:
      DFS(v)
  time_out[u] = ++clock
```

- **O(V + E)**
- 边分类：**树边 / 返祖边 / 前向边 / 交叉边**
- 时间戳可推导祖孙关系

### 2.4 拓扑排序（Topological Sort）

适用于 **DAG**（有向无环图）。

**方法 1 · Kahn 算法**（BFS）：
- 维护入度为 0 的队列，取出、删边、更新入度
- O(V + E)

**方法 2 · DFS 逆后序**：
- DFS 完成时压栈
- 最终栈中从上到下即拓扑序
- O(V + E)

**用途**：任务调度、课程安排

### 2.5 强连通分量（SCC）

**定义**：有向图中极大的"任意两点可互达"子集

**Kosaraju 算法**：
1. 对原图 DFS，按**完成时间逆序**压栈
2. 反转图
3. 按栈顺序再 DFS，每次 DFS 得到一个 SCC
- O(V + E)

**Tarjan 算法**：单次 DFS + low-link 值，O(V + E)

### 2.6 双连通分量 · 割点 · 桥

- **割点（Articulation point）**：删除后图不连通
- **桥（Bridge）**：删除后图不连通的边
- Tarjan 算法：维护 `disc[v]`、`low[v]`
  - `low[v] = min(disc[v], disc[祖先], low[子节点])`
  - u 是割点 ⇔ 根且 ≥ 2 子 / 非根且 ∃子 `low[子] ≥ disc[u]`

### 2.7 有向环检测

- DFS 过程中若遇到**正在访问**（灰色）的节点 → 有环
- 或 Kahn 拓扑排序后若仍有未处理节点 → 有环

### 2.8 二分图判定

- BFS / DFS 染色：相邻节点颜色不同
- 遇到冲突 → 非二分图
- O(V + E)

---

## 3. 算法速查

| 问题 | 算法 | 复杂度 |
|---|---|---|
| 无权最短路 | BFS | O(V + E) |
| 拓扑排序 | Kahn / DFS 逆后序 | O(V + E) |
| 强连通分量 | Kosaraju / Tarjan | O(V + E) |
| 割点 / 桥 | Tarjan | O(V + E) |
| 二分图判定 | BFS 染色 | O(V + E) |
| 环检测 | DFS 着色 | O(V + E) |

---

## 4. 常考陷阱

- ❌ **有向图** vs 无向图的 DFS 边分类规则不同
- ❌ 拓扑排序要求**无环**，有环就没有拓扑序
- ❌ BFS 求最短路只对**无权图**成立；有权要用 Dijkstra
- ❌ 邻接表的空间是 O(V+E)，不是 O(E)
- ❌ Kosaraju **反向图** 和 **逆序 DFS** 两个步骤顺序不能颠倒

---

## 5. 面试高频题

1. **岛屿数量 / 岛屿面积** —— DFS / BFS
2. **克隆图** —— BFS/DFS + 哈希
3. **课程表（I / II）** —— 环检测 + 拓扑排序
4. **太平洋大西洋水流问题** —— 双 BFS
5. **单词接龙** —— BFS 构图
6. **被围绕的区域** —— 边界 DFS

---

## 6. 三视角职业分析

- **工程师视角**：LeetCode 图论必备；地图导航、社交网络
- **研究员视角**：图算法是图论 + 数据结构的完美交叉
- **系统架构师视角**：依赖管理（Maven/npm 拓扑排序）、编译链接、任务调度器

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Vertex / Edge | 顶点 / 边 |
| Adjacency list / matrix | 邻接表 / 矩阵 |
| BFS / DFS | 广度 / 深度优先搜索 |
| Directed / Undirected | 有向 / 无向 |
| DAG | 有向无环图 |
| Topological sort | 拓扑排序 |
| Strongly Connected Component | 强连通分量 |
| Articulation point / Bridge | 割点 / 桥 |
| Bipartite | 二分图 |

---

## 8. 关联模块

- 前置 → [[Module 2: Basic Data Structures]]（队列、栈）
- 下一站 → [[Module 12: Shortest Paths]]、[[Module 13: Minimum Spanning Trees]]
- 工具 → [[Module 6: Union-Find Structures]]（无向图连通性的动态版本）
