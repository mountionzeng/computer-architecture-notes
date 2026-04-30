# Module 12: Shortest Paths

> **教材章节**：Ch 14 · **对应周次**：Week 12
> **核心问题**：给定加权图，从一点到另一点（或所有点对）的**最短路径**是多少？

---

## 0. 知识地图

```
                Shortest Paths
                      │
    ┌─────────────────┼─────────────────┐
  单源                  单源              全源
  非负权                任意权（无负环）   任意权
    │                    │                 │
  Dijkstra          Bellman-Ford       Floyd-Warshall
  贪心              DP / 松弛 V-1 轮   DP / 三重循环
  O((V+E)logV)      O(VE)              O(V³)
```

---

## 1. 思考脉络

1. 核心操作：**松弛（Relaxation）**
   `if dist[v] > dist[u] + w(u,v): dist[v] = dist[u] + w(u,v)`
2. 非负权 → Dijkstra（贪心 + 堆）
3. 有负权（无负环）→ Bellman-Ford（松弛 V-1 轮）
4. 有负环 → 无最短路概念；Bellman-Ford 可检测
5. 全源 → Floyd-Warshall 或 n 次 Dijkstra
6. DAG → 拓扑序 + 线性松弛 → **O(V + E)**

---

## 2. 核心知识点

### 2.1 松弛操作

```
relax(u, v, w):
  if dist[v] > dist[u] + w:
    dist[v] = dist[u] + w
    parent[v] = u
```

所有最短路径算法本质都是"以不同顺序/次数做 relax"。

### 2.2 Dijkstra 算法 ⭐

**前提**：边权非负

```
dist[s] = 0; 其余 ∞
PQ 放 (0, s)
while PQ 非空:
  u = PQ.extractMin()
  for v in adj[u]:
    relax(u, v, w(u,v))
    PQ.update(v)
```

- **复杂度**：
  - 二叉堆：**O((V+E) log V)**
  - Fibonacci 堆：O(E + V log V)
- **贪心正确性**：一旦出堆，dist[u] 已是最终最短

⚠️ **有负权则失败**（一旦出堆不再更新）

### 2.3 Bellman-Ford 算法

```
重复 V-1 次：
  for 每条边 (u,v,w):
    relax(u, v, w)
再扫一轮：若还能松弛 → 存在负环
```

- **O(VE)**
- 允许**负权边**
- 可**检测负环**
- 可扩展为 SPFA（队列优化，期望更快但最坏仍 O(VE)）

### 2.4 DAG 最短路

- 先拓扑排序
- 按拓扑序对每个 u 松弛所有出边
- **O(V + E)**，可处理负权（无环保证）

### 2.5 Floyd-Warshall（全源最短路）

```
dp[i][j] = 不经过 {1..k} 外的中间节点时 i→j 最短路
dp[i][j] = min(dp[i][j], dp[i][k] + dp[k][j])
```

- **O(V³)**，可处理负权（无负环）
- 也可用于**传递闭包**（可达性矩阵）

### 2.6 Johnson 算法

全源最短路的稀疏图版本：
1. 用 Bellman-Ford 计算**重新加权**（使所有边非负）
2. 对每个源点跑 Dijkstra
- **O(VE + V² log V)**，稀疏图优于 Floyd-Warshall

### 2.7 A* 搜索（启发式）

- Dijkstra + 启发函数 h(n)
- 若 h 可采纳（admissible）且一致，则最优
- 广泛用于地图导航、游戏 AI

---

## 3. 算法对比

| 算法 | 适用 | 负权 | 负环检测 | 复杂度 |
|---|---|---|---|---|
| BFS | 单源无权 | — | — | O(V + E) |
| **Dijkstra** | 单源非负权 | ✗ | ✗ | O((V+E) log V) |
| **Bellman-Ford** | 单源任意权 | ✓ | **✓** | O(VE) |
| DAG 拓扑 | 单源 DAG | ✓ | — | O(V + E) |
| **Floyd-Warshall** | 全源任意权 | ✓ | ✓ | O(V³) |
| Johnson | 全源稀疏 | ✓ | ✓ | O(VE + V² log V) |

---

## 4. 常考陷阱

- ❌ Dijkstra 遇**负权**直接失败（即便无负环）
- ❌ Bellman-Ford 必须松弛 **V-1 轮**，不是 V 轮
- ❌ Floyd-Warshall 三重循环 **k 要在最外层**
- ❌ 松弛写成 `<` 不是 `>` 是常见 bug
- ❌ 有负环时"最短路"不存在（可无限减小）
- ❌ Dijkstra 用 PQ 时，要么 decrease-key，要么放多份然后跳过已处理的

---

## 5. 面试高频题

1. **网络延迟时间** —— 单源 Dijkstra
2. **廉价航班（K 站中转）** —— Bellman-Ford 变种
3. **概率最大路径** —— 修改松弛为乘法 / 取对数
4. **矩阵中的最短路径** —— BFS（无权）
5. **找到最终的安全状态** —— 反向图 + 拓扑

---

## 6. 三视角职业分析

- **工程师视角**：地图导航、网络路由（OSPF 用 Dijkstra，RIP 用 Bellman-Ford）
- **研究员视角**：A* 启发式、双向 Dijkstra、Contraction Hierarchies
- **系统架构师视角**：数据中心网络路由、CDN 路径选择

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Shortest path | 最短路径 |
| Relaxation | 松弛 |
| Dijkstra's algorithm | Dijkstra 算法 |
| Bellman-Ford | Bellman-Ford 算法 |
| Floyd-Warshall | Floyd-Warshall 算法 |
| Negative cycle | 负环 |
| Admissible heuristic | 可采纳启发函数 |

---

## 8. 关联模块

- 前置 → [[Module 4: Priority Queues and Heaps]]（Dijkstra 用堆）
- 前置 → [[Module 11: Graphs and Traversals]]（图基础）
- 方法学 → [[Module 8: The Greedy Method]]（Dijkstra 是贪心）、[[Module 10: Dynamic Programming]]（Floyd/Bellman 是 DP）
- 下一站 → [[Module 13: Minimum Spanning Trees]]
