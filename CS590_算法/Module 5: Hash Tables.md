
> **教材章节**：Ch 6 · **对应周次**：Week 5
> **核心问题**：牺牲顺序性，换取**期望 O(1)** 的查找 / 插入 / 删除。

---

## 0. 知识地图

```
                Hash Table
                    │
        ┌───────────┼───────────┐
     抽象基础      哈希函数     冲突解决
        │           │             │
    Map ADT     Hash Code    Separate Chaining
    Lookup Tbl  Compression  Open Addressing
                  MAD/Div      Linear / Quadratic
                  Tabulation   Double Hashing
                               Cuckoo Hashing
                    │
               理论保证
                    │
           Universal / Perfect Hashing
```

---

## 1. 思考脉络

1. **问题起点**：需要"根据 key 快速存/取/删 value"——Map（字典）抽象
2. **最简单实现**：无序链表——put O(1)，但 get/remove O(n)，大数据不可用
3. **直接跳到数组**：key 当索引，O(1)——但 key 空间太大（全部 SSN 需要 10 亿格）
4. **压缩 key 空间**：`h(key) → [0, m-1]`——**哈希函数**，代价是引入冲突
5. **哈希函数是两步组合**：先把 key 变成整数（hash code），再压缩到 [0, m-1]（compression）
6. **冲突一定会发生**（生日悖论），问题从来不是"避免"，而是"优雅处理"
7. **两条处理路线**：同槽挂链（Separate Chaining）vs 找下一个空槽（Open Addressing）
8. **理论保证**：Universal Hashing 让对手无法构造最坏输入

> **教授提醒**：哈希表在数据结构里极其重要，应用非常广泛；但也被大量误用——最常见的误区是"以为可以用哈希来排序"。哈希表给你的是快速查找，**不给你顺序**。

---

## 2. 核心知识点

### 2.1 Map ADT——哈希表解决的抽象问题

哈希表是 Map（也叫 Dictionary）这个抽象接口的一种实现。先搞清楚接口：

**Map = 存储 (key, value) 对的可搜索集合**，不允许重复 key。

**核心操作**：

| 操作 | 语义 | 返回值 |
|---|---|---|
| `get(k)` | 返回 key k 对应的 value | 找到返回 v，否则 null |
| `put(k, v)` | 插入 (k, v)；若 k 已存在，**替换** value | 被替换的旧值（或 null） |
| `remove(k)` | 删除 key k 的条目 | 被删除的 value（或 null） |
| `size()` / `isEmpty()` | — | — |

**真实应用场景**：
- **网络路由器**：收到 IP 数据包 (目标IP, 数据)，需极快查到"该把这包送往哪个网络接口"——`get(dest_ip)` 必须 O(1)，因为路由器每秒处理百万级数据包
- **地址簿**：姓名 → 联系方式
- **学生数据库**：学号 → 学籍记录
- **编译器符号表**：变量名 → 内存地址

---

### 2.2 List-based Map——最简实现的局限

最简单的 Map 实现：双向链表，随机顺序存储所有 (key, value)：

```
[header] ↔ (9,C) ↔ (6,A) ↔ (5,D) ↔ (8,B) ↔ [trailer]
```

| 操作 | 复杂度 | 原因 |
|---|---|---|
| put | **O(1)** | 插到链表头，不排序 |
| get | O(n) | 最坏要扫完整个链表 |
| remove | O(n) | 先要找到再删 |

**适用场景**：小 map，或 put 远多于 get/remove 的历史记录类场景。大数据完全不行。

---

### 2.3 Lookup Table——从直接寻址到哈希的过渡

如果 key 已知是整数且范围是 `[0, N-1]`：直接用数组，key 当索引——所有操作 O(1)。

```python
A = [None] * N
A[k] = v      # put(k, v)
return A[k]   # get(k)
```

**问题**：现实 key 不在 `[0, N-1]` 范围内。
- 社会安全号（9位数）→ 需要 10 亿格数组——内存炸
- 字符串 key → 根本不是整数

**解决方案**：用哈希函数把任意 key 映射到 `[0, m-1]`，m 是我们控制的表大小。

---

### 2.4 哈希函数设计——两步组合

**教授强调**：哈希函数实际上是一个**复合函数** `h = h₂ ∘ h₁`：

```
任意 key  →[h₁: hash code]→  整数  →[h₂: compression]→  [0, m-1]
```

**好哈希函数的两个目标**：
![[Pasted image 20260429235343.png]]

1. 计算快（O(1) 或接近）
2. 分布均匀（尽量减少冲突）

---

#### 第一步：Hash Code（任意 key → 整数）

根据 key 的类型选择不同策略：

**① 内存地址法（Memory Address）**
- 把 key 对象的内存地址直接当整数
- 适合"对象身份"作 key 的场景；不适合字符串（相同内容可能地址不同）

**② 整数转换法（Integer Cast）**
- 把 key 的二进制位直接重新解释为整数
- 适合 byte/short/int/float 等长度 ≤ 整数位数的 key

**③ 分量求和法（Component Sum）**
- 把 key 的二进制分成固定长度的分量（如16位或32位），然后求和（忽略溢出）或 XOR
- 适合固定长度的数字类型 key

**④ 多项式累积法（Polynomial Accumulation）**——最重要，字符串专用

把字符串 `s = a₀a₁...aₙ₋₁` 的每个字符当系数，计算多项式：

```
p(z) = a₀ + a₁z + a₂z² + ... + aₙ₋₁zⁿ⁻¹
```

实验表明：**z = 33** 时，对50,000个英语单词只产生6次碰撞。Java 的 `String.hashCode()` 用的就是这个思路（z = 31）。

**效率问题**：直接计算 zⁿ 很贵。用 **Horner's Rule** 降到 O(n)：

```
p(z) = a₀ + z(a₁ + z(a₂ + ... + z·aₙ₋₁)...)
```

从内层往外乘，每步一次乘法一次加法，n-1 步完成。

**⑤ 表格化哈希法（Tabulation-based Hashing）**

把 key 视为元组 `k = (x₁, x₂, ..., xₐ)`，每个 xᵢ ∈ [0, M-1]：
- 预先初始化 d 张随机表 T₁...Tₐ，每张大小 M，每格是 [0, N-1] 的随机数
- 哈希值：`h(k) = T₁[x₁] ⊕ T₂[x₂] ⊕ ... ⊕ Tₐ[xₐ]`（XOR）

可以数学证明碰撞概率 ≈ 1/N，接近完美随机函数的效果。

---

#### 第二步：压缩函数（整数 → [0, m-1]）

**① 除法法（Division）**

```
h₂(y) = y mod m
```

**为什么 m 必须取素数？** 教授的解释：素数能打散数据中的周期性。如果数据中有很多 4 的倍数（比如内存地址），m = 8 时只会映射到 0/4/0/4...；m = 7（素数）会把规律彻底打乱。实践中：直接 Google 搜"prime number greater than 1 million 300,000"，用列出的素数就行。

**② MAD（Multiply-Add-Divide）/ 随机线性函数**

```
h₂(y) = (ay + b) mod m
```

- a, b 是随机非负整数，且 `a mod m ≠ 0`（否则所有 key 都映射到同一个槽）
- 这也叫"线性哈希函数"——记住是 ay + b 这条线
- **这就是 Universal Hashing 的核心构造**（见 2.11）

---

### 2.5 负载因子——所有性能的总开关

$$\alpha = \frac{n}{m}$$

- n = 表中元素数，m = 槽数
- **α 是哈希表性能最重要的单一指标**

| 实现 | α 的含义 | 建议阈值 |
|---|---|---|
| Separate Chaining | 平均链长 = α（允许 > 1）| 保持 α < 1 为佳 |
| Open Addressing | 槽使用率（必须 < 1）| α < 0.75 |

α 接近 1 时，Open Addressing 性能急剧恶化（下面会推导）。

---

### 2.6 冲突——它一定会发生（生日悖论）

哈希函数再好，**冲突必然发生**——不是设计缺陷，是数学定律。

![[Pasted image 20260429235629.png]]

**生日悖论**：23 个人中，有两人同生日的概率 > 50%。为什么？23 人产生 23×22/2 = 253 对组合，每对碰撞概率 1/365，累积起来就超过 50%。

**映射到哈希表**：m 个槽仅需 √m 个 key 就很可能产生冲突。问题从来不是"避免冲突"，而是"优雅处理冲突"。

---

### 2.7 冲突解决一：链地址法（Separate Chaining）

![[Pasted image 20260429235829.png]]

**设计**：每个槽不存元素，而是存一个链表（或小型 BST）。冲突的元素都挂在同一个链表上。

**算法伪代码**：

```
get(k):
    if A[h(k)] == null: return null
    else: return A[h(k)].search(k)   # 在链表里线性搜索

put(k, v):
    if A[h(k)] == null: A[h(k)] = new LinkedList
    return A[h(k)].put(k, v)         # 链表内处理（O(1)插头）

remove(k):
    if A[h(k)] == null: return null
    else: return A[h(k)].remove(k)
```

**性能**（假设哈希均匀随机）：
- 每个槽的期望长度 = α = n/m
- 查找期望：O(1 + α)，若 α = O(1) 则 O(1)
- 插入：O(1)（链表头插）

**Java HashMap 的优化**：链表长度超过 8 时自动转成红黑树，查找从 O(链长) 降到 O(log 链长)，防止退化攻击。

**优点**：实现简单，允许 α > 1，删除容易（直接从链表删）。
**缺点**：需要额外内存存指针；链节点分散在内存，缓存不友好。

---

### 2.8 冲突解决二：开放寻址（Open Addressing）

**核心思想**：所有元素存在表内，冲突时"探测"下一个可用槽。三代演进：

![[Pasted image 20260430000042.png]]

探测序列：`h(k, i) for i = 0, 1, 2, ...`

---

#### ① 线性探测（Linear Probing）

```
h(k, i) = (h(k) + i) mod m
```

**示例**（教材例子）：m = 13，h(x) = x mod 13，插入 18, 41, 22, 44, 59, 32, 31, 73：

```
索引:  0   1   2   3   4   5   6   7   8   9  10  11  12
值:   [_] [41][_] [_] [_] [_] [18][_] [_] [_] [_] [_] [_]
                               ↑ 18 mod 13 = 5, 冲突 → 放6
```

**查找算法**：
```
get(k):
    i = h(k);  p = 0
    repeat:
        c = A[i]
        if c == EMPTY: return null        # 空槽 → 肯定不存在
        elif c.key == k: return c.value   # 找到
        else: i = (i+1) mod m; p++       # 继续探测
    until p == m
    return null
```

**一次聚集问题（Primary Clustering）**：hash 到相邻位置的 key 形成"连续块"，连续块越来越长，后续探测越来越慢。

**期望探测次数** ≈ 1/(1-α)，当 α → 1 时趋于无穷。

---

#### ② 二次探测（Quadratic Probing）

```
h(k, i) = (h(k) + c₁i + c₂i²) mod m
```

打破了连续块，但有**二次聚集（Secondary Clustering）**：hash 值相同的 key 走完全相同的探测路径，形成"虚拟的连续块"。

---

#### ③ 双哈希（Double Hashing）——最均匀

```
h(k, i) = (h₁(k) + i·h₂(k)) mod m
```

步长由第二个哈希函数决定，不同 key 走不同路径，彻底打散。

**约束**：
- m 必须是素数（保证所有槽都能被探到）
- h₂(k) 不能为 0
- 常用：`h₂(k) = q - (k mod q)`，其中 q < m 且 q 是素数

**教材示例**：m = 13，h₁(k) = k mod 13，h₂(k) = 7 - k mod 7，插入 18, 41, 22, 44, 59, 32, 31, 73：

```
索引:  0   1   2   3   4   5   6   7   8   9  10  11  12
值:   [_] [41][18][32][59][_] [_] [73][22][44][31][_] [_]
```
比线性探测分布均匀得多。

**三种探测的递进逻辑**：
- 线性：打散 hash 冲突，但制造了**位置聚集**（连续块）
- 二次：打破连续块，但制造了**路径聚集**（相同 hash 走相同跳法）
- 双哈希：步长也随机化，**彻底打散**，是 open addressing 的最优选择

---

### 2.9 删除的陷阱——墓碑机制

**开放寻址最致命的问题**：不能直接清空已删除的槽。

![[tombstone_necessity.png]]

> 💡 想点按钮逐步看探测链断裂的过程？👉 [点这里在浏览器打开](tombstone_necessity.html)

**原因**：查找代码依赖"遇到空槽就停"来判断"元素不存在"。直接删除会截断探测链，导致后面的元素永远找不到。

**两种修复方案**：

**方案 A：墓碑（Tombstone）标记**——工程首选
```
删除时：A[i] = TOMBSTONE（不是 EMPTY）
查找时：遇到 TOMBSTONE 继续探测（不停）
插入时：遇到 TOMBSTONE 可以复用此位置
```

**方案 B：向左移位**
```
删除后：把后续探测链上的元素逐个前移填洞
优点：不需要墓碑，表更紧凑
缺点：实现复杂；每次删除可能要移动多个元素
```

**关键理解**：墓碑让探测链保持完整——查找时跳过它继续探，插入时把它当空槽复用。Python dict、Go map 等用的就是 tombstone 方案。

---

### 2.10 Cuckoo Hashing——最坏情况 O(1) 查找

**动机**：Separate Chaining 和 Open Addressing 的查找都是**期望** O(1)——最坏情况仍 O(n)。能否做到**最坏 O(1)**？

**设计**："两个选择的力量"（Power of Two Choices）

- 维护**两张表** T₀、T₁，各大小 N（N ≥ 2n）
- 各有哈希函数 h₀、h₁
- 每个 key k **只有两个可能的位置**：T₀[h₀(k)] 或 T₁[h₁(k)]

**查找（Worst-case O(1)）**：
```
get(k):
    if T₀[h₀(k)].key == k: return T₀[h₀(k)].value
    if T₁[h₁(k)].key == k: return T₁[h₁(k)].value
    return null
```
只需**最多检查两个位置**，严格 O(1)。

**插入（布谷鸟驱逐）**——名字的由来：
> 布谷鸟（Common Cuckoo）是巢寄生鸟——它把蛋下在别的鸟巢里，先把原来的蛋踢走。

```
put(k, v):
    尝试放入 T₀[h₀(k)]
    若该位置已有元素 (k', v'):
        → 把 (k,v) 放入，驱逐 (k',v')
        → (k',v') 去找它的另一个位置 T₁[h₁(k')]
        → 若 T₁[h₁(k')] 也有元素，再次驱逐...
    直到找到空槽，或检测到驱逐循环
    若出现驱逐循环 → 用新哈希函数重新 rehash 所有元素
```

**性能**：
- 查找：**最坏 O(1)**
- 删除：**最坏 O(1)**（找到后直接删）
- 插入：期望 O(1) 摊还（长驱逐链极少发生）
- 循环概率很低，需要 rehash 的概率极小

---

### 2.11 Universal Hashing——对抗性输入的理论保证

**问题**：如果攻击者知道你的哈希函数，可以构造一组 key 让它们全部碰撞（HashDoS 攻击），退化到 O(n)。

**解决方案**：从**函数族** H 中**随机**选一个函数，让对手无法预测。

**定义（Universal / 2-universal 家族）**：
> 对任意两个不同 key j ≠ k，从 H 中随机选 h，碰撞概率 Pr[h(j) = h(k)] ≤ 1/m

**定理**（Theorem 6.3）：MAD 函数族是 Universal 的。

```
H = { h_{a,b}(k) = (ak + b mod p) mod m }
  其中：p 是 ≥ M 的素数（M = key 空间大小）
         0 < a < p，0 ≤ b < p
```

这个族里随机选一个函数，任意两个不同 key 的碰撞概率 ≤ 1/m。对手不知道随机选的 a, b，无法构造最坏输入。

---

### 2.12 Perfect Hashing——静态场景的最优解

**适用条件**：key 集合**不会改变**（静态），建表后只有查询。

**设计**：两层哈希，第二层保证**零碰撞**：
- 第一层：Universal hash 函数把 n 个 key 分到 m 个桶
- 第二层：对每个桶再建一个小哈希表，大小 = 桶内元素数²，保证零碰撞

**性能**：
- 查找：**最坏 O(1)**（两次哈希即可）
- 空间：期望 O(n)
- 建表时间：期望 O(n)

**限制**：只适合不变集合（编译期常量表、静态路由表等）。有任何增删就要重建。

---

## 3. 复杂度对比

| 方案 | 查找（期望）| 查找（最坏）| 插入（期望）| 删除（期望）| 特点 |
|---|---|---|---|---|---|
| List-based Map | O(n) | O(n) | O(1) | O(n) | 无用，仅作对比 |
| Separate Chaining | O(1+α) | O(n) | O(1) | O(1+α) | 简单，α 可 > 1 |
| 线性探测 | O(1/(1-α)) | O(n) | O(1/(1-α)) | O(1/(1-α)) | 聚集严重 |
| 双哈希 | O(1/(1-α)) | O(n) | O(1/(1-α)) | O(1/(1-α)) | 接近理想 |
| **Cuckoo** | **O(1) 最坏** | **O(1)** | O(1) 摊还 | **O(1) 最坏** | 查找有硬上界 |
| Perfect | O(1) | **O(1)** | — 静态 — | — 静态 — | 只适合不变集合 |

> 为什么所有方案"最坏"都是 O(n)（除了 Cuckoo 和 Perfect）？因为如果哈希函数极端退化（全部碰撞），所有元素堆在同一个槽/探测链里，查找要扫完所有 n 个元素。Universal Hashing 让这种极端退化的概率趋近于零，但不能完全排除。

---

## 4. 常考陷阱

- ❌ **Hash 表不保证顺序**——要有序遍历用 Tree；BST 的优势正是此
- ❌ **Open addressing 删除不能直接清空**——必须用 tombstone 或移位，否则查找链断
- ❌ **m 取 2^k 时除法法退化**——只看低 k 位，数据有规律时碰撞严重；要取素数
- ❌ **字符串哈希要覆盖所有字符**——只看首字母或首几个字符会大量碰撞
- ❌ **以为 hashCode 均匀**——对抗性输入可制造 O(n)（HashDoS）；Java 在 HashMap 用了随机种子防御
- ❌ **Cuckoo Hashing 的插入不是 O(1) worst-case**——查找和删除才是；插入是期望 O(1) 摊还
- ❌ **α mod m ≠ 0 这个条件**——MAD 中 a mod m = 0 会让所有 key 映射到同一槽
- ❌ **"哈希能排序"是误区**——不能；哈希破坏了元素间的顺序关系

---

## 5. 面试高频题

1. **两数之和（Two Sum）** —— Hash 存 `target - x`，一次遍历 O(n)
2. **最长不重复子串** —— 滑动窗口 + HashSet，O(n)
3. **LRU Cache** —— HashMap（O(1) 查找）+ 双向链表（O(1) 移位）
4. **设计哈希表** —— 考察 Chaining vs Open Addressing 的取舍理由
5. **为什么 Java HashMap 容量是 2 的幂？** —— `hash & (n-1)` 代替取模（位运算更快），但需要额外 spread hash 防低位碰撞
6. **什么是 tombstone？为什么需要？** —— 见 2.9，探测链断裂问题
7. **解释 Universal Hashing** —— 为什么要随机选函数族，对抗 HashDoS

---

## 6. 三视角职业分析

- **工程师视角**：`std::unordered_map`（C++，open addressing）、Python `dict`（open addressing + tombstone）、Java `HashMap`（chaining，链表转红黑树）是日常工具；Redis、Memcached 以哈希表为核心存储引擎
- **研究员视角**：Universal Hashing 是随机化算法的经典案例（Pr 分析）；Cuckoo Hashing 是理论最优的实用哈希方案之一；Bloom Filter（基于多哈希）是近似集合查询的研究热点
- **系统架构师视角**：分布式缓存用**一致性哈希（Consistent Hashing）**——普通取模在节点增减时大量 key 失效，一致性哈希把影响控制在 O(1/n) 的 key；HashDoS 攻击（2011年曝光）让 PHP/Java/Python 都加入了哈希随机化

---

## 7. 术语速查表

| 英文 | 中文 |
|---|---|
| Map / Dictionary | 映射 / 字典 |
| Hash function | 哈希函数 |
| Hash code | 哈希码（第一步：key→整数）|
| Compression function | 压缩函数（第二步：整数→[0,m-1]）|
| Collision | 冲突 |
| Load factor α | 负载因子 |
| Separate chaining | 链地址法 |
| Open addressing | 开放寻址 |
| Linear probing | 线性探测 |
| Quadratic probing | 二次探测 |
| Double hashing | 双重哈希 |
| Primary clustering | 一次聚集（线性探测的问题）|
| Secondary clustering | 二次聚集（二次探测的问题）|
| Tombstone | 墓碑（开放寻址删除标记）|
| Cuckoo hashing | 布谷鸟哈希 |
| Universal hashing / 2-universal | 全域哈希 |
| MAD | Multiply-Add-Divide 压缩函数 |
| Perfect hashing | 完美哈希 |
| Horner's rule | 霍纳规则（多项式 O(n) 求值）|
| Tabulation-based hashing | 表格化哈希 |
| Polynomial accumulation | 多项式累积哈希码 |
| Rehash | 重哈希（扩容或更换哈希函数）|

---

## 8. 关联模块

- 前置 → [[Module 1: Algorithm Analysis]]（期望复杂度分析）、[[Module 2: Basic Data Structures]]（链表，用于 chaining）
- 对比 → [[Module 3: Binary Search Trees]]（有序 vs 无序；range query 哈希做不到）
- 图算法使用 → [[Module 11: Graphs and Traversals]]（邻接表可用哈希存储）
- 延伸 → Consistent Hashing（分布式系统）、Bloom Filter（近似集合查询）
