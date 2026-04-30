# Module 13: Minimum Spanning Trees

> **教材章节**：Ch 15 · **对应周次**：Week 13 · **期末节点**
> **核心问题**：连通所有顶点的**总权值最小**的无环子图是什么？

---

## 0. 知识地图

```
              Minimum Spanning Tree
                       │
        ┌──────────────┼──────────────┐
    两大性质         三大算法          应用
        │              │               │
   Cut Property    Kruskal          网络设计
   Cycle Property  Prim-Jarnik      聚类
                   Borůvka          逼近算法 (TSP)
```

---

## 1. 思考脉络

1. 生成树 = 连通 + 无环 + 含所有 V 顶点（E = V-1）
2. 加权图中权重最小的生成树 = MST
3. 两条关键性质奠基所有算法：
   - **Cut property**：任一割中最轻边必在某 MST
   - **Cycle property**：任一环中最重边**不**在任何 MST
4. 三种实现贪心策略：
   - Kruskal：按边权排序，Union-Find 判环
   - Prim：从一点扩展，类似 Dijkstra
   - Borůvka：多起点并行

---

## 2. 核心知识点

### 2.1 生成树基本事实

- n 顶点的生成树有 **n-1** 条边
- 连通图必有生成树；若 G 含权重则 MST 可能不唯一（权重相同）
- **MST 权重唯一**（即使多棵 MST）

### 2.2 Cut Property（割性质）

> 对图的任一割（顶点划分为两个非空集合），横跨两集合的边中**权重最小的**必属于某个 MST。

是所有 MST 算法"贪心可行性"的基础。

### 2.3 Cycle Property（环性质）

> 图中任一环里**权重最大**的那条边**不会**出现在任何 MST 中。

用于"删边"类算法的正确性。

### 2.4 Kruskal 算法

```
sort edges by weight
for each edge (u,v,w) in 排序后:
  if find(u) != find(v):
    加入 MST
    union(u, v)
```

- 数据结构：**Union-Find**
- **O(E log E) = O(E log V)**
- 适合稀疏图

### 2.5 Prim-Jarnik 算法

```
从任一起点开始
PQ 放起点
while PQ 非空:
  u = PQ.extractMin()
  加入 MST
  for v in adj[u], v 不在 MST:
    PQ.decreaseKey(v, w(u,v))
```

- 数据结构：**最小堆**（优先队列）
- **O((V + E) log V)** 用二叉堆
- 用斐波那契堆可降到 **O(E + V log V)**
- 适合稠密图

### 2.6 Borůvka 算法

每轮对每个"当前连通分量"找**外部最轻边**，同时加入 MST。
- 每轮分量数至少减半 → **O(log V)** 轮
- **O(E log V)**
- 并行友好（GPU / 分布式 MST）

---

## 3. 算法对比

| 算法 | 思路 | 数据结构 | 复杂度 | 适用 |
|---|---|---|---|---|
| **Kruskal** | 按边权排序贪心 | Union-Find | O(E log V) | 稀疏图 |
| **Prim** | 从一点扩展 | 最小堆 | O((V+E) log V) | 稠密图 |
| **Borůvka** | 分量并行扩展 | — | O(E log V) | 并行场景 |

---

## 4. 常考陷阱

- ❌ MST **不等于** Shortest Path Tree —— 目标不同
- ❌ 多棵边权相同的 MST 权值相同，但具体树不同
- ❌ Kruskal 判断加边是否成环的**唯一正确工具**是 Union-Find
- ❌ Prim 要从**同一棵树**扩展，不能跨多棵
- ❌ 有向图的 MST 是"最小生成树形图（Arborescence）"，算法不同（Chu–Liu/Edmonds）

---

## 5. 面试高频题

1. **连接所有点的最小费用** —— Prim / Kruskal
2. **找到关键连接（桥）** —— Tarjan（非 MST 但同类思维）
3. **水资源分配 / 建村问题** —— 抽象成 MST
4. **Kruskal 手画** —— 考 Union-Find 使用
5. **次小生成树** —— MST + 枚举替换边

---

## 6. 三视角职业分析

- **工程师视角**：网络布线、电路布线的经典优化问题
- **研究员视角**：MST 在近似 TSP、聚类分析（single-linkage）中的核心作用
- **系统架构师视角**：数据中心主干网络拓扑设计、分布式共识中的广播树

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Spanning tree | 生成树 |
| Minimum Spanning Tree (MST) | 最小生成树 |
| Cut / Cycle property | 割 / 环性质 |
| Kruskal's algorithm | Kruskal 算法 |
| Prim's algorithm | Prim / Prim-Jarnik 算法 |
| Borůvka's algorithm | Borůvka 算法 |
| Arborescence | 有向生成树 |

---

## 8. 关联模块

- 前置 → [[Module 4: Priority Queues and Heaps]]（Prim 用堆）
- 前置 → [[Module 6: Union-Find Structures]]（Kruskal 用并查集）
- 方法学 → [[Module 8: The Greedy Method]]（三者都是贪心）
- 对比 → [[Module 12: Shortest Paths]]（目标不同：MST vs SPT）
