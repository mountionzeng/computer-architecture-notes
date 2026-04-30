# Module 6: Union-Find Structures

> **教材章节**：Ch 7 · **对应周次**：Week 6
> **核心问题**：动态维护**不相交集合**，支持"合并两个集合 + 查询某元素属于哪个集合"。

---

## 0. 知识地图

```
              Union-Find (DSU / Partition)
                        │
        ┌───────────────┼───────────────┐
     实现方式           优化            应用
        │                │               │
   ┌────┴────┐      ┌────┴────┐    ┌────┴────┐
 List-based Tree-based Union by  Path    社交网络  迷宫  渗流
             Quick- Rank/Size Compr.  连通分量  生成  理论
             Union        │
                    摊还 O(α(n))
```

---

## 1. 思考脉络

1. **问题起点**：给 n 个人、m 条关系，问"两人是否属于同一社群？""合并两个社群"
2. **关键洞察**：Union-Find 管的是**归属（membership）**，不是排列顺序——哈希/排序全是错误方向
3. **旧名字**：Union-Find 以前叫 **Partition（划分）**——名字变了，本质是把全集划分成若干不相交子集
4. **最简单**：数组直存组号（Quick-Find）→ find O(1)，union O(n) 太慢
5. **改成链表**：每个集合用链表存成员，成员指向集合名字 → find O(1)，union 可以优化
6. **改成树形**：每个元素指向"父亲"，同组共享根（Quick-Union）→ 统一 find 和 union
7. **树可能退化**为链 → **按秩/大小合并（Union by Rank/Size）** 保持树矮：O(log n)
8. **find 时顺手摊平** → **路径压缩（Path Compression）**：后续访问近 O(1)
9. **两者结合** → 摊还 **O(α(n))**，对所有实际数据 α(n) ≤ 4，视同常数

> **教授原话**："Union-Find 的命名是有意为之——强调你在做什么：union 两个集合，find 某元素属于哪个集合。find 不是在寻找元素是否存在，我们早就知道它存在了。"

> **常见误区**：Java 的 `HashSet`、Python 的 `set` 都用哈希实现——它们只管"元素在不在"，完全没有实现 Union-Find 的**归属**语义。这是整个编程社区中对"集合"最普遍的误解之一。

---

## 2. 核心知识点

### 2.1 ADT 接口与核心概念

三个核心操作：

| 操作 | 语义 | 返回值 |
|---|---|---|
| `makeSet(x)` | 建立只含 x 的单元素集合 | x 在集合中的位置 |
| `find(x)` | 返回 x **所属集合的代表（名字/根）** | 集合的代表元 |
| `union(A, B)` | 合并 A 和 B 所在的两个集合 | 新集合的引用 |

**最重要的语义澄清——"find 不是在找 x"**：

> "Find does not mean finding the element. Find means finding the **set** it belongs to."

`find(x)` 的问题是："x 属于哪个组？"而不是"x 在不在这里？"——x 一定存在，否则你根本不应该调用 find。

类比：你不是在找"张三这个人"，而是在问"张三属于哪个部门"。部门是你真正关心的答案。

**不相交集合（Disjoint Sets）的本质**：把全集所有元素划分成若干**互不重叠**的子集，每个子集用一个"代表元"来标识。

---

### 2.2 List-based 实现（链表版）

**设计**：每个集合 = 一个**集合名（代表元）+ 成员链表**。每个成员节点都有一个指针，直接指向自己所属集合的名字。

```
集合 A = {41, 7}：
    [A 名字] ← [41] ← [7]
    每个成员节点都有指针 → [A 名字]

集合 B = {2, 6, 3, 9}：
    [B 名字] ← [2] ← [6] ← [3] ← [9]
    所有成员节点指针 → [B 名字]
```

**三种操作的代价**：

- `makeSet(x)`：建一个名字节点 + x 节点，x 指向名字 → **O(1)**
- `find(x)`：跟着 x 的指针走到名字节点 → **O(1)**（一步到位）
- `union(A, B)`：**把小集合的所有成员指针改指到大集合的名字**

**为什么小并入大？** 因为每次一个元素被"重定向"，它所在的新集合至少是原集合的两倍大。因此任何单个元素最多被重定向 **log n 次**。

**摊还分析**：

- n 个元素，每个最多被重定向 log n 次
- 每次重定向代价 O(1)
- **总 union 代价：O(n log n)**
- m 次 find 代价：O(m)（每次 O(1)）
- **总计：O(n log n + m)**

| 操作 | 单次代价 | n 次总摊还 |
|---|---|---|
| makeSet | O(1) | — |
| find | **O(1)** | O(m) |
| union（单次最坏）| O(min 集合大小) | **O(n log n) 总计** |

**优点**：find 严格 O(1)。**缺点**：union 单次最坏 O(n)；需要额外链表结构。

---

### 2.3 Tree-based 实现（Quick-Union）

更优雅的统一方案：每个集合用一棵树表示，**树的根就是集合的"代表名字"**。

**关键设计**：
- 每个元素 x 存一个 `parent[x]` 指针
- 根节点的 parent 指向自身：`parent[root] = root`
- 所有属于同一集合的元素共享同一个根

```python
def makeSet(x):
    parent[x] = x     # x 是只有自己的树，自指即为根
    size[x] = 1

def find(x):
    while parent[x] != x:
        x = parent[x]
    return x           # 走到根就是集合代表

def union(x, y):
    rx = find(x)      # 先找到两棵树的根
    ry = find(y)
    if rx == ry: return    # 已在同一集合
    parent[rx] = ry        # 把一棵根挂到另一棵根下面
```

**朴素 Quick-Union 的问题**：

按顺序 `union(1,2), union(2,3), union(3,4)...` 会退化成一条链，树高 h = O(n)，导致 find = O(n)。

```
坏情况：1 → 2 → 3 → 4 → 5    find(1) 需走 4 步
```

---

### 2.4 优化 1：Union by Rank / Size（按秩/大小合并）

**核心思想**：合并时，**把小树的根挂到大树的根下面**，永远不把大树挂到小树下。

**为什么叫 Rank 不叫 Height？**

Rank 初始等于树高，但一旦加上路径压缩，它就不再是真实高度了——路径压缩会把树压矮，但 rank 不会更新。所以用 rank 而不是 height，强调它是**高度的上界估计**，而不是精确高度。

把 rank 理解成这棵树的"资历"或"等级"——合并时等级低的并入等级高的，等级相同时任选一方，并把胜者等级 +1。

```python
def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry: return

    if rank[rx] < rank[ry]:
        parent[rx] = ry          # 小秩挂大秩
    elif rank[rx] > rank[ry]:
        parent[ry] = rx
    else:                        # 秩相等：任选一为主，主的秩 +1
        parent[ry] = rx
        rank[rx] += 1
```

**关键规则**：只有两棵树秩相等时合并，合并结果的秩才会 +1。不等时，矮的挂高的下面，高的秩不变。

**效果**：
- **数学性质**：rank 为 s 的树至少包含 2^(s-2) 个节点（归纳可证）
- 由此推出：**任何树的高度 ≤ log n**
- find 和 union 都是 **O(log n)**

**Union by Size 变体**：很多工程实现用节点总数（size）代替 rank，逻辑完全一样——小树并大树，语义更直白：

```python
def union(x, y):
    rx, ry = find(x), find(y)
    if rx == ry: return
    if size[rx] < size[ry]:
        parent[rx] = ry;  size[ry] += size[rx]
    else:
        parent[ry] = rx;  size[rx] += size[ry]
```

---

### 2.5 优化 2：Path Compression（路径压缩）

**核心洞察**：find(x) 已经从 x 一路爬到了根——路径上每个节点都"知道了根是谁"。何不顺手把它们全部直接连到根？这样下次 find 这些节点就是 O(1)。

```python
def find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])   # 递归：让 x 直接指向根
    return parent[x]
```

这段代码做了两件事：向上爬找到根，同时在回溯时把路径上每个节点的 parent 改成根。

![[path_compression_visualization.png]]

> 💡 想点按钮逐步看上爬和压缩的过程？👉 [点这里在浏览器打开](path_compression_visualization.html)

**直觉**：第一次 find(F) 走了 3 步，代价是 3。但爬完之后，F、C、B 都直接认识根了——下次 find 任何一个都是 1 步。一次贵的操作让后续所有访问变快，这正是**摊还分析**要捕捉的模式。

**注意**：路径压缩后，rank **不再等于真实树高**，只是高度的上界。这不影响正确性——rank 只在 union 时用于决定方向，而 rank 的单调递增关系仍然成立。

---

### 2.6 两种优化结合——摊还 O(α(n))

| 策略组合 | Find 代价 | 备注 |
|---|---|---|
| 只用 Union by Rank | O(log n) | 树高有界，但没摊平 |
| 只用路径压缩 | 摊还 O(log n) | 摊平了，但树可能长歪 |
| **两者同时使用** | 摊还 **O(α(n))** | 工程首选 |

**什么是 α(n)——反阿克曼函数？**

先看 Ackermann 函数，它增长极其快：

```
A(1, x) = x + 1
A(2, x) = 2x
A(3, x) = x^(2x)           ← 指数级
A(4, x) = 2^(2^(2^...x层)) ← 2 的 x 层塔
```

具体数字：
- A(3) = 2048
- A(4) ≥ 一个由 2048 个 2 堆成的幂塔 >> 宇宙中所有亚原子粒子的数量

**α(n) = "需要多少层才能让 A 超过 n"**——因为 Ackermann 函数涨得极快，它的反函数 α(n) 涨得极慢：

> **对任何实际的 n，α(n) ≤ 4**。

所以摊还 O(α(n)) 在所有实际场景下等价于 **O(1)**。

**总复杂度定理**：m 次 union/find 操作，从 n 个单元素集合出发，总时间是 **O((n + m) · α(n))**。由于 α(n) ≤ 4，这就是线性的 O(n + m)。

**摊还分析的证明思路（了解即可）**：
- 对每个节点 v 定义 rank：r(v) = ⌊log n(v)⌋ + 2，其中 n(v) 是子树大小
- 秩为 s 的节点最多有 n/2^(s-2) 个（秩大 → 子树大 → 总数受限）
- 沿父链追踪时，秩严格单调递增
- 用"虚拟货币"法：把追踪代价分配给节点或 find 操作，证明每次 find 摊还代价是 O(α(n))

---

### 2.7 连通分量算法（Union-Find 最经典应用）

**问题**：给 n 个人（元素）、m 条"认识"关系（边），求所有连通分量——即"哪些人彼此（直接或间接）相连"。

**应用背景**：社交网络研究中，连通分量就是"社群"——教授举例：美国独立战争时期的人际网络，可以清晰分出不同派系的社群。连通分量内的任意两人，必须有直接或间接的朋友链。

**算法**：

```
输入：n 个元素（集合 S），m 条边（集合 E）
输出：每个元素 x 的连通分量代表 find(x)

Phase 1 — 初始化：
    for each x in S:
        makeSet(x)          # 每个人先是自己一个独立社群

Phase 2 — 处理关系：
    for each edge (x, y) in E:
        if find(x) ≠ find(y):      # 两人还不在同一社群
            union(x, y)             # 合并（建立连通）

Phase 3 — 查询：
    find(x) == find(y)  →  x 和 y 在同一连通分量
    数 find(x) == x 的节点个数  →  连通分量总数
```

**时间复杂度**：O(t · (n + m))，其中 t = 单次 union/find 代价
- 用两种优化：t = O(α(n))，整体 **O((n + m) · α(n)) ≈ O(n + m)**

**为什么这个算法重要？** 它给 Kruskal 最小生成树算法提供了核心的"判断是否成环"子程序（见 M13）。

---

## 3. 实际应用

### 3.1 迷宫生成（Maze Generation）

**问题**：给一个 n×n 方格，初始状态所有格子间的墙全部封闭。要生成一个"从左上角到右下角有且仅有一条路"的合法迷宫。

**关键约束**：
- 去掉的墙太少 → 没有路
- 去掉的墙太多 → 多条路（迷宫太简单）
- **精确地去掉 n²-1 堵墙** → 所有格子连通，且无环（= 唯一路径）

**为什么是 n²-1？** n² 个格子连通且无环 = 生成树，生成树恰好有 n²-1 条边。

**算法（MazeGen）**：

```
初始化：
    n² 个格子，每个 makeSet
    所有相邻格子间的墙放入集合 E

while |E| > 0:
    随机从 E 中取一堵墙 (x, y)
    E.remove(x, y)
    if find(x) ≠ find(y):        # 两格还不连通
        union(x, y)               # 打通这堵墙，建立连通
        R.add(x, y)               # 记录被拆除的墙
    # 否则跳过：打通会产生环（第二条路径），不符合迷宫定义

输出：被拆除的墙集合 R（即迷宫结构）
```

**结果**：恰好拆除 n²-1 堵墙，所有格子连通，无环。

**时间复杂度**：O(t · (n² + m))，m = 墙的总数 ≈ 2n(n-1)

**深层联系**：迷宫生成本质是在图上随机构造**生成树**——与 M13 的 Kruskal 算法思路完全相同（都是判断加边是否成环，只是 Kruskal 按权重排序选边，迷宫随机选）。

---

### 3.2 渗流理论（Percolation Theory）

**背景**：多孔材料（石头、土壤）的物理性质——水从顶部注入，能否渗透到底部？

**模型**：把石头建模为 n×n×n 的三维格子。每对相邻格子之间的隔板，以概率 p 被移除（保留概率 1-p）。

**Union-Find 的作用**：

```
for each pair (x, y) of adjacent cells:
    以概率 p：union(x, y)      # 这两格之间的隔板被移除

if find(顶层某格) == find(底层某格):
    水能渗透（percolate）✓
```

**为什么这有趣？** 存在一个**临界概率 p\***：当 p 低于 p\* 时，系统几乎肯定不导通；超过 p\* 时，几乎肯定导通。这种突变叫**相变（phase transition）**，Union-Find 是研究这类现象的核心计算工具。

**与迷宫的类比**：

| | 迷宫生成 | 渗流理论 |
|---|---|---|
| 格子 | n×n（二维）| n×n×n（三维）|
| 触发 | 随机选墙拆除 | 以概率 p 移除隔板 |
| 问题 | 入口到出口是否连通 | 顶部到底部是否连通 |
| 本质 | 构造生成树 | 模拟相变 |

---

## 4. 策略对比总表

| 策略 | Find（单次）| Union（单次）| n 次操作总摊还 |
|---|---|---|---|
| List-based（随机 union）| O(1) | O(n) 最坏 | O(n²) |
| List-based（union by size）| **O(1)** | O(min size) 最坏 | **O(n log n + m)** |
| Tree Quick-Union（裸）| O(n) 最坏 | O(n) | O(n²) |
| + Union by Rank | O(log n) | O(log n) | O((n+m) log n) |
| + Path Compression only | 摊还 O(log n) | — | — |
| **两者结合** | **O(α(n)) ≈ O(1)** | **O(α(n))** | **O((n+m)·α(n))** |

---

## 5. 常考陷阱

- ❌ **find 不是查"元素在不在"**——是查"元素属于哪个集合"，两者有本质区别
- ❌ **Java Set / Python set 不等于 Union-Find**——哈希实现的集合只管成员存在性，没有 union/find 语义
- ❌ **路径压缩后 rank 不再等于真实高度**——它只是上界，但算法正确性不受影响
- ❌ **union 操作必须先 find 到根**再合并根，不能直接挂元素到元素
- ❌ **单独路径压缩 ≠ O(α(n))**——必须搭配 union by rank/size 才能达到最优
- ❌ **`union(x, x)` 是空操作**——find(x) == find(x) 恒成立，直接返回
- ❌ **list-based 实现的 find 是 O(1)**——不要和 tree-based 混淆；list-based 的代价集中在 union

---

## 6. 面试高频题

1. **朋友圈 / 连通分量数** —— 套用连通分量算法，最后数 `find(x) == x` 的根节点个数
2. **Kruskal 最小生成树** —— 加每条边前：`if find(x) != find(y): union(x, y)` 防止成环
3. **冗余连接（Redundant Connection）** —— 第一条 `find(x) == find(y)` 的边就是答案
4. **账户合并（Accounts Merge）** —— 把共享邮箱的账户 union 在一起，用 find 归组
5. **岛屿数量 / 迷宫问题** —— 本质都是"动态连通性"，union-find 比 BFS/DFS 更适合动态更新

---

## 7. 三视角职业分析

- **工程师视角**：竞赛和面试中性价比极高的"模板"工具；几十行代码解决一类图连通问题；DSU（Disjoint Set Union）是 Competitive Programming 标准组件
- **研究员视角**：α(n) 的摊还证明是 Tarjan（1975）的经典贡献，开创了用势函数分析摊还复杂度的先河；渗流理论是复杂系统**相变**研究的核心模型，Union-Find 是其主要计算工具
- **系统架构师视角**：分布式系统的**网络连通性检测**（节点是否在同一集群）；社交平台的**社区发现**（谁和谁在同一朋友圈）；Kruskal 用于网络基础设施最优连通设计

---

## 8. 术语速查表

| 英文 | 中文 |
|---|---|
| Disjoint Set / Union-Find / DSU | 不相交集合 / 并查集 |
| Partition | 划分（Union-Find 的旧称）|
| makeSet / find / union | 建集 / 查归属 / 合并 |
| Representative / Root | 代表元 / 根 |
| Membership | 归属关系（Union-Find 的核心语义）|
| Union by Rank | 按秩合并 |
| Union by Size | 按大小合并 |
| Rank | 秩（高度的上界估计，不一定是真实高度）|
| Path Compression | 路径压缩 |
| Amortized cost | 摊还代价 |
| Ackermann function A(x) | 阿克曼函数（极速增长）|
| Inverse Ackermann α(n) | 反阿克曼函数（≤ 4 for all practical n）|
| Connected Component | 连通分量 |
| Spanning Tree | 生成树（迷宫生成的本质）|
| Percolation | 渗流 |
| Phase transition | 相变（渗流理论的核心现象）|

---

## 9. 关联模块

- 前置 → [[Module 1: Algorithm Analysis]]（摊还分析势函数法）
- 相似结构 → [[Module 2: Basic Data Structures]]（树、链表基础）
- 直接应用 → [[Module 13: Minimum Spanning Trees]]（Kruskal 用 find 判环，是 Union-Find 最经典的图算法应用）
- 同样处理连通性 → [[Module 11: Graphs and Traversals]]（DFS/BFS 是静态连通性分析；Union-Find 是**动态**连通性维护）
- 迷宫生成 = 随机生成树 → 对比 [[Module 13: Minimum Spanning Trees]]（Kruskal = 按权重选边的生成树）
