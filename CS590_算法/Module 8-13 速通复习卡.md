# Module 8-13 速通复习卡

> 如果你觉得这份还是太快、太密,先看:
> [[Module 8-13 前置理解版]]
>
> 目标：**看到题 5 秒判断章节，30 秒默写骨架，1 分钟说出复杂度的为什么。**
>
> 用法：
> - 考前最后一晚只读这一份
> - 做题前先在脑里跑一遍这里的"决策骨架"
> - 跟 AI 对代码时，对照"不变量"判断它有没有走偏

---

## 一张总图

| Module | 触发问题（看到这种字眼就先想到它） | 关键词 | 一句话钩子 |
|---|---|---|---|
| **8 Greedy** | "每一步选当前最好" / "不回头" / "可切开 / 可比例" | local best | 能切开 → 多半 Greedy |
| **9 D&C** | "拆成两半独立做" / "递归 + 合并" | split + recurse + combine | 子问题**独立** |
| **10 DP** | "子问题重叠" / "暴力指数爆炸" / "求最优 + 不能贪心" | memo / table | 子问题**重复** |
| **11 Graph** | "图遍历" / "判环" / "拓扑" / "无权最短路" | BFS / DFS | 无权用 BFS，结构性问题用 DFS |
| **12 Shortest Paths** | "带权图最短路" / "可能负权" | relax | 一切围绕**松弛** |
| **13 MST** | "把所有点连起来 + 总成本最低" | connect all cheaply | 不是最短路，是最便宜的连通子图 |

---

# Module 8 · Greedy

## 触发器
- 局部最优 → 全局最优
- 选了**不回头**
- 物品/资源**可切分** → 强烈贪心信号
- 0-1 选 / 不能切分 → 警惕，多半要 DP

## 成立条件（必须同时满足）
1. **Greedy choice property**：局部最优能导向全局最优
2. **Optimal substructure**：全局最优解包含子问题最优解

只有 (2) → DP；都没有 → 回溯/暴力。

## 经典算法骨架

### 活动选择（Activity Selection）

```python
def activity_selection(acts):
    # acts: list of (start, finish)
    acts.sort(key=lambda a: a[1])    # ① 按 finish 升序
    chosen, last_end = [], -float('inf')
    for s, f in acts:
        if s >= last_end:             # ② 不冲突就选
            chosen.append((s, f))
            last_end = f              # ③ 更新尾部边界
    return chosen
```

**🔍 代码识别特征**
- 看到 `sort(key=...finish/end...)` + 单层 for + 一个`last_end`变量 → 99% 是活动选择
- 没有 PQ、没有 DP 表、没有递归
- 只看 `start ≥ last_end` 一个条件

**关键行解读**
- ① **必须按 finish 排序，不是 start**：按 start 排序会出错（反例：长活动开始最早会霸占资源）
- ② `>=` 还是 `>`：取决于"刚好接上"算不算冲突——题面会说

**不变量**：在所有"已选活动结束时间最早"的方案中，存在最优解。
**复杂度**：O(n log n)（瓶颈是排序）。

### Fractional Knapsack

```python
def fractional_knapsack(items, W):
    # items: list of (weight, value)
    items.sort(key=lambda x: x[1]/x[0], reverse=True)  # ① 按单位价值降序
    total_value, remaining = 0.0, W
    for w, v in items:
        if remaining == 0: break
        take = min(w, remaining)                        # ② 能拿多少拿多少
        total_value += take * (v / w)                   # ③ 按比例计价
        remaining -= take
    return total_value
```

**🔍 代码识别特征**
- 看到 `sort(key=v/w)` 降序 → Fractional Knapsack（不是 0-1）
- 关键字 `take * (v / w)` —— **按比例**取价值，0-1 没有这一步
- 没有 dp 表

**为什么贪心成立**：能切分 → 单位价值最高的部分一定该全拿。
**0-1 不能贪心反例**：W=50, 物品 (10,60),(20,100),(30,120)，单位价值 6,5,4。贪心拿前两个得 160，但最优是后两个得 220。

### Huffman 编码

```python
import heapq

def huffman(freqs):
    # freqs: dict {char: frequency}
    heap = [[f, ch] for ch, f in freqs.items()]
    heapq.heapify(heap)                     # ① 最小堆
    while len(heap) > 1:
        a = heapq.heappop(heap)             # ② 弹两个最小
        b = heapq.heappop(heap)
        merged = [a[0] + b[0], a, b]        # ③ 合并成新内部节点
        heapq.heappush(heap, merged)        # ④ 放回堆
    return heap[0]                          # 树根
```

**🔍 代码识别特征**
- `heapq` + while 循环弹**两个**最小（不是一个）→ Huffman
- 弹两次再压一次 → 堆大小每轮 -1，O(n log n) 来源
- 内部节点的"频次 = 子频次之和"是经典签名

**复杂度**：O(n log n)。
**为什么贪心成立**（交换论证）：最低频的两个字符在最优编码树中一定是兄弟且在最深层。

## 手算 mini
活动 [(1,4),(3,5),(0,6),(5,7),(3,9),(5,9),(6,10),(8,11),(8,12),(2,14),(12,16)]
→ 按 finish 排序后选：(1,4) → (5,7) → (8,11) → (12,16)，共 4 个。

## 高频陷阱
- 0-1 背包用贪心 → 错
- 任意硬币面额用贪心 → 错（如 {1,3,4} 找 6，贪心给 4+1+1=3 枚，最优 3+3=2 枚）
- 没证明就用贪心 → **必须给出交换论证**

---

# Module 9 · Divide and Conquer

## 触发器
- 能拆成几个**独立**子问题
- 拆解后**规模显著缩小**（一般 n/2）
- 合并步骤可控

## 三步框架
```
divide_and_conquer(P):
    if P 足够小: 直接解
    把 P 切成 P1, P2, ..., Pk
    对每个 Pi 递归求解 → 得到 R1, ..., Rk
    合并 R1, ..., Rk 得到 P 的解
```

## Master Theorem 速查（必背）

T(n) = a · T(n/b) + f(n)

设 c* = log_b(a)：
| f(n) 与 n^c* 的关系 | 结论 |
|---|---|
| f(n) = O(n^c*-ε) | T(n) = Θ(n^c*) |
| f(n) = Θ(n^c* · log^k n) | T(n) = Θ(n^c* · log^(k+1) n) |
| f(n) = Ω(n^c*+ε) 且正则 | T(n) = Θ(f(n)) |

## 三个必记的递推

| 递推 | 解 | 例子 |
|---|---|---|
| T(n) = T(n/2) + O(1) | O(log n) | 二分查找 |
| T(n) = 2T(n/2) + O(n) | O(n log n) | 归并排序 |
| T(n) = 2T(n/2) + O(1) | O(n) | 树后序遍历 |
| T(n) = 7T(n/2) + O(n²) | O(n^2.81) | Strassen 矩阵乘 |
| T(n) = 2T(n/2) + O(n log n) | O(n log² n) | 最近点对（部分实现） |

## 经典算法
- **Merge Sort**：O(n log n)，稳定，需 O(n) 辅助空间
- **Quick Sort**：期望 O(n log n)，原地，不稳定
- **二分查找**：O(log n)
- **最近点对**：O(n log n)（合并步只看跨界 7 个点）
- **大整数乘法（Karatsuba）**：O(n^1.59)

## 手算 mini
T(n) = 3T(n/4) + n
c* = log_4(3) ≈ 0.79
f(n) = n = n^1，n^1 = Ω(n^0.79+ε) → Case 3 → T(n) = Θ(n)。

## 与 DP 的本质区别
- **D&C**：子问题**独立**（解一个不影响另一个）
- **DP**：子问题**重叠**（同一子问题被求多次 → 必须记忆化）

## 高频陷阱
- 合并步代价算错（最容易在 Master Case 1 vs 2 间犯错）
- "拆"的代价被忽略
- 子问题之间有依赖却用 D&C → 应该用 DP

---

# Module 10 · Dynamic Programming

## 触发器
- "求最优"且**不能贪心**
- 暴力递归出现重复子问题
- 状态空间是离散小整数（容量、长度、位置）

## DP 四要素（每题先把这 4 件事写出来）
1. **状态**：`dp[i]` / `dp[i][j]` 含义是什么（**最关键**——状态错则全错）
2. **转移**：`dp[i] = f(dp[<i])`
3. **边界**：dp[0] = ?
4. **顺序**：从小到大？倒序？对角线？

## 自顶向下 vs 自底向上
- **记忆化递归（top-down）**：好写，但有递归开销
- **递推填表（bottom-up）**：快，便于空间优化

## 必背模板

### 0-1 Knapsack

```python
def knapsack_01(items, W):
    # items: list of (weight, value); 二维版
    n = len(items)
    dp = [[0]*(W+1) for _ in range(n+1)]
    for i in range(1, n+1):
        wi, vi = items[i-1]
        for w in range(W+1):
            dp[i][w] = dp[i-1][w]                              # ① 不拿
            if w >= wi:
                dp[i][w] = max(dp[i][w], dp[i-1][w-wi] + vi)   # ② 拿
    return dp[n][W]

def knapsack_01_rolling(items, W):
    # 滚动数组：一维 dp[w]
    dp = [0]*(W+1)
    for wi, vi in items:
        for w in range(W, wi-1, -1):                # ③ 倒序！
            dp[w] = max(dp[w], dp[w-wi] + vi)
    return dp[W]
```

**🔍 代码识别特征**
- 二维 `dp[i][w]` + `max(不拿, 拿)` 二选一 → 0-1 背包
- 一维 + **`range(W, wi-1, -1)` 倒序** → 0-1 滚动版
- 一维 + **`range(wi, W+1)` 正序** → 完全背包（每物品可拿无限件）

**关键行解读**
- ① **必须先写"不拿"**（继承 dp[i-1][w]），保证 w<wi 的情况也对
- ③ **为什么倒序**：dp[w] 更新依赖 dp[w-wi] 的"上一行"值。若正序遍历，dp[w-wi] 已是本物品更新过的值 → 等于这个物品被取了两次（变成完全背包）

**完全背包反向利用了这个 bug**：正序遍历就是"每个物品可取多次"。

![[knapsack_dp_table.png]]

> 💡 想交互式逐步填表 / 看回溯过程？👉 [点这里在浏览器打开](knapsack_dp_table.html)

### LCS（最长公共子序列）

```python
def lcs(X, Y):
    m, n = len(X), len(Y)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(1, m+1):
        for j in range(1, n+1):
            if X[i-1] == Y[j-1]:                      # ① 字符相等
                dp[i][j] = dp[i-1][j-1] + 1           #    左上+1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])# ② 上方/左方取大
    return dp[m][n]
```

**🔍 代码识别特征**
- 二维 dp + **两个字符串作为两个维度** → LCS / 编辑距离 / 公共子串家族
- `X[i-1] == Y[j-1]` 等就 +1，否则 max 上左 → **LCS 专属签名**
- 注意 `dp[i-1][j-1]+1` 这一手——是子序列（不要求连续）的标志

**与"最长公共子串"的区别**：子串要求连续，相等时 +1，不等时**直接置 0**（不取 max）。

O(mn) 时空。

### 编辑距离（Levenshtein）

```python
def edit_distance(X, Y):
    m, n = len(X), len(Y)
    dp = [[0]*(n+1) for _ in range(m+1)]
    for i in range(m+1): dp[i][0] = i      # ① 边界：删 i 次
    for j in range(n+1): dp[0][j] = j      #    边界：插 j 次
    for i in range(1, m+1):
        for j in range(1, n+1):
            if X[i-1] == Y[j-1]:
                dp[i][j] = dp[i-1][j-1]     # ② 不动
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j-1],           # 替换
                    dp[i-1][j],             # 删除
                    dp[i][j-1])             # 插入
    return dp[m][n]
```

**🔍 代码识别特征**
- 二维 dp + **三个 min** → 编辑距离的招牌
- 边界 `dp[i][0]=i, dp[0][j]=j` 是另一个签名（LCS 边界全 0）
- 字符相等时 **`dp[i-1][j-1]`**（不是 +1，跟 LCS 区别开）

### LIS（最长递增子序列）
- O(n²) DP：`dp[i] = max(dp[j]+1) for j<i, a[j]<a[i]`
- O(n log n) 二分 + 贪心数组

### 矩阵链乘 / 区间 DP
```
dp[i][j] = 把 A_i..A_j 相乘的最少标量乘法
dp[i][j] = min over k in [i, j-1]: dp[i][k] + dp[k+1][j] + p_{i-1}·p_k·p_j
```
**填表顺序**：按区间长度从短到长。

## 手算 mini（LCS）
X = "ABCB", Y = "BDC"
```
       ""  B  D  C
   ""   0  0  0  0
   A    0  0  0  0
   B    0  1  1  1
   C    0  1  1  2
   B    0  1  1  2
```
LCS 长度 = 2（"BC"）。

## 复杂度直觉
- 状态数 × 每状态转移代价 = 总时间
- LCS：O(mn) 个状态，每个 O(1) 转移 → O(mn)

## 高频陷阱
- 状态没把"必要的所有维度"包进去（如背包忘了"剩余容量"维）
- 0-1 背包滚动数组**忘了倒序**
- 转移方程边界条件没写
- 把"必须连续"的子串问题用了"子序列"的 DP（或反之）

---

# Module 11 · Graphs and Traversals

## 触发器
- 任何"图"问题：节点、边、连通性、路径
- 关键词：拓扑、判环、遍历、连通分量、SCC

## 表示法选择
| | 邻接表 | 邻接矩阵 |
|---|---|---|
| 空间 | O(V+E) | O(V²) |
| 查边 (u,v) | O(deg(u)) | O(1) |
| 遍历邻居 | O(deg(u)) | O(V) |
| 适合 | 稀疏图（一般图） | 稠密图 / Floyd |

## BFS（广度优先）

```python
from collections import deque

def bfs(adj, s):
    dist = {s: 0}
    parent = {s: None}
    q = deque([s])                      # ① 队列（FIFO）
    while q:
        u = q.popleft()                  # ② 从前面弹
        for v in adj[u]:
            if v not in dist:            # ③ 没访问过
                dist[v] = dist[u] + 1    # ④ 距离+1（无权图最短路）
                parent[v] = u
                q.append(v)              # ⑤ 后面入队
    return dist, parent
```

**🔍 代码识别特征**
- `deque` + `popleft()` + `append()` → **BFS 铁三角**
- `dist[v] = dist[u] + 1`（边权固定为 1）→ 无权图最短路
- 没有 PQ、没有权重 → 不是 Dijkstra

**不变量**：先出队的节点 dist 一定 ≤ 后出队的。
**用途**：无权图最短路、层次遍历、二分图判定（染色）。
**复杂度**：O(V+E)。

## DFS（深度优先）

```python
def dfs(adj, start):
    visited = set()
    pre, post = {}, {}
    timer = [0]                          # 用 list 包装，闭包内可改

    def visit(u):
        visited.add(u)
        pre[u] = timer[0]; timer[0] += 1   # ① 进入时间戳
        for v in adj[u]:
            if v not in visited:
                visit(v)                   # ② 递归深入
        post[u] = timer[0]; timer[0] += 1  # ③ 离开时间戳

    visit(start)
    return pre, post
```

**🔍 代码识别特征**
- **递归**调用自己 + `visited` 集合 → DFS（也可用 stack 显式实现）
- 有 `pre/post` 时间戳 → 用于拓扑排序、SCC、桥/割点
- 没有 queue、没有 dist 数组

**用途**：拓扑排序、判环、SCC（Tarjan/Kosaraju）、桥与割点、强连通。

### DFS 边分类（有向图）
- **Tree edge**：dfs 树上的边
- **Back edge**：指向祖先 → **有向图判环**信号
- **Forward edge** / **Cross edge**：仅有向图

无向图判环：DFS 中遇到非 parent 的 visited 节点。

## 拓扑排序

**Kahn（BFS 风）**：

```python
from collections import deque

def topo_kahn(adj, V):
    in_deg = {u: 0 for u in V}            # ① 算每个点的入度
    for u in V:
        for v in adj[u]: in_deg[v] += 1
    q = deque([u for u in V if in_deg[u] == 0])  # ② 入度=0 入队
    order = []
    while q:
        u = q.popleft()
        order.append(u)
        for v in adj[u]:
            in_deg[v] -= 1                # ③ "拆掉" u→v
            if in_deg[v] == 0:            # ④ 拆完入度为 0 → 入队
                q.append(v)
    if len(order) < len(V):
        return None                       # ⑤ 有环
    return order
```

**🔍 代码识别特征**
- **`in_deg` 数组 + 队列**（入度 0 的节点）→ Kahn 拓扑（BFS 风）
- `len(order) < V` 作环检测 → Kahn 招牌
- DFS 风：直接 DFS 完按 `post` 时间**降序**输出，识别签名是"DFS + post 排序"

## 连通分量
- 无向图：每次 DFS/BFS 一个未访问点
- 有向图 SCC：Kosaraju（两次 DFS）/ Tarjan（一次 DFS + 栈）

## 手算 mini（拓扑）
```
V = {a,b,c,d,e}, E = {a→b, a→c, b→d, c→d, d→e}
入度: a=0, b=1, c=1, d=2, e=1
顺序: a → b,c → d → e
```

## 高频陷阱
- 有向图判环 ≠ 无向图判环（前者看 back edge，后者看非 parent 的 visited）
- BFS 求带权最短路 → 错（要 Dijkstra）
- 拓扑排序前没判环
- 邻接矩阵在稀疏图上 → 浪费 O(V²)

---

# Module 12 · Shortest Paths

## 全章核心一个词：**松弛（Relaxation）**

```
relax(u, v, w):
    if dist[v] > dist[u] + w(u,v):
        dist[v] = dist[u] + w(u,v)
        parent[v] = u
```
**所有最短路算法 = 决定松弛的顺序**。

## 算法选择决策骨架

```
是图问题求最短路？
├── 无权 → BFS                              O(V+E)
├── 有权
│   ├── 全是非负 → Dijkstra                 O((V+E) log V)
│   ├── 有负权
│   │   ├── 单源 → Bellman-Ford             O(VE)
│   │   └── 全源 → Floyd-Warshall           O(V³)
│   ├── DAG → 拓扑序松弛                     O(V+E)
│   └── 全源非负 → Johnson                  O(VE log V)
```

## Dijkstra

```python
import heapq

def dijkstra(adj, s):
    # adj[u]: list of (v, weight)
    dist = {u: float('inf') for u in adj}
    dist[s] = 0
    pq = [(0, s)]                           # ① (距离, 节点) 最小堆
    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:                      # ② 过期项跳过
            continue
        for v, w in adj[u]:
            nd = d + w
            if nd < dist[v]:                 # ③ 松弛
                dist[v] = nd
                heapq.heappush(pq, (nd, v))  # ④ 不删旧项，直接压新项
    return dist
```

**🔍 代码识别特征**
- `heapq` + `dist` 字典 + **`if d > dist[u]: continue`**（过期项跳过）→ Dijkstra
- 松弛条件是 `dist[u] + w < dist[v]`（**累计距离**）
- 没有"循环 V-1 次"的外层 for → 不是 Bellman-Ford

**与 Prim 的关键区别**（外形几乎一样！）：
- Dijkstra：`nd = d + w`（**累加**当前距离）
- Prim：`nd = w`（**只看单边**权重）—— 这是唯一最重要的差别

**不变量**：被永久"确定"的节点（弹出时第一次见的）的 dist 不再变。
**为什么不能负权**：弹出 u 时假设 dist[u] 已最小，但若有负权边 (x→u, -10)，后来还能更小。
**复杂度**：O((V+E) log V) 用二叉堆；O(V² + E) 用数组（稠密图反而更快）。

![[dijkstra_relaxation.png]]

> 💡 想看 PQ 里"过期项"如何产生与跳过、逐步松弛动画？👉 [点这里在浏览器打开](dijkstra_relaxation.html)

## Bellman-Ford

```python
def bellman_ford(V, edges, s):
    # V: list of vertices; edges: list of (u, v, w)
    dist = {u: float('inf') for u in V}
    dist[s] = 0
    for _ in range(len(V) - 1):                  # ① 重复 V-1 轮
        updated = False
        for u, v, w in edges:                    # ② 扫所有边
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                updated = True
        if not updated: break                    # 提前收敛优化
    # 第 V 轮检测负环
    for u, v, w in edges:
        if dist[u] + w < dist[v]:                # ③ 还能松弛 → 有负环
            return None
    return dist
```

**🔍 代码识别特征**
- **双层 for**：外层 `range(V-1)` 重复 V-1 轮，内层扫**所有边** → Bellman-Ford 招牌
- **没有 PQ**（Dijkstra 才有）
- 末尾再来一轮检测负环 → 鉴别签名

**为什么 V-1 轮够**：最短路至多 V-1 条边（无负环时）。
**复杂度**：O(VE)。
**额外用途**：检测负环。

![[bellman_ford_rounds.png]]

> 💡 想看为什么需要 V−1 轮（信息逐轮传递）+ 第 V 轮如何检测负环？👉 [点这里在浏览器打开](bellman_ford_rounds.html)

## DAG 最短路
```
拓扑排序
按拓扑序逐个 u: 对每条出边松弛
```
O(V+E)，**比 Dijkstra 还快**，可处理负权（前提是 DAG）。

## Floyd-Warshall

```python
def floyd_warshall(V, edges):
    INF = float('inf')
    dist = {u: {v: 0 if u == v else INF for v in V} for u in V}
    for u, v, w in edges:
        dist[u][v] = w                            # 初始化直接边
    for k in V:                                   # ① k 必须最外层！
        for i in V:
            for j in V:
                if dist[i][k] + dist[k][j] < dist[i][j]:
                    dist[i][j] = dist[i][k] + dist[k][j]
    return dist
```

**🔍 代码识别特征**
- **三重 for 循环 (k, i, j)** + 二维 dist 矩阵 → Floyd-Warshall 一眼看出
- **`k` 必须在最外层**——这是 DP 的状态轴（"允许经过的中间点集合"）
- 没有 PQ、没有 visited、没有源点参数（全源）

**含义**：第 k 轮结束后，dist[i][j] = 只允许用 {V[0..k]} 作为中间点的最短路。
**复杂度**：O(V³)，能负权（无负环），全源最短路。
**判负环**：执行后看 `dist[i][i]` 是否 < 0。

## 手算 mini（Dijkstra）
```
图: A--1--B, A--4--C, B--2--C, B--5--D, C--1--D
源: A
初始: A=0, B=∞, C=∞, D=∞
弹 A(0) → 松弛 B=1, C=4
弹 B(1) → 松弛 C=min(4,1+2)=3, D=1+5=6
弹 C(3) → 松弛 D=min(6,3+1)=4
弹 D(4) → 完
答案: A→B=1, A→C=3, A→D=4
```

## 高频陷阱
- **Dijkstra + 负权** → 错
- **负环 + 求最短路** → 无意义（可无限减小），只能检测
- 用 BFS 求带权最短路 → 错
- Bellman-Ford 忘了执行 V-1 轮（不是 V 轮）
- 把 SPT（最短路径树）和 MST 混淆

---

# Module 13 · Minimum Spanning Trees

## 别和最短路混
- **MST**：把所有 V 个点连起来，用 V-1 条边，**总权重最小**
- **Shortest Path Tree**：以某点 s 为根，每条 s→v 的路径最短

它们一般**不一样**。

## 两个核心性质

### Cut Property（切割性质 → Kruskal/Prim 正确性的根）
对任意切割 (S, V\S)，**横跨切割的最小权边**一定在某棵 MST 中。

### Cycle Property
任意环路中**最大权边**一定**不在** MST 中（除非有相同权重）。

## Kruskal

```python
class DSU:
    def __init__(self, n): self.p = list(range(n))
    def find(self, x):
        while self.p[x] != x:
            self.p[x] = self.p[self.p[x]]    # 路径压缩
            x = self.p[x]
        return x
    def union(self, a, b):
        ra, rb = self.find(a), self.find(b)
        if ra == rb: return False
        self.p[ra] = rb
        return True

def kruskal(V, edges):
    edges.sort(key=lambda e: e[2])           # ① 边按权升序
    dsu = DSU(len(V))
    mst, total = [], 0
    for u, v, w in edges:
        if dsu.union(u, v):                  # ② 不在同一组才加
            mst.append((u, v, w))
            total += w
            if len(mst) == len(V) - 1: break # ③ V-1 条边就够
    return mst, total
```

**🔍 代码识别特征**
- **`edges.sort()` + `DSU/Union-Find`** → Kruskal 标志组合
- 没有 PQ、没有 dist 数组
- 以**边**为遍历单位（`for u, v, w in edges`）

**复杂度**：O(E log E) = O(E log V)。
**关键工具**：Union-Find（M6）。
**适合**：稀疏图 / 边已排序。

## Prim

```python
import heapq

def prim(adj, s):
    in_mst = set()
    pq = [(0, s, None)]                       # (边权, 到达点, 来源)
    mst, total = [], 0
    while pq and len(in_mst) < len(adj):
        w, u, parent = heapq.heappop(pq)
        if u in in_mst: continue              # 已加入 → 跳过
        in_mst.add(u)
        if parent is not None:
            mst.append((parent, u, w))
            total += w
        for v, ew in adj[u]:                  # 探索邻居
            if v not in in_mst:
                heapq.heappush(pq, (ew, v, u))# ① 只压**单边**权重 ew
    return mst, total
```

**🔍 代码识别特征**
- `heapq` + `in_mst` 集合 → Prim
- **入堆是 `(ew, v, u)`**——`ew` 是单边权重，**不是累计距离** ← 与 Dijkstra 唯一关键区别
- 没有 sort（区别于 Kruskal）
- 以**节点**为遍历单位（while 还有节点没进 MST）

**Dijkstra vs Prim 代码对比**（背下来这两行就能立刻分辨）：
```python
# Dijkstra: nd = d + w           # 累加距离（从源点到 v 的总和）
# Prim:     nd = w                # 单边权重（跨入 MST 的代价）
```

**复杂度**：O((V+E) log V) 用二叉堆；O(V² + E) 用数组（稠密图）。

**和 Dijkstra 目标的不同**：
- Dijkstra：dist[v] = 从源点到 v 的最短路径累计权重
- Prim：dist[v] = 从当前 MST 跨到 v 的**最小单边**权重

## 手算 mini（Kruskal）
```
边: (A,B,1), (B,C,2), (A,C,4), (C,D,1), (B,D,5)
排序: (A,B,1), (C,D,1), (B,C,2), (A,C,4), (B,D,5)
取 (A,B,1) → 取 (C,D,1) → 取 (B,C,2) → 已 V-1=3 条，停
MST: {(A,B,1),(C,D,1),(B,C,2)}, 权重 = 4
```

## 高频陷阱
- 误把 MST 当最短路径树
- Kruskal 不用 Union-Find → 判环代价升到 O(V)，总复杂度退化
- 边权有重复时，MST 不唯一（但权重唯一）
- Prim 的 dist 含义和 Dijkstra 不同 → 容易写错松弛条件

---

# 易混 6 组（精确分辨）

| 对比 | 区别要点 |
|---|---|
| **Greedy vs DP** | Greedy 选了不回头；DP 把所有相关子问题答案都存下 |
| **DP vs D&C** | DP 子问题**重叠**；D&C 子问题**独立** |
| **BFS vs Dijkstra** | BFS 无权 → 用队列；Dijkstra 非负权 → 用最小堆 |
| **Dijkstra vs Bellman-Ford** | Dijkstra 快但禁负权；Bellman-Ford 慢但允许负权 + 检测负环 |
| **MST vs SPT** | MST 总连通最便宜；SPT 单源到各点最短 |
| **Kruskal vs Prim** | Kruskal 以**边**为中心 + Union-Find；Prim 以**当前树**为中心 + 堆 |

---

# 复杂度速查（背到能脱口）

| 算法 | 复杂度 | 关键工具 |
|---|---|---|
| Activity Selection | O(n log n) | 排序 |
| Huffman | O(n log n) | 最小堆 |
| Merge Sort | O(n log n) | 递归 |
| Quick Sort | 期望 O(n log n) | 随机 pivot |
| 0-1 背包 | O(nW) | DP 表 |
| LCS | O(mn) | DP 表 |
| BFS / DFS | O(V+E) | 队列/栈 |
| 拓扑排序 | O(V+E) | Kahn 或 DFS |
| Dijkstra（堆） | O((V+E) log V) | 最小堆 |
| Bellman-Ford | O(VE) | 松弛 V-1 轮 |
| Floyd-Warshall | O(V³) | 三重循环 DP |
| Kruskal | O(E log V) | 排序 + Union-Find |
| Prim（堆） | O((V+E) log V) | 最小堆 |

---

# 考前必背 15 句

1. Greedy 必须证（**交换论证**），否则别用
2. Fractional Knapsack 用贪心，0-1 用 DP
3. D&C 子问题独立，DP 子问题重叠
4. Master Theorem：比较 f(n) 与 n^log_b(a)
5. DP 四要素：状态 / 转移 / 边界 / 顺序
6. 0-1 背包滚动数组要**倒序**
7. 完全背包**正序**
8. BFS 求**无权**最短路，O(V+E)
9. 有向图判环看 **back edge**；无向图看非 parent 的 visited
10. 拓扑排序：Kahn (BFS 风) 或 DFS post 降序
11. **松弛是所有最短路算法的核心**
12. Dijkstra **不能负权**；Bellman-Ford **能**且能检测负环
13. DAG 最短路 = 拓扑序松弛，O(V+E)
14. **MST 不是最短路径树**
15. Kruskal + **Union-Find**；Prim + **最小堆**

---

# 10 题速判（看到题先答这 10 个）

1. "每次取结束最早的活动" → ?
2. "暴力递归算 fib(50) 极慢" → ?
3. "课程依赖顺序" → ?
4. "无权图，从 s 到所有点的最短跳数" → ?
5. "图里有负权边" → ?
6. "求所有点对最短路" → ?
7. "Wi-Fi 全网最少线" → ?
8. "按边升序选，避免成环" → ?
9. "二维网格找最少操作转换字符串" → ?
10. "把数组排序，但不能比较元素，元素是 0..k 的小整数" → ?

### 答案
1. Greedy（活动选择）
2. DP（记忆化 fib）
3. 拓扑排序
4. BFS
5. Bellman-Ford
6. Floyd-Warshall（或 Johnson）
7. MST（Kruskal/Prim）
8. Kruskal
9. DP（编辑距离）
10. 计数排序（M7）

---

# 复习路径建议

**最低投入版（90 分钟）**：
1. 背"15 句"（15 分钟）
2. 在每个模块的"骨架"上手默写一遍（每个 10 分钟，共 60 分钟）
3. 做 10 题速判（15 分钟）

**进阶版（半天）**：
- 上面 + 每个模块的"手算 mini"亲自跑一遍（30 分钟）
- 再过一遍"高频陷阱"（15 分钟）

**冲刺版（一天）**：
- 上面 + 把 6 组易混对比写到独立卡片，反复看（45 分钟）
- 把 Kruskal、Prim、Dijkstra 三个最容易混的算法**手写完整代码**一遍（1 小时）

---

**建立日期**：2026-04-30
**v2 重写**：增加每模块伪代码骨架、不变量、手算例子、复杂度推导
**用途**：考前最后冲刺 / 给 AI 当上下文 / 建立"看到题秒判章节"的反射
