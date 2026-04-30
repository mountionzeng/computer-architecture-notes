# CS 590 算法设计与应用 · 学习控制台

> Stevens Institute of Technology · Instructor: Reza Peyrovian
> 教材：Goodrich & Tamassia, *Algorithm Design and Applications* (Wiley)

---

## 进度追踪

> 把 `[ ]` 改成 `[x]` 标记完成。三列：读完 → 能复述 → 能做题

| 模块 | 读完 | 能复述 | 能做题 |
|---|:---:|:---:|:---:|
| M1 Algorithm Analysis | [ ] | [ ] | [ ] |
| M2 Basic Data Structures | [ ] | [ ] | [ ] |
| M3 Binary Search Trees | [ ] | [ ] | [ ] |
| M3.1 Balanced BSTs (补充) | [ ] | [ ] | [ ] |
| M4 Priority Queues & Heaps | [ ] | [ ] | [ ] |
| M5 Hash Tables | [ ] | [ ] | [ ] |
| M6 Union-Find | [ ] | [ ] | [ ] |
| M7 Sorting & Selection | [ ] | [ ] | [ ] |
| M8 Greedy Method | [ ] | [ ] | [ ] |
| M9 Divide & Conquer | [ ] | [ ] | [ ] |
| M10 Dynamic Programming | [ ] | [ ] | [ ] |
| M11 Graphs & Traversals | [ ] | [ ] | [ ] |
| M12 Shortest Paths | [ ] | [ ] | [ ] |
| M13 Minimum Spanning Trees | [ ] | [ ] | [ ] |

---

## 模块依赖图

```
M1 分析工具
    │
    ├─→ M2 基础数据结构
    │       │
    │       ├─→ M3 BST ──→ M3.1 平衡BST
    │       │       │
    │       ├─→ M4 Heap ──→ M7 排序（HeapSort）
    │       │
    │       ├─→ M5 Hash
    │       │
    │       └─→ M6 Union-Find ──→ M13 MST（Kruskal）
    │
    ├─→ M7 排序 & 选择（需要 M2 + M4）
    │
    ├─→ M8 贪心 ──→ M13 MST（Prim）
    │
    ├─→ M9 分治 ──→ M7 排序（MergeSort/QuickSort）
    │
    ├─→ M10 动态规划（需要 M9 思维）
    │
    └─→ M11 图遍历 ──→ M12 最短路 ──→ M13 MST
```

**关键前置规则**：
- 不懂 M1 → 所有复杂度分析都是猜的
- 不懂 M3 → M4/M5/M6 学起来很吃力
- M11 是 M12/M13 的硬前置，别跳

---

## 核心复杂度速查表

### 数据结构

| 结构 | 查找 | 插入 | 删除 | 范围查询 | 有序？|
|---|---|---|---|---|---|
| 数组（无序）| O(n) | O(1)尾 | O(n) | — | ✗ |
| 数组（有序）| O(log n) | O(n) | O(n) | O(log n+s) | ✓ |
| 链表 | O(n) | O(1)* | O(1)* | — | ✗ |
| 平衡 BST | O(log n) | O(log n) | O(log n) | O(log n+s) | ✓ |
| 哈希表 | O(1)均摊 | O(1)均摊 | O(1)均摊 | **不支持** | ✗ |
| 堆（Heap）| O(n) | O(log n) | O(log n) | — | 仅最值 |
| Union-Find | — | O(α n)≈O(1) | — | — | ✗ |

*链表操作本身 O(1)，但找位置 O(n)

### 排序算法

| 算法 | 时间（平均） | 时间（最坏） | 空间 | 稳定？| 关键思路 |
|---|---|---|---|---|---|
| MergeSort | O(n log n) | O(n log n) | O(n) | ✓ | 分治，合并 |
| QuickSort | O(n log n) | O(n²) | O(log n) | ✗ | 分治，partition |
| HeapSort | O(n log n) | O(n log n) | O(1) | ✗ | 堆 |
| BucketSort | O(n+k) | O(n²) | O(n+k) | ✓ | 非比较，分桶 |
| RadixSort | O(d·n) | O(d·n) | O(n+k) | ✓ | 非比较，按位 |
| 比较排序下界 | — | **Ω(n log n)** | — | — | 决策树高度 |

### 图算法

| 算法 | 时间 | 适用场景 |
|---|---|---|
| BFS | O(V+E) | 无权最短路、层序 |
| DFS | O(V+E) | 拓扑排序、强连通分量 |
| Dijkstra（堆）| O((V+E) log V) | 非负权最短路 |
| Bellman-Ford | O(V·E) | 含负权、检测负环 |
| Floyd-Warshall | O(V³) | 全源最短路 |
| Kruskal | O(E log E) | MST，稀疏图 |
| Prim-Jarnik | O((V+E) log V) | MST，稠密图 |

---

## 问题 → 范式决策树

```
你面对一个新问题
        │
        ▼
问题能分解成"局部最优 = 全局最优"吗？
        │
       YES → 试 Greedy（M8）
        │    例：Huffman编码、区间调度、Kruskal
        │
        NO
        │
        ▼
问题有"最优子结构 + 重叠子问题"吗？
        │
       YES → 试 DP（M10）
        │    例：LCS、0-1背包、编辑距离
        │
        NO
        │
        ▼
问题能被切成独立子问题再合并吗？
        │
       YES → 试 Divide & Conquer（M9）
        │    例：MergeSort、二分查找、Strassen
        │
        NO
        │
        ▼
问题涉及图上的路径/连通/生成树？
        │
       YES → 图算法（M11-M13）
        │
        NO → 考虑选对数据结构（M2-M6）
```

---

## "选哪个数据结构"速查

| 我需要… | 选这个 |
|---|---|
| 按 key 快速查找，不需要有序 | **哈希表**（M5）|
| 有序 + 动态插删 + 范围查询 | **平衡 BST**（M3.1）|
| 快速拿最大/最小值 | **堆**（M4）|
| 判断两个元素是否同属一组 | **Union-Find**（M6）|
| 先进先出 / 后进先出 | **队列 / 栈**（M2）|
| 中间频繁插删 | **链表**（M2）|

---

## 三大范式核心对比

| | Greedy（M8）| Divide & Conquer（M9）| DP（M10）|
|---|---|---|---|
| 决策方式 | 每步局部最优，不回头 | 拆成独立子问题，递归解 | 记录子问题结果，避免重复 |
| 子问题关系 | 无重叠 | 无重叠 | **有重叠**（核心区别）|
| 正确性保证 | 贪心选择性质 + 最优子结构 | 分治正确性 | 最优子结构 |
| 典型例子 | Huffman、Kruskal | MergeSort、快速幂 | LCS、0-1背包 |
| 时间复杂度 | 通常 O(n log n) | T(n)=2T(n/2)+O(n) | 状态数 × 转移代价 |

---

## 跨模块高频连接

- **中序遍历 = BST 有序性**（M2 遍历 → M3 BST 核心）
- **Heap 就是一棵完全二叉树**（M2 树 → M4 Heap）
- **DFS 回溯 = 函数调用栈**（M2 栈 → M11 图遍历）
- **Kruskal 用 Union-Find 判环**（M6 → M13）
- **Master Theorem 解分治递归式**（M1 → M9）
- **DP 本质是 DAG 上的最短路**（M10 → M12）
- **Hash 用于图的邻接表存储**（M5 → M11）

---

## 周次 → 模块（考试节点标注）

| 周 | 模块 | 主题 | 节点 |
|---|---|---|---|
| W1 | [[Module 1: Algorithm Analysis]] | 大O、Master Theorem、摊还 | — |
| W2 | [[Module 2: Basic Data Structures]] | 栈、队列、链表、树、遍历 | — |
| W3 | [[Module 3: Binary Search Trees]] | BST + 平衡 BST | — |
| W4 | [[Module 4: Priority Queues and Heaps]] | Heap、HeapSort、PQ | — |
| W5 | [[Module 5: Hash Tables]] | 哈希函数、冲突、Cuckoo | — |
| W6 | [[Module 6: Union-Find Structures]] | 并查集、路径压缩、按秩合并 | — |
| W7 | [[Module 7: Sorting and Selection]] | 排序全家桶、比较下界、线性选择 | **Midterm** |
| W8 | [[Module 8: The Greedy Method]] | 贪心、Huffman | — |
| W9 | [[Module 9: Divide and Conquer]] | 递归式、分治范式 | — |
| W10 | [[Module 10: Dynamic Programming]] | LCS、背包、博弈 | — |
| W11 | [[Module 11: Graphs and Traversals]] | DFS、BFS、拓扑、SCC | — |
| W12 | [[Module 12: Shortest Paths]] | Dijkstra、BF、Floyd | — |
| W13 | [[Module 13: Minimum Spanning Trees]] | Kruskal、Prim、Borůvka | **Final** |

---

## 关联资源

- [[CS570↔CS590 联动地图]] — 两门课知识点对应关系
- [[内存联动地图]] — 内存/指针/缓存在各门课的视角
- 可视化工具：visualgo.net · algorithm-visualizer.org
- CLRS 对照（进阶）：*Introduction to Algorithms*（Cormen et al.）
