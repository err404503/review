# 241880136 构造 1（热身）学习

> 论文：*How to Approximate A Set Without Knowing Its Size In Advance* (Pagh, Segev, Wieder, 2013)
> 主要参考：论文 §1.3、§4 原文，参考Roman-Rao相关论文补全了原论文证明链。
> 学习日期：2026/7/17

---

## 定理 4.1

对任意 $0 < \epsilon < 1$，存在一个针对未知大小集合的近似成员数据结构，满足：

| 属性 | 值 |
|---|---|
| 假阳性率 | $\le \epsilon$ |
| 空间（$n$ 次插入后） | $(1+o(1))n\log(1/\epsilon) + O(n\log\log n)$ |
| 插入时间 | 期望摊还 $O(1)$ |
| 查询时间 | $O(\log n)$ |
| 已知 $n$？ | 否 |
| 假阴性 | 无 |

---

## 构造思路

### 关于Roman-Rao字典

一个精确字典。简单阅读Roman-Rao原文（Rajeev Raman, S. Srinivasa Rao. Succinct Dynamic Dictionaries and Trees. ICALP 2003, LNCS 2719, pp. 357–368. DOI: 10.1007/3-540-45061-0_30）的摘要，它支持期望摊还$O(1)$的插入、最坏$O(1)$的查询、空间是紧凑的，大小为$B+o(B)$，其中 $B = \left\lceil \lg\binom{m}{n} \right\rceil$ 为信息论下界。

### 框架

- 把插入流按分块，第 $i$ 块大小为 $2^i$ ，每块由专门的 $B_i$ 处理
- 第 $i$ 个子序列开始时，分配并初始化 $B_i$，其假阳性率控制到 $\Theta(\epsilon/i^2)$
- 共需要约 $\log n$ 个 $B_i$

### 插入及其时间复杂度

绝大多数插入是直接插入当前的 $B_i$。新建 $B_i$ 的成本是其大小本身，因此插入全局来看摊还到 $O(1)$ 。

### 查询及其时间复杂度

遍历所有 $B_i$，因为不知道元素属于哪一块。任一返回 Yes 则判定存在。 $B_i$ 是一个Roman-Rao字典，查询复杂度最坏$O(1)$，总查询时间复杂度 $O(\log n)$ 。

### B_i 内部设计及整体假阳性率

B_i 是一个数据存储结构，存储逻辑为：

- 哈希函数 $h: U \to [2^i/ε_i]$，碰撞概率 $\epsilon/i^2$
- 对每个插入元素 $x$，计算 $h(x)$ ，把结果存入Raman-Rao（一个精确字典）
- 联合处理：所有 $B_i$ 的总假阳性率$\sum_{i=1}^{\infty} \epsilon_i = \Theta(\epsilon \cdot \pi^2/6) \approx \epsilon$

### 整体空间复杂度

由Raman-Rao 的空间结论，$B = \left\lceil \lg\binom{m}{n} \right\rceil \approx n\log\frac{m}{n}$。

代入 $n = 2^i$，$m = 2^i / \epsilon_i$，$m/n = 1/\epsilon_i$，单个 $B_i$ 的空间为：$(1+o(1)) \cdot 2^i \cdot \log\frac{1}{\epsilon_i}$。

每个元素开销不超过最贵那个，$\Sigma_{i} (1+o(1)) \cdot 2^i \cdot \log\frac{1}{\epsilon_i} \le (1+o(1)) \cdot \max_{i}\bigl\{\log(1/\epsilon_i)\bigr\} \cdot \Sigma_{i} 2^i = (1+o(1)) \cdot n \cdot \max_{i}\bigl\{\log(1/\epsilon_i)\bigr\}$。

由 $\epsilon_i = \Theta(\epsilon/i^2)$，$\log(1/\epsilon_i) = \log(1/\epsilon) + 2\log i + O(1)$。

由设定 $i < \log n$，直接取$\max i = \log n$，$\max_{i}\bigl\{\log(1/\epsilon_i)\bigr\} = \log\frac{1}{\epsilon} + \log ((\log n)^2) + O(1) = \log\frac{1}{\epsilon} + O(\log \log n)$。

总空间：$n\log\frac{1}{\epsilon} + O(n\log \log n)$，论文的结论得证。

---
