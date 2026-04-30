

> **教材章节**：Ch 12 · **对应周次**：Week 10
> **核心问题**：子问题**重复出现**时，记住每个子问题的答案 → 从指数变多项式。

---

## 0. 知识地图

```
               Dynamic Programming
                      │
        ┌─────────────┼─────────────┐
   成立条件        实现方式         经典题型
        │             │                │
    最优子结构    记忆化(Top-down)   0-1 背包
    重叠子问题    自底向上(Bottom-up) LCS / LIS
                  状态压缩          编辑距离
                                    矩阵链乘
                                    游戏博弈 DP
```

---

## 1. 思考脉络

1. 分治 → 如果子问题**重复**出现，反复计算极度浪费
2. 记住每个子问题答案 → **记忆化（Memoization）**
3. 更进一步：按依赖顺序迭代填表 → **Tabulation**
4. 成立条件：
   - **最优子结构**：全局最优由子问题最优组成
   - **重叠子问题**：子问题会被多次求解

---

## 2. 核心知识点

### 2.1 DP 四要素

1. **状态定义** dp[i] / dp[i][j] 的语义
2. **状态转移方程**
3. **边界条件**
4. **计算顺序**（保证依赖都已算好）

### 2.2 Top-down（记忆化搜索）

```python
memo = {}
def f(state):
    if state in memo: return memo[state]
    # base case
    # recursive case: dp[state] = ...
    memo[state] = result
    return result
```

- 直观、只算用到的状态
- 递归栈深

### 2.3 Bottom-up（自底向上）

```python
for i in range(n+1):
    for j in range(W+1):
        dp[i][j] = ...
```

- 没有递归栈开销
- 常可滚动数组降维（O(nW) → O(W)）

### 2.4 经典题 · 0-1 背包

```
dp[i][w] = max(
    dp[i-1][w],                        # 不选第 i 个
    dp[i-1][w - wᵢ] + vᵢ  if w ≥ wᵢ    # 选第 i 个
)
```
- O(nW)，**伪多项式**（W 是数值，不是规模）
- 滚动到 O(W)：**w 从大到小**遍历

### 2.5 经典题 · LCS（最长公共子序列）

```
dp[i][j] = dp[i-1][j-1] + 1       if A[i]=B[j]
         = max(dp[i-1][j], dp[i][j-1])  else
```
- O(mn)

### 2.6 经典题 · LIS（最长递增子序列）

- DP：O(n²)
- **贪心 + 二分**：维护 tails 数组 → **O(n log n)**

### 2.7 编辑距离（Levenshtein）

```
dp[i][j] = dp[i-1][j-1]            if A[i]=B[j]
         = 1 + min(dp[i-1][j],      删除
                   dp[i][j-1],      插入
                   dp[i-1][j-1])    替换
```
- O(mn)

### 2.8 矩阵链乘

- A₁A₂...Aₙ 的最小乘法次数
- `dp[i][j] = min(dp[i][k] + dp[k+1][j] + pᵢ₋₁·pₖ·pⱼ)` for k in [i, j-1]
- O(n³)

### 2.9 博弈 DP

- 两人最优对抗 → minimax
- `dp[i][j] = max(A[i] - dp[i+1][j], A[j] - dp[i][j-1])`

### 2.10 区间 DP、树 DP、状态压缩 DP

- 区间 DP：按区间长度从小到大递推
- 树 DP：DFS 回溯时聚合子树状态
- 状压 DP：状态用 bitmask 表示（适合 n ≤ 20）

---

## 3. Greedy vs DP vs D&C

| 方法 | 子问题 | 选择方式 |
|---|---|---|
| **D&C** | 独立子问题 | 合并 |
| **Greedy** | 最优子结构 | 局部最优（不回头） |
| **DP** | 最优子结构 + 重叠 | 穷举子问题，记表 |

---

## 4. 常考陷阱

- ❌ 没有最优子结构用 DP → 错误（需验证）
- ❌ 状态定义不清 → 转移方程必错
- ❌ 滚动数组时**遍历方向**没想清楚（0-1 背包必须倒序）
- ❌ 0-1 vs 完全背包：内外层 + 遍历方向区别
- ❌ 把 "伪多项式" 当 "多项式"（数值型复杂度）

---

## 5. 面试高频题

1. **爬楼梯 / 打家劫舍** —— 一维 DP 入门
2. **最长回文子串 / 子序列** —— 区间 DP
3. **编辑距离** —— 二维 DP
4. **零钱兑换** —— 完全背包
5. **股票买卖系列** —— 状态机 DP
6. **正则表达式匹配 / 通配符** —— 二维 DP
7. **戳气球** —— 区间 DP 经典难题

---

## 6. 三视角职业分析

- **工程师视角**：LeetCode 最大类别；思路是"定义状态 → 写转移"
- **研究员视角**：DP 与马尔可夫决策过程（MDP）的桥梁，引出强化学习
- **系统架构师视角**：数据库查询优化器使用 DP 求最优 join 顺序

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Dynamic Programming (DP) | 动态规划 |
| Memoization | 记忆化 |
| Tabulation | 自底向上填表 |
| Overlapping subproblems | 重叠子问题 |
| State / Transition | 状态 / 状态转移 |
| 0-1 Knapsack | 0-1 背包 |
| LCS / LIS | 最长公共子序列 / 最长递增子序列 |
| Edit distance | 编辑距离 |
| Bitmask DP | 状态压缩 DP |

---

## 8. 关联模块

- 对比 → [[Module 8: The Greedy Method]]、[[Module 9: Divide and Conquer]]
- 应用 → [[Module 12: Shortest Paths]]（Floyd-Warshall、Bellman-Ford 都是 DP）
- 工具 → [[Module 1: Algorithm Analysis]]（复杂度分析）
