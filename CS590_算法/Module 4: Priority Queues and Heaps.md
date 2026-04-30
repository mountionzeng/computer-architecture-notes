
> **教材章节**：Ch 5 · **对应周次**：Week 4
> **核心问题**：只关心"最大/最小值"，不关心全序，该用什么结构？

---

## 0. 知识地图

```
            Priority Queue（抽象接口）
                  │
        ┌─────────┼─────────┐
     实现方式   操作语义    应用场景
        │          │         │
   无序数组    insert      Heap-Sort
   有序数组    removeMin   Selection-Sort
     Heap     peek-min    Insertion-Sort
       │      decrease-key Dijkstra/Prim
   数组表示               Huffman
                           Top-K
```

---

## 1. 思考脉络

1. **现实需求**：不是"把所有任务排好序"，而是"每次只取最紧急的那个"
   - 医院急诊室：优先处理最重的病人，而不是等待时间最长的
   - 机场候补：有头等舱会员、常旅客、普通旅客三种级别
   - 股票撮合系统：买方出价最高的和卖方要价最低的优先成交——**同时维护两个 PQ**
2. 用有序数组？插入 O(n) 太慢
3. 用 BST？功能过剩（不需要全序），还要维护平衡
4. → **堆（Heap）**：只维护"局部序"（父 ≤ 子），insert/removeMin 都 O(log n)
5. → 堆还能就地排序 → **Heap-Sort**，O(n log n)，O(1) 额外空间

---

## 2. 核心知识点

### 2.1 优先队列的语义基础——全序关系

Priority Queue 能比较"谁更优先"，依赖底层的**全序关系（Total Order Relation）**：

| 性质 | 定义 |
|---|---|
| 自反性（Reflexive） | k ≤ k |
| 反对称性（Anti-symmetric） | k₁ ≤ k₂ 且 k₂ ≤ k₁ → k₁ = k₂ |
| 传递性（Transitive） | k₁ ≤ k₂ 且 k₂ ≤ k₃ → k₁ ≤ k₃ |

PQ 存储的是 **(key, value)** 对——key 用来比较优先级，value 是实际数据。

---

### 2.2 Priority Queue 接口

**必需方法（ADT 核心）**：
- `insert(k, v)` — 插入优先级为 k 的元素 v
- `removeMin()` — 删除并返回优先级最低（最高优先）的元素

**附加方法**：
- `min()` — 查看最小值但不删除（O(1) with heap）
- `size()` / `isEmpty()`

> **教授强调**：insert 和 removeMin 是 PQ 的两个核心方法。min() 是锦上添花的查看，不是必需的。

---

### 2.3 用 PQ 构造排序——三种排序的统一视角

**PQ-Sort 框架**：

```
阶段一：把 n 个元素全部 insert 进 PQ
阶段二：反复 removeMin，按序取出所有元素
```

底层 PQ 实现的不同，产生了三种不同的排序算法：

| 排序 | PQ 实现 | 插入代价 | 取出代价 | 总复杂度 |
|---|---|---|---|---|
| Selection-Sort | **无序**数组 | O(1)×n | O(n)×n | O(n²) |
| Insertion-Sort | **有序**数组 | O(n)×n | O(1)×n | O(n²) |
| **Heap-Sort** | **堆** | O(log n)×n | O(log n)×n | **O(n log n)** |

**关键洞察**：Selection-Sort 慢是因为每次"取最小"都扫全场；Insertion-Sort 慢是因为每次"插入"都要搬元素保持有序。堆两边都是 log n，没有明显短板。

**原地排序（In-place）的动机**：
- 如果把元素 copy 进新数组再排序，需要 O(n) 额外空间——对大数据集这很昂贵
- Heap-Sort 直接在原数组上操作，**O(1) 额外空间**
- 这是 Heap-Sort 优于 Merge-Sort（O(n) 额外空间）的核心优势

---

### 2.4 什么是堆——先搞清楚定义

> **教授原话**："Heap is not a data structure. Heap is a tree that has certain properties."

堆是一种有特定性质的树，不是某种独立的数据结构类别。Binary Heap（二叉堆）就是满足两条性质的**完全二叉树**：

**性质 1：形状——完全二叉树（Complete Binary Tree）**
- 所有层全部填满，除了最后一层
- 最后一层从**左向右**填充，不能有空洞

![[Pasted image 20260429112311.png]]

为什么必须是完全二叉树？因为：
- 保证树高是 O(log n)：n 个节点的完全二叉树，`n ≥ 2^h`，所以 `h ≤ log n`
- 可以用**数组紧凑存储**，无需指针（见 2.5）

**性质 2：堆序（Heap-Order Property）**
- 最小堆：每个父节点的 key ≤ 两个子节点的 key
- 即：**根节点是整棵树中 key 最小的元素**
- 这只是局部约束（父 ≤ 子），不要求兄弟节点之间有序

> **堆 vs BST**：BST 是全局有序（左子树全 < 根 < 右子树全），中序遍历有序；堆只有局部的父子关系，**中序遍历无序**。

---

### 2.5 堆的数组存储——"双重身份"

同一个堆，**既是树又是数组**，两种视角无缝切换：

![[Pasted image 20260429112546.png]]

**索引规则**（根在索引 1）：

| 关系 | 公式 |
|---|---|
| parent(i) | i / 2（整除）|
| left child(i) | 2i |
| right child(i) | 2i + 1 |
| 最后一个非叶节点 | n / 2 |

**为什么数组存储比指针存储好？**
1. **零指针开销**：不需要存 left/right/parent 指针，节省 3×8 bytes 每节点
2. **缓存友好**：数组连续存储，CPU 预取缓存行一次读入多个节点；指针树节点分散在堆内存，缓存命中率低
3. **下标计算快**：位运算 `i<<1`、`i>>1` 比指针跳转快

---

### 2.6 关键操作——"修复"是唯一动作

堆的所有动态操作，本质上都是**同一个动作的变种：修复局部破坏的堆序**。

理解了"修复"，所有操作迎刃而解。

#### Insert（插入）→ 上浮（Up-heap / Bubble-up）

```
1. 把新元素挂到数组末尾（完全二叉树的下一个位置）
2. 比较新元素和父节点：如果新元素 < 父，swap
3. 重复第 2 步，直到 新元素 ≥ 父，或到达根
```

**为什么这样对？** 末尾插入不破坏形状；上浮只修复从插入点到根的路径，其余节点未受影响。最多走 log n 步（树高），所以 **O(log n)**。

![[heap_insert.png]]

> 💡 想点按钮逐步看交换过程？👉 [点这里在浏览器打开](heap_insert_up_heap.html)

**注意**：上浮不一定走到根——只要遇到父节点 ≤ 自己就停下。上图是恰好插入了最小值，所以一路浮到顶。

#### RemoveMin（删除最小值）→ 下沉（Down-heap / Trickle-down）

```
1. 记录根节点（即最小值），准备返回
2. 把数组最后一个元素移到根位置
3. 数组大小 -1（删掉末尾）
4. 新根和两个孩子比较：如果新根 > min(左孩子, 右孩子)，和较小的孩子 swap
5. 重复第 4 步，直到 新根 ≤ 两个孩子，或到达叶节点
```

**为什么要把末尾移到根？** 直接删根会留空洞——形状被破坏。末尾元素填进根，形状保持（完全二叉树），然后向下修复堆序。同样最多走 log n 步，**O(log n)**。

#### Peek-Min

直接返回根节点，**O(1)**。这是堆相比 BST 的主要优势之一——BST 找最小还要走到最左叶节点 O(log n)。

---

### 2.7 操作复杂度速查

| 操作 | 思路 | 复杂度 |
|---|---|---|
| insert | 末尾插入 → **上浮（up-heap）** | O(log n) |
| removeMin | 末尾替换根 → **下沉（down-heap）** | O(log n) |
| min（peek） | 返回根 | **O(1)** |
| buildHeap | 从底往上 heapify | **O(n)** |

---

### 2.8 反直觉的 O(n) buildHeap

**天真做法**（一个个 insert）：n 次 insert，每次 O(log n) → **O(n log n)**

**buildHeap 的聪明做法**：

```
1. 把 n 个元素直接放进数组（不管堆序）
2. 从最后一个非叶节点（索引 n/2）开始，往根方向，对每个节点做 down-heap
3. 叶节点不需要处理（叶节点没有孩子，天然满足）
```

![[Pasted image 20260429114109.png]]

**为什么是 O(n) 而不是 O(n log n)？**

关键是各层节点数 × 该层节点下沉距离的总和：
- 底层节点最多（约 n/2 个），但它们最多只下沉 1 步
- 上层节点少（约 n/4 个），但下沉距离最多 2 步
- ……
- 根只有 1 个，最多下沉 log n 步

严格求和：`Σ (节点数 × 下沉步数) = O(n)`

**实际意义**：如果你一次性有 n 个数要建堆，用 buildHeap 比一个个 insert 快 log n 倍。

---

### 2.9 Heap-Sort

利用最大堆原地排序：

```
1. 用 buildHeap 把数组建成最大堆     → O(n)
2. 循环 n-1 次：
   - 交换根（当前最大值）和末尾元素
   - 堆大小 -1
   - 对新根做 down-heap               → O(log n) 每次
3. 最终数组从小到大有序
```

**总计 O(n log n)，原地（O(1) 额外空间），不稳定**

> 为什么用最大堆而不是最小堆？最大堆的根是当前最大值，每次交换到末尾就是"已排好的最大值沉底"，最终数组升序。用最小堆反而要每次把最小值放到前面，逻辑一样只是顺序相反。

---

### 2.10 Top-K 问题

**场景**：给你 10 亿个数，找出最大的 100 个。不能直接排序——内存爆炸。

![[Pasted image 20260429114555.png]]

**解法**：维护大小为 K 的**最小堆**

```
1. 把前 K 个数 buildHeap → 大小为 K 的最小堆
2. 对剩余 n-K 个数，每个新数 x：
   - 如果 x > 堆顶（堆顶 = 当前 K 个里最小的）：
     - 弹出堆顶，插入 x
   - 否则跳过
3. 最终堆里剩的就是最大的 K 个
```

**时间复杂度**：O(n log K)；**空间复杂度**：O(K)

**精妙之处**——用最小堆找最大值，很反直觉。理解的关键：
- 堆的根在 Top-K 里**不是"答案"，而是"门槛"**
- 我们维护的是一个大小为 K 的"VIP 名单"，门槛 = 名单里最弱的
- 新人要进 VIP 必须超过门槛，进来后顶替门槛
- 这种"持续过滤"是堆最核心的应用模式之一

---

### 2.11 PQ 扩展操作

- **Decrease-Key(v, k)**：把节点 v 的 key 降低到 k，然后上浮修复
  - 为什么重要：Dijkstra / Prim 算法的核心操作
  - 挑战：必须知道节点在堆中的位置 → 需要维护**索引映射**（element → heap position）
  
- **Merge 两个堆**：
  - 二叉堆：O(n)（重新 buildHeap）
  - Leftist Heap / Binomial Heap：O(log n)
  - Fibonacci Heap：摊还 O(1) decrease-key → Dijkstra 理论最优 O((V+E) log V)

---

## 3. 复杂度对比

| 实现 | insert | removeMin | min | buildHeap |
|---|---|---|---|---|
| 无序数组 | O(1) | O(n) | O(n) | O(n) |
| 有序数组 | O(n) | O(1) | O(1) | O(n log n) |
| **二叉堆** | O(log n) | O(log n) | **O(1)** | **O(n)** |
| Fibonacci Heap | O(1) 摊还 | O(log n) 摊还 | O(1) | O(n) |

**为什么二叉堆是"最佳综合"？**
- 无序数组：insert O(1) 但 removeMin O(n)——有短板
- 有序数组：removeMin O(1) 但 insert O(n)——有短板
- BST：log n 但维护了不需要的全序信息
- **堆：所有常用操作都是 log n，没有 O(n) 的短板。精确地做了刚好够用的工作，不多不少**

---

## 4. 常考陷阱

- ❌ **堆不是 BST**——中序遍历**不**有序；兄弟节点之间无大小关系
- ❌ `buildHeap` 从 **n/2 向下**开始（叶子无需 heapify），不是从 1 到 n
- ❌ Heap-Sort **不稳定**；需要稳定排序用 Merge-Sort
- ❌ decrease-key 必须知道节点在堆中的位置 → 需要额外维护**索引映射**
- ❌ **堆不是独立的数据结构**，它是一棵有特定性质的完全二叉树
- ❌ buildHeap 从底往上，不是从顶往下——方向搞反了结论就错了

---

## 5. 面试高频题

1. **Top-K 问题** —— 维护大小为 K 的最小堆，O(n log K)；为什么用最小堆找最大？门槛思维
2. **合并 k 个有序链表** —— k 路归并 + 最小堆，O(N log k)，每次从堆里取最小头节点
3. **数据流的中位数** —— 大顶堆（较小一半）+ 小顶堆（较大一半），维持两堆大小差 ≤ 1
4. **Heap-Sort vs Quick-Sort** —— Heap 最坏 O(n log n)；Quick 平均更快（常数小、缓存友好）但最坏 O(n²)
5. **手画 insert 和 removeMin** —— 能追踪上浮/下沉的每一步交换

---

## 6. 三视角职业分析

- **工程师视角**：`std::priority_queue`（C++）、`heapq`（Python）、`PriorityQueue`（Java）的底层实现；Top-K 和定时任务是最高频场景
- **研究员视角**：Fibonacci Heap 是 Dijkstra 理论最优复杂度的关键；摊还分析工具（势函数法）在此有经典应用
- **系统架构师视角**：操作系统进程调度（Linux CFS）、定时任务队列（Kafka 延迟消息）、股票撮合引擎——工业中无处不在

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Priority Queue | 优先队列 |
| Total Order Relation | 全序关系 |
| Heap | 堆 |
| Min-Heap / Max-Heap | 最小堆 / 最大堆 |
| Complete Binary Tree | 完全二叉树 |
| Heap-Order Property | 堆序性质 |
| Up-heap / Bubble-up | 上浮 |
| Down-heap / Trickle-down | 下沉 |
| Heapify | 堆化 |
| buildHeap | 建堆 |
| In-place | 原地 |
| Decrease-Key | 减小键值 |

---

## 8. 关联模块

- 前置 → [[Module 2: Basic Data Structures]]（树的概念）、[[Module 3: Binary Search Trees]]（对比全序 vs 局部序）
- 排序应用 → [[Module 7: Sorting and Selection]]（Heap-Sort vs Merge-Sort vs Quick-Sort）
- 图算法使用 → [[Module 12: Shortest Paths]]（Dijkstra 用 decrease-key）、[[Module 13: Minimum Spanning Trees]]（Prim 同样需要 PQ）
- 贪心工具 → [[Module 8: The Greedy Method]]（Huffman 编码用最小堆合并两个最小权值树）
