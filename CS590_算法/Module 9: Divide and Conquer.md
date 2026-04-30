# Module 9: Divide and Conquer

> **教材章节**：Ch 11 · **对应周次**：Week 9
> **核心问题**：把大问题拆成若干**独立**子问题，递归求解后合并。如何分析其复杂度？

---

## 0. 知识地图

```
              Divide and Conquer
                     │
        ┌────────────┼────────────┐
    三步走        递归式分析       经典应用
    Divide        Master Theorem   Karatsuba
    Conquer       递归树           Strassen
    Combine       代换法           Closest Pair
                                   FFT / 整数/矩阵乘法
```

---

## 1. 思考脉络

1. 大问题 → 分成若干**规模更小**的独立子问题
2. 递归解决每个子问题
3. 合并子问题答案得到原问题答案
4. 核心递推式：**T(n) = a · T(n/b) + f(n)**
5. 三种工具求解：**Master Theorem**、**递归树**、**代换法**

---

## 2. 核心知识点

### 2.1 三步框架

```
def DC(P):
    if 规模小:
        直接解
    else:
        分 P 为 P₁,...,Pₐ  (Divide)
        递归解 DC(Pᵢ)       (Conquer)
        合并得答案            (Combine)
```

### 2.2 Master Theorem ⭐

#### 一句话讲清它在干什么

T(n) = a·T(n/b) + f(n) 这个递推式里有**两股力量在比赛**：
- 力量 A：每深一层，递归调用变多 ×a（更多节点）
- 力量 B：每深一层，子问题变小 ÷b^d（每个节点的活变少；这里 d 是 f(n) = n^d 的指数）

**Master Theorem 只回答一个问题：哪股力量赢？**

谁赢决定了递归树的"代价分布形状"——以及总代价由哪一层决定。

#### 三种形状（看图就懂）

![[master_theorem_visualizer.png]]

> 💡 想拖滑块改 a、b、d 实时看形状变化？👉 [点这里在浏览器打开](master_theorem_visualizer.html)

#### 用一个数 r 判断（替代教材里的 ε）

设 f(n) = n^d，定义**比例数**：

$$r = \frac{a}{b^d}$$

r 就是"每深一层，总代价乘以多少"。

| r | 形状 | 谁赢 | Case | T(n) |
|---|---|---|---|---|
| **r > 1** | ↗ 越深越大 | 叶子层 | **Case 1** | **Θ(n^c)**, c=log_b(a) |
| **r = 1** | ═ 各层等高 | 打平 | **Case 2** | **Θ(n^c · log n)** |
| **r < 1** | ↘ 越深越小 | 根层 | **Case 3** | **Θ(f(n))** |

#### 为什么是这样（一行直觉）

- **Case 1**（r > 1）：每层代价像等比数列上升 → 总和被最末层（叶子）支配。叶子总数 = a^(log_b n) = n^c → 总 = Θ(n^c)
- **Case 2**（r = 1）：每层代价相同（≈ n^c）→ 共 log n 层 → 总 = Θ(n^c · log n)
- **Case 3**（r < 1）：每层代价像等比数列下降 → 总和被最首层（根）支配，根的代价就是 f(n) → 总 = Θ(f(n))

**c 不是用来分类的**，c 是用来**计算 Case 1 答案**的（叶子有多少 = n^c）。分类用 r。

#### 实战流程（三步走）

1. 把 f(n) 写成 n^d 形式（如 f(n) = n → d=1；f(n) = n² → d=2；f(n) = 1 → d=0）
2. 算 r = a / b^d
3. 看 r 与 1 的关系，套上表

#### 用这套方法重新做经典例子

| 算法 | a | b | f(n) | d | r = a/b^d | 形状 | c=log_b(a) | T(n) |
|---|---|---|---|---|---|---|---|---|
| Merge Sort | 2 | 2 | n | 1 | 2/2 = **1** | 等高 | 1 | **Θ(n log n)** |
| Binary Search | 1 | 2 | 1 | 0 | 1/1 = **1** | 等高 | 0 | **Θ(log n)** |
| Karatsuba | 3 | 2 | n | 1 | 3/2 = **1.5** | 上升 | 1.585 | **Θ(n^1.585)** |
| Strassen | 7 | 2 | n² | 2 | 7/4 = **1.75** | 上升 | 2.807 | **Θ(n^2.807)** |
| 反例 1 | 2 | 2 | n² | 2 | 2/4 = **0.5** | 下降 | 1 | **Θ(n²)** |
| 反例 2 | 4 | 2 | n | 1 | 4/2 = **2** | 上升 | 2 | **Θ(n²)** |

#### 跟教材里的 ε 怎么对应？

教材里的 `f(n) = O(n^(c-ε))` 等价于 d < c，等价于 b^d < b^c = a，等价于 a/b^d > 1，等价于 **r > 1**。三种说法说的是同一件事，r 这个角度最直观。

### 2.3 递归树法

画出递归树：
- 第 k 层：n/bᵏ 规模，aᵏ 个节点
- 每层代价 f(n/bᵏ) · aᵏ
- 求各层代价和 + 叶子代价

![[recursion_tree_explorer.png]]

> 💡 想切换 Merge Sort / Karatsuba / Strassen 三种 case 看每层代价的差异？👉 [点这里在浏览器打开](recursion_tree_explorer.html)
>
> **三种 case 的视觉直觉**：
> - **Case 2（Merge Sort）**：每层代价相等 → log n 层均摊 → n·log n
> - **Case 1（Karatsuba/Strassen）**：节点数爆炸主导 → 叶子层决定总代价
> - **Case 3**（一般出现在合并步很重时）：根代价主导，递归层无关紧要

### 2.4 代换法 / 归纳法

先猜答案 T(n) = O(g(n))，再用归纳法验证。

### 2.5 经典应用 · 整数乘法（Karatsuba）

- 普通竖式：O(n²)
- 分治：x = x₁·B + x₀, y = y₁·B + y₀
- 原始需要 4 次乘法 → Karatsuba **只需 3 次**：
  - (x₁+x₀)(y₁+y₀) − x₁y₁ − x₀y₀
- **O(n^log₂ 3) ≈ O(n^1.585)**

![[karatsuba_walkthrough.png]]

> 💡 想用 1234 × 5678 一步步走完 4 次乘法 → 3 次乘法的全过程？👉 [点这里在浏览器打开](karatsuba_walkthrough.html)
>
> **关键 insight**：加减法是"白嫖"的，每层省 1 次乘法 → 递归层层叠加 → O(n²) 降到 O(n^1.585)。Strassen 矩阵乘法（8→7 次乘）是同一思路在矩阵世界的应用。

### 2.6 矩阵乘法（Strassen）

- 普通 O(n³)
- Strassen：把 n×n 分成 2×2 块，**7 次乘法**（普通需 8 次）
- **O(n^log₂ 7) ≈ O(n^2.807)**

### 2.7 Closest Pair of Points

- 1D：排序 → 扫描相邻
- 2D：
  1. 按 x 排序后分两半递归
  2. d = min(左, 右)
  3. 检查跨越中线的 2d 条带内的点对
  4. 利用 y 排序剪枝，每点最多比 6 个
- **O(n log n)**

![[closest_pair_animation.png]]

> 💡 想看分治 + 2d 条带 + 7 点引理的完整 6 步动画？👉 [点这里在浏览器打开](closest_pair_animation.html)
>
> **这道题的精髓**：合并步看似要 O(n²) 检查所有跨界对，但通过 y 排序 + 7 点引理（宽 2d 高 d 的矩形最多塞 8 个点），降到 O(n)。这是分治算法里"合并步聪明地用结构性约束"的经典案例。

### 2.8 Convex Hull（凸包）

- 分治法：按 x 排序分两半，递归求左右凸包，合并
- O(n log n)

---

## 3. 算法复杂度速查

| 算法 | 递推式 | 复杂度 |
|---|---|---|
| Binary Search | T(n/2) + O(1) | O(log n) |
| Merge-Sort | 2T(n/2) + O(n) | O(n log n) |
| Quick-Sort 平均 | 2T(n/2) + O(n) | O(n log n) |
| Karatsuba | 3T(n/2) + O(n) | O(n^1.585) |
| Strassen | 7T(n/2) + O(n²) | O(n^2.807) |
| Closest Pair | 2T(n/2) + O(n) | O(n log n) |
| BFPRT | T(n/5) + T(7n/10) + O(n) | O(n) |

---

## 4. 常考陷阱

- ❌ Master Theorem **三种情况之间有 gap**（例如 n·logⁿ 不符合任何一种）→ 要用递归树
- ❌ 子问题必须**规模减小且独立** —— 否则退化成不合算的递归
- ❌ Case 3 的正则条件 `a·f(n/b) ≤ k·f(n), k < 1` 容易被忽略
- ❌ Karatsuba 不是"3 步变 1 步"，而是"4 次乘法减到 3 次"

---

## 5. 面试高频题

1. **Merge-Sort 实现 + 求逆序对** —— 合并时统计
2. **数组中第 K 大** —— Quickselect / BFPRT
3. **矩阵中查找（有序）** —— 每步排除 1/4 区域
4. **最大子数组和** —— 分治 / Kadane（DP）
5. **幂运算 x^n** —— O(log n) 快速幂

---

## 6. 三视角职业分析

- **工程师视角**：递归思维的底子；所有分治算法都是范本
- **研究员视角**：矩阵乘法下界（ω）是未解难题（当前 ≈ 2.37）
- **系统架构师视角**：MapReduce 本质是分治；Spark 的 shuffle + reduce

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Divide and Conquer | 分治 |
| Recurrence | 递推式 |
| Master Theorem | 主定理 |
| Recursion tree | 递归树 |
| Substitution method | 代换法 |
| Karatsuba | Karatsuba 整数乘法 |
| Strassen's algorithm | Strassen 矩阵乘法 |
| Closest pair | 最近点对 |

---

## 8. 关联模块

- 前置 → [[Module 1: Algorithm Analysis]]（Master Theorem）
- 方法学兄弟 → [[Module 8: The Greedy Method]]、[[Module 10: Dynamic Programming]]
- 应用实例 → [[Module 7: Sorting and Selection]]（Merge/Quick/BFPRT）
